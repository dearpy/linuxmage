小实验：
	提前准备：系统安装环境 cobber（ftp（yum 仓库） tftp（提供引导文件等） dhcp（提供所需IP）） 安装dns服务器 named
	1：安装配置调试：httpd两台 172.16.174.100 php wordpress 172.16.174.160 tomcat
			httpd php tomcat
	2：安装配置调试：mysql
	3：安装配置调试：varnish
	4：安装配置调试：nginx
	5：安装配置调试：haproxy
	6：安装配置调试：keepalived
	7：安装配置调试：zabbix监控
	8：安装配置调试：memcached xcache（缓存php code）
##1.httpd
	主配置文件：httpd.conf
	安装文件：base
	程序名称：httpd
	配置文件：/etc/httpd/{conf/,conf.d/}下文件
	启动文件：service  httpd start systemctl start httpd.service
		后端tomcat的http主机
			<VirtualHost 172.16.174.160:8888> 	设置虚拟主机
			        ServerName  httpd160.magedu.com
			        DocumentRoot /var/www/html
			        CustomLog  logs/access_httpd177_log combined
			        ErrorLog logs/error_httpd177_log
			        DirectoryIndex index.html index.php
			        ProxyRequests  Off
			        ProxyPreserveHost On
			        ProxyVia On
			        <Proxy *>
			                Require all granted
			        </Proxy>
			        ProxyPass / ajp://127.0.0.1:8009/

			        <Location />
			                Require all granted
			        </Location>

			<Location /server-status>
			        SetHandler server-status
			        Options none
			        AllowOverride None
			        Require ip 172.16.0.0/16
			</Location>
			</VirtualHost>
		后端php的http主机
			<VirtualHost 172.16.174.100:80>
                    ServerName varnish1.magedu.com
                    DocumentRoot /web/data/wordpress
                    CustomLog logs/access_user1_log combined
                    ErrorLog        logs/error_user1_log
            <Location /server-status>
                    SetHandler server-status
                    Options None
                    AllowOverride None
                    AuthType  basic
                    AuthName "For Administrators"
                    AuthUserFile  "conf/.htpasswd"
                    AuthGroupFile "conf/.htpsswd"
                    Require Group mygrps
                    Order allow,deny
                            allow from 172.16.0.0/16
                            deny from 172.16.174.201
            </Location>
            </VirtualHost>
	php
			安装文件及位置：base  php（fpm-php） php-mysql
			配置文件：/etc/php.ini /etc/php.d/*.ini  /etc/httpd/conf.d/php.conf
	tomcat
		a.准备环境：JDK
			配置jdk  PATH  JAVA_HOME
			jdk 多版本并存 alternates
		b.tomcat 配置
			systemctl start tomcat
			tomcat-users.xml 授权文件
			catalina.sh 编译安装文件bin目录下
			主要文件 /etc/tomcat/server.xml 主要修改项
				添加主机 <Context path="/shopxx" docBase="/web/jspweb/shopxx" reloadable="true"/>
				提供服务的contactor
					aip：<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
					http：<Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
	    					maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
	               			clientAuth="false" sslProtocol="TLS" />
					属性：
					port="8080"
					protocol="HTTP/1.1"
					connectionTimeout="20000"

					address：监听的IP地址；默认为本机所有可用地址；
					maxThreads：最大并发连接数，默认为150；
					enableLookups：是否启用DNS查询功能；
					acceptCount：等待队列的最大长度；
					secure：
				默认的主机：<Engine name="Catalina" defaultHost="localhost">

#2.mysql
	文件：centos7 mariadb-server mariadb；centos 6 mysql-server mysql
	配置文件：/etc/my.cnf
			 innodb_file_per_table = on
			 skip_name_resolve = on  centos6 应该不加（提示没有发现参数）
		util file:centos7 systemctl start mariadb.service ;centos 6 service mysqld start
	安装后可以做一些初始化的配置
		/usr/bin/mysql_install_db
		/usr/bin/mysql_secure_installation
	mysql语句：
		create database tomcat charset 'utf8'
		grant all on tomcat.* to 'tomcat'@'172.16.%.%' identified by 'tompass'

#3.varnish
	文件：epel源
	配置文件：/etc/varnish/default.vcl线程工作特性child cache   varnish.params 服务进程工作特性
	system unit file:
		/usr/lib/systemd/system/varnish.service			主程序
		/usr/lib/systemd/system/varnishlog.service		log日志：原始日志
		/usr/lib/systemd/system/varnishncsa.service		combined：format日志
	管理工具：
		varnishadm
		varnishadm -T 127.0.0.1:6082 -S /etc/varnish/secret
			登陆后help帮助
			常用命令
				vcl.load conf1 default.vcl
				vcl.use conf	1
				backend.list
				help [<command>]
				ping [<timestamp>]
	配置文件相关设置：
		default.vcl
		varnaish.params

#4.nginx
	程序文件：epel nginx.org
	system util file:启动nginx nginx -t nginx -s reload -s stop
	http upstream
	stream
#5.haproxy
	程序文件：epel源
	system util file：systemctl start haproxy
	配置文件:/etc/haproxy/haproxy.cfg
		global
			- log  <address> <facility> [max level [min level]]：定义全局的syslog服务器，最多可以定义两个
			-
		性能相关
			- maxconn <number>：设定每个haproxy进程所接受的最大并发连接数，其等同于命令行选项“-n”；“ulimit -n”自动计算的结果正是参照此参数设定的；
				永久有效配置文件：/etc/security/ulimit.conf {root       hard    nofile           4096}
		proxy  “defaults”、“listen”、“frontend”和“backend”
			“defaults”段用于为所有其它配置段提供默认参数，这配置默认配置参数可由下一个“defaults”所重新设定。

			 “frontend”段用于定义一系列监听的套接字，这些套接字可接受客户端请求并与之建立连接。

			 “backend”段用于定义一系列“后端”服务器，代理将会将对应客户端的请求转发至这些服务器。

			 “listen”段通过关联“前端”和“后端”定义了一个完整的代理，通常只对TCP流量有用。
	配置：
		haproxy正常工作需定义：frontend，backend；linsten
			frontend  main *:80  [可以此处定义，也可以bind单独定义]
						bind :80,:443
						bind 10.0.0.1:10080,10.0.0.1:10443
						bind /var/run/ssl-frontend.sock user root mode 600 accept-proxy
			    rspadd X-Via:\ HAPorxy  	/添加响应给客户端的首部信息 X-Via:\ HAPorxy
				rspdel X-Powered-By*    	/删除响应给客户端的首部信息 X-Powered-By*
			    default_backend       webapp /默认后端
				use_backend static 			/static 为静态服务器组名称
				http-request allow if url_block  /http 请求 allow deny  使用url_block 中定义的  acl url_block src 172.16.0.0
				block if url_block
				errorfile 403 /usr/share/haproxy/400.http  /错误文件 当出现 403错误时 响应页为 路径下的页面

			backend webapp
			    balance     roundrobin	（算法：static-rr,leastconn[推荐长连接情况使用LDAP,SQL],source[源地址hash，hash-type consistent推荐无cookie中使用]，uri [uri hash 代理缓存和反病毒代理 中使用提高缓存命中率，只适用于后端 http服务器场景，hash-type consistent]， url_param[追中用户时使用，hash-type consistent]，hdr [hash 请求首部的定义值，hash-type consistent ]）
							balance roundrobin
							balance url_param userid
							balance url_param session_id check_post 64
							balance hdr(User-Agent)
							balance hdr(host)
							balance hdr(Host) use_domain_only
				option  httpchk		/健康检测机制为httpchk
							option httpchk
							option httpchk <uri>
							option httpchk <method> <uri>
							option httpchk <method> <uri> <version>
				timeout client-fin <timeout> 优化 tcp段开超时时长（断开的后半段）
				timeout http-keep-alive <timeout> 持久连接的持久时长；
				rspadd X-Via:\ HAPorxy 	/相对于haproxy主机来说,添加响应报文(响应客户端) X-Via： reqadd（请求报文为发往后端 主机的）
				option forwardfor [ except <network> ] [ header <name> ] [ if-none ]
				compression algo <algorithm> ...：启用http协议的压缩机制，指明压缩算法gzip, deflate；
				compression type <mime type> ...：指明压缩的MIMI类型；html text
				compression offload		:不压缩
				maxconn <conns>：为指定的frontend定义其最大并发连接数；默认为2000；
				mode { tcp|http|health } 可以继承global中设置
			    cookie WEBSRV insert nocache indirect	{rewrite，insert，prefix}
			    server  web100 172.16.174.100:80 check
			    server  web167 172.16.174.167:80 check
					定义server 及选项：server <name> <address>[:[port]] [param*]
							[param*]：参数
								maxconn <maxconn>：当前server的最大并发连接数；
								backlog <backlog>：当前server的连接数达到上限后的后援队列长度；
								backup：设定当前server为备用服务器；
								check：对当前server做健康状态检测；option 中调用 httpchk
									addr ：检测时使用的IP地址；
									port ：针对此端口进行检测；
									inter <delay>：连续两次检测之间的时间间隔，默认为2000ms;
									rise <count>：连续多少次检测结果为“成功”才标记服务器为可用；默认为2；
									fall <count>：连续多少次检测结果为“失败”才标记服务器为不可用；默认为3；

										注意：httpchk，"smtpchk", "mysql-check", "pgsql-check" and "ssl-hello-chk" 用于定义应用层检测方法；

								cookie <value>：为当前server指定其cookie值，用于实现基于cookie的会话黏性；选项中调用：cookie WEBSRV insert nocache indirect
								disabled：标记为不可用；
								maxqueue <maxqueue>
								redir <prefix>：将发往此server的所有GET和HEAD类的请求重定向至指定的URL；
								weight <weight>：权重，默认为1;

				acl：实现动静分离
			listen stats  （可以自己定义前端和后端）
		        bind :9009
		        stats enable
		        stats admin if LOCALHOST
		        stats realm HAProxy\ Stats\ Page
		        stats uri /haproxy?stats
		        stats auth admin:admin
		        stats admin if TRUE

		配置文件相关参数：


#6.keepalived
	程序文件：base  keepalived
	配置文件：/etc/keepalived
		配置段说明
global_defs {
                   notification_email {
                        root@localhost
                   }
                   notification_email_from keeplived@localhost
                   smtp_server 127.0.0.1
                   smtp_connect_timeout 30
                   router_id Keepalive_1
                  vrrp_mcast_group4 224.0.174.18
                }
                vrrp_script chk_haproxy {
                        script "killall -0 haproxy && exit 0 || exit 1"
                        interval 1
                        weight 2
                }
                vrrp_instance VI_1 {
                    state MASTER
                    interface eno16777736
                    virtual_router_id 59
                    priority 100
                    advert_int 1
                    authentication {
                        auth_type PASS
                        auth_pass 1111
                        }
                    virtual_ipaddress {
                        172.16.174.3
                        }
                    track_interface {
                        eno16777736
                        }
 					track_script {
                        chk_haproxy
                        }
                notify_master "/etc/keepalived/notify.sh master"
                notify_backup "/etc/keepalived/notify.sh backup"
                notify_fault "/etc/keepalived/notify.sh fault"
                }


#7.zabbix
#8.增加性能memcached xcache
#9.lvs 补充