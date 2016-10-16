#LAMP
	LAMP是最强大的网站解决方案
	LAMP指的Linux（操作系统）、ApacheHTTP 服务器，MySQL（有时也指MariaDB，数据库软件） 和PHP（有时也是指Perl或Python） 的第一个字母，一般用来建立web应用平台。​
##
	linux httpd  mariadb  php  php-xcache
		A: httpd
		M: mysql或mariadb
		P: php/perl/python/rubyF
		   XCache
			快速而且稳定的PHP opcode缓存，经过严格测试且被大量用于生产环境。项目地址，http://xcache.lighttpd.net/

##httpd 
	http状态码：
		1XX：信息性状态码
		2XX：成功状态码
			200：OK
			201：CREATED
		3XX: 重定向类的状态码
			301: Moved Permanently, 永久重定向
			302: Found, 临时重定向，会在响应报文中使用“Location: 新位置”;
			304: Not Modified
		4XX：客户端类错误
			403：Forbidden
			404: Not Found
			405: Method Not Allowed
		5XX：服务器类的错误
			500：Internal Server Error, 服务器内部错误
			502：Bad Gateway, 代理服务器从上游服务器收到一条伪响应；
			503：Service Unavailable, 服务暂时不可用http状态码：
		http协议首部：
		通用首部
		请求首部
		响应首部
		实体首部
		扩展首部：非标准首部，可由程序员自行创建；X-Forward-For, X-Via

		通用首部：
			Connection: 定义C/S之间关于请求、响应的有关选项
				Connection: keep-alive
			Cache-Control: 缓存控制

		请求首部：
			Client-IP:
			Host: 请求的主机
			Referer: 指明了请求当前资源原始资源的URL
			User-Agent: 用户代理

			Accept首部：
				Accept: 服务端能够发送的媒体的类型
				Accept-Charset: 
				Accept-Encoding: 
				Accept-Language: 

			条件式请求：
			跟安全相关请求：
				Authorization：
				Cookie:


		响应首部：
			Age: 
			Server: 向客户说明自己的程序名称和版本

			协商首部：
				Vary: 首部列表，服务器会根据列表中的内容挑一个最适用的版本发送给客户端

			跟安全相关：
				WWW-Authentication：
				Set-Cookie

		实体首部：
			Location: 资源的新位置
			Allow: 允许对此资源使用的请求方法

			内容相关的首部：
				Content-Encoding: 
				Content-Language:
				Content-Length:
				Content-Location:
				Content-Type:

			缓存相关：
				ETag
				Expires
				Last-Modified:
	web服务端主应用程序
	服务端口号：http：80 https：443
	启动服务命令：C6 service httpd start C7 systemctl start httpd
	IPC Inter-Process Communication
	一次完整的http请求的处理过程：
		(1) 建立或处理连接：接收请求或拒绝请求；
		(2) 接收请求：接收客户端发来的具体请求报文；
		(3) 处理请求：对请求报文进行解析；
		(4) 访问资源：通过存储IO获取用户请求的资源； 
		(5) 构建响应报文：
		(6) 发送响应报文 ：
		(7) 记录于日志中：
	高度模块化：core + modules
		DSO: Dynamic shared objects
			支持动态装载和卸载；
		MPM：multipath processing modules
			prefork：一个主进程，多个子进程；一个进程响应一个请求；
				主进程：管理子进程；创建套接字；接收用户请求，并派发给某子进程处理；...
				子进程：处理请求、响应请求；
			worker：多进程多线程模型；一个线程响应一个请求； 
				主进程：管理子进程；创建套接字；接收用户请求，并派发给某子进程处理；...
				子进程：负责管理线程； 
				线程：处理并响应请求； 
			event：事件驱动模型，多进程模型，每个进程响应多个请求；
				主进程：管理子进程；创建套接字；接收用户请求，并派发给某子进程处理；...
				子进程：处理并响应请求； 
			切换模块方法
					C6：/etc/sysconfig/httpd HTTPD=/usr/sbin/httpd.worker
					C7:Include conf.modules.d/*.conf  /etc/httpd/conf.moudles.d 00-mpm.conf
						LoadModule mpm_worker_module modules/mod_mpm_worker.so
				httpd-2.2：event为测试模型；
					CentOS 6：MPM不支持DSO机制；
				httpd-2.4：production ready；支持DSO机制；
					CentOS 7：
###配置文件：
	/etc/httpd/conf/httpd.conf 
	/etc/httpd/conf.d/http.conf	
		主要选项
	####Section 1: Global Environment
		ServerRoot "/etc/httpd"
		Alias /images/ "/web/images/" 别名 通过定义别名 经查找的URL定位到指定路径
		PidFile run/httpd.pid 存放进程pid号C6：6204 C7 48521
		receives and sends time out：Timeout 60
		KeepAlive On
		MaxKeepAliveRequests 100 最大同时连接数量0为不限制
		KeepAliveTimeout 15 
		KeepAliveTimeout 500ms（C7 毫秒级限制）
	 	Listen 172.16.174.170:80 监听在IP：80端口下		
		LoadModule foo_module modules/mod_foo.so:动态的装载模块DSO:Dynamic Shared Object (DSO)
		ExtendedStatus On Server-status 页面加载额外信息
		User apache
		Group apache
		TypesConfig /etc/mime.types mime类型
		LoadModule deflate_module modules/mod_deflate.so 使用mod_deflate模块压缩页面优化传输速度
			适用场景：
				(1) 节约带宽，额外消耗CPU；同时，可能有些较老浏览器不支持；
				(2) 压缩适于压缩的资源，例如文件文件；
					SetOutputFilter DEFLATE					
					# mod_deflate configuration							
					# Restrict compression to these MIME types
					AddOutputFilterByType DEFLATE text/plain 
					AddOutputFilterByType DEFLATE text/html
					AddOutputFilterByType DEFLATE application/xhtml+xml
					AddOutputFilterByType DEFLATE text/xml
					AddOutputFilterByType DEFLATE application/xml
					AddOutputFilterByType DEFLATE application/x-javascript
					AddOutputFilterByType DEFLATE text/javascript
					AddOutputFilterByType DEFLATE text/css
					# Level of compression (Highest 9 - Lowest 1)
					DeflateCompressionLevel 9					 
					# Netscape 4.x has some problems.
					BrowserMatch ^Mozilla/4  gzip-only-text/html					 
					# Netscape 4.06-4.08 have some more problems
					BrowserMatch  ^Mozilla/4\.0[678]  no-gzip					 
					# MSIE masquerades as Netscape, but it is fine
					BrowserMatch \bMSI[E]  !no-gzip !gzip-only-text/html
	####Section 2: 'Main' server configuration
		ServerName www.deardy.com:80 开启设置后启动服务时不会提示
		DocumentRoot "/data/web" 默认网站文件的默认根路径
		DirectoryIndex index.html test.html 网站中的默认访问页
		AccessFileName .htaccess 不建议设置 极具的影响性能
		Redirect permanent /foo http://www.example.com/bar 永久重定向
		AddDefaultCharset UTF-8 默认字符集
	权限限制： 
		限制文件夹的访问权限，并通过IP及用户认证进行限制
			#<Directory "/data/web/admin">
			#       Options none
			#       AllowOverride none
			#       AuthType basic      #认证类型 还可以基于digest
			#       AuthName "For Administrators"  #认证时提示的信息
			#       AuthUserFile "/etc/httpd/conf/.htpasswd"    用户认证的密码文件可以htpasswd -c（创建）-m （md5）/etc/httpd/conf/.htpasswd USERNAME 
			#		AuthGroupFile "/etc/httpd/conf/.htgrps"		用户组文件：vim一个名叫.htgrps 文件即可 格式：mygrp（组名）： user1 （空格）user2
			#       Require valid-user/ Group mygrps
			#</Directory>
			通过IP认证 
			   	C6：Order allow,deny 
						allow from 172.16.0.0/16
						deny  from 172.16.174.100  匹配原则 基于最佳原则 匹配较小的
				C7:<RequireAll>
							Require ip 172.16.100.67
							Require all denied
						</RequireAll>					
						控制特定的客户端IP地址访问：
							Require ip IPADDR：授权
							Require not ip IPADDR：拒绝
							Require all granted
						控制特定的主机名访问：
							Require host HOSTNAME
							Require not host HOSTNAME
								HOSTNAME：
									FQDN：单个主机名
									domain.tld：域名内的所有主机
		限制URL的访问
			<Location /server-status>
					其他同上
			</Location>
		基于文件的访问限制方法
			<Files ~ "^\.ht">
		    	Order allow,deny
		    	Deny from all
		    	Satisfy All
		    </Files>
		可以支持的类型
			TypesConfig /etc/mime.types
		错误日志
			ErrorLog logs/error_log   LogLevel warn
				LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
				LogFormat "%h %l %u %t \"%r\" %>s %b" common
				LogFormat "%{Referer}i -> %U" referer
				LogFormat "%{User-agent}i" agent
			格式：
					%h	Remote host
					%l	Remote logname (from identd, if supplied).
					%u	Remote user (from auth; may be bogus if return status (%s) is 401)
					%t	Time the request was received (standard english format)
					%r	First line of request
					%s	Status. For requests that got internally redirected, this is the status of the *original* request --- %>s for the last.
					%b	Size of response in bytes, excluding HTTP headers.
					%{Foobar}i	The contents of Foobar: header line(s) in the request sent to the server. 
		访问日志
			CustomLog logs/access_log combined #必须定义格式
		虚拟主机
	#### Section 3: Virtual Hosts
			可以使基于 IP  端口 FQDN的类型 最常用的为基于FQDN 
			实例配置：
				打开虚拟主机设置：NameVirtualHost 172.16.174.170:80
				vim /etc/httpd/conf.d/user1.conf
					<VirtualHost 172.16.174.170:80>
						ServerName user1.magedu.com
						DocumentRoot /data/vhost/user1
						CustomLog logs/access_user1_log combined
						ErrorLog	logs/error_user1_log
					<Location /server-status>
						SetHandler server-status
						Options None
						AllowOverride None
						AuthType  basic
						AuthName "For Administrators"
						AuthUserFile  "conf/.htpasswd"
						AuthGroupFile "conf/.htpsswd"
						Require Group mygrps
					</VirtualHost>
				（2）实现运行tom 查看/server-status信息
					htpasswd -m /etc/conf/.htpasswd tom
					输入密码mageedu
					在组文件中添加Tom vim/etc/httpd/conf/,htgrps    组名：组成员  mygrps：wang 空格 tom
					必须有mod_status模块  可以添加  ExtendedStatus On   看额外信息
				（3）不允许192.168.0.0/24任何主机访问 添加 基于IP的访问控制
					<Directory />
			        			Options None
			       			 AllowOverride None
			      			 Order allow,deny
			     			 Allow from 172.16.0.0/16
			        		 Deny from  192.168.0.0/24
					</Directory>
			
##配置httpd支持https：
	(1) 为服务器申请数字证书；
		测试：通过私建CA发证书
			(a) 创建私有CA
			(b) 在服务器创建证书签署请求
			(c) CA签证
			
	(2) 配置httpd支持使用ssl，及使用的证书；
		# yum -y install mod_ssl

		配置文件：/etc/httpd/conf.d/ssl.conf
			DocumentRoot
			ServerName
			SSLCertificateFile	/etc/pki/tls/certs/httpd.crt 
			SSLCertificateKeyFile /etc/pki/tls/private/httpd.key 
				本地生成httpd.key私钥：
					(umask 077;openssl genrsa -out httpd.key 1024 )
				向CA请求颁发证书：	
				openssl req -new -key httpd.key -out httpd.csr -days 365		
	(3) 测试基于https访问相应的主机；
		# openssl  s_client  [-connect host:port] [-cert filename] [-CApath directory] [-CAfile filename]
##PHP
	动态资源技术服务器端响应程序；脚本编程语言、专为web开发而设计、将代码嵌入到html中
		PHP的源码在结构上非常清晰。其代码根目录中主要包含了一些说明文件以及设计方案，并提供了如下子目录：		
			1、build —— 顾名思义，这里主要放置一些跟源码编译相关的文件，比如开始构建之前的buildconf脚本及一些检查环境的脚本等。
			2、ext —— 官方的扩展目录，包括了绝大多数PHP的函数的定义和实现，如array系列，pdo系列，spl系列等函数的实现。 个人开发的扩展在测试时也可以放到这个目录，以方便测试等。
			3、main —— 这里存放的就是PHP最为核心的文件了，是实现PHP的基础设施，这里和Zend引擎不一样，Zend引擎主要实现语言最核心的语言运行环境。
			4、Zend —— Zend引擎的实现目录，比如脚本的词法语法解析，opcode的执行以及扩展机制的实现等等。
			5、pear —— PHP 扩展与应用仓库，包含PEAR的核心文件。
			6、sapi —— 包含了各种服务器抽象层的代码，例如apache的mod_php，cgi，fastcgi以及fpm等等接口。
			7、TSRM —— PHP的线程安全是构建在TSRM库之上的，PHP实现中常见的*G宏通常是对TSRM的封装，TSRM(Thread Safe Resource Manager)线程安全资源管理器。
			8、tests —— PHP的测试脚本集合，包含PHP各项功能的测试文件。
			9、win32 —— 这个目录主要包括Windows平台相关的一些实现，比如sokcet的实现在Windows下和*Nix平台就不太一样，同时也包括了Windows下编译PHP相关的脚本。
	Web资源的类型：
		静态资源：原始形式与响应结果一致；
		动态资源：原始形式通常为程序文件，需要运行后将运行结果呈现给用户；
			客户端技术：js
			服务器端技术：php, jsp	
	
		CGI：Common Gateway Interface（协议）
			通过CGI调用支持CGI接口的应用程序，并让其加载到指定的程序文件，完成程序执行，并将数据放回给前端httpd
			注：服务器端（httpd）可以通过用户请求的URL后缀名判断是动态资源还是静态资源，然后进行处理；如果是静态资源自给处理，动态资源将启动解释器（perl）进行执行加载处理（加载执行动态资源，如果涉及到数据读取还要连接到数据库），返回给（httpd），httpd将返回给强求的客户端。有
		可以让一个客户端，从客户端代理向运行在网络服务器上程序传输数据；CGI描述了客户端和服务器程序之间传输数据的一种标准；
		
		请求流程：
			Client --(http) --> httpd --> (cgi) --> application process (code) --> (mysql) --> mysqld (mariadb)	
	httpd与php结合方式：			
			静态资源：Client -- http --> httpd
			动态资源：Client -- http --> httpd --> libphp5.so ()
			动态资源：Client -- http --> httpd --> libphp5.so () -- mysql --> MySQL server
			CGI： httpd 启动一个子进程
			module （把php编译成为httpd的扩展模块）
				MPM：
					prefork: libphp5.so
					event, worker: libphp5-zts.so
			FastCGI：
				fpm	 fast process manager
					其像prefork worker event 一样 生成套接字服务 可以通用开启几个进程最多空闲的进程等等。
					相当于反代程序（模糊）	
			LAMP的实现方式：
				httpd(prefork)+libphp5.so+mysql
				httpd(event)+libphp5-zts.so+mysql
				httpd+fpm(php)+mysql			
			CentOS 6：
				mysql-server-5.1
			CentOS 7
				mariadb-server-5.5
	php的安装：
		httpd(prefork)
			# yum install php 
				(phplib5.so)	
		配置文件：
			/etc/php.ini, /etc/php.d/*.ini 
		配置文件在php解释器启动时被读取，因此，对配置文件的修改如何生效？
			Modules：重启httpd服务；
			FastCGI：重启php-fpm服务；	
		ini：
			[foo]：Section Header
			directive = value
			
			注释符：较新的版本中，已经完全使用;进行注释；
			#：纯粹的注释信息
			;：用于注释可启用的directive
			
			php.ini的核心配置选项文档：  http://php.net/manual/zh/ini.core.php
			php.ini配置选项列表：http://php.net/manual/zh/ini.list.php
		测试php：
			<?php 
				phpinfo();
			?>				
		CentOS 6：
			# yum install httpd php php-mysql mysql-server
			# service mysqld start
			# service httpd start 
			
		CentOS 7：
			# yum install httpd php php-mysql mariadb-server
			# systemctl start mariadb.service httpd.service 
		测试mysql连接性：
			<?php
				$conn = mysql_connect('172.16.100.71','testuser','testpass');
				if($conn)
					echo "OK";
				else
					echo "Failure";
			?>				
##php：编程语言；普遍需求，通用的功能模块；扩展，extensions；
	程序：分解多个包
		主包：php-5.3.3-
		支包：name-subname-version-release.rpm 
	
		CentOS 6：os:dvd1 dvd2  extra updates clouds
		CentOS 7：
		EPEL：
			发行版 或更新  epel 源   项目官方提供
	性能扩展方式：
		向上扩展：Scale Up
		向外扩展：Scale Out 
	
	httpd与php结合方式：
		CGI：Common Gateway Interface 创建子进程销毁子进程（调度可理解请求资源的解释器：如 Perl）
		Module：php，编译成httpd的扩展模块；只需要httpd启动时装载php模块（libphp5.so libphp5-zts.so）
		prefork：libphp5.so
		worker, event：libphp5-zts.so 
		fastCGI：C/S架构，socket(可以单独放在另一个主机服务进程；当请求动态资源时httpd将自己扮演成fastcgi客户端，将请求发送php。fastcfi完成资源URL映射，到本地加载资源，并执行后发送给客户端httpd)；
			更换httpd模块方法：C7：/etc/httpd/conf.modules.d/00-mpm.conf
							 C6：/etc/sysconfig/httpd
		php-fpm：fastcgi管理器
		CentOS 6：
		httpd-2.2：默认不支持fcgi模块，需要自行编译扩展；
		php-5.3.3：不支持fpm机制，需要自行打补丁编译安装；
			注：MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式
	LAMP的实现方式：
		httpd(prefork)+libphp5.so+mysql
		httpd(event)+libphp5-zts.so+mysql
		httpd+fpm(php)+mysql	
##MySQL的命令行客户端程序：mysql
		-uUSERNAME
		-hHOST
		-pPASSWORD	
	支持SQL语句对数据完成管理：
		DDL，DML
			DDL：CREATE, ALTER, DROP
			DML：INSERT，DELETE，SELECT，UPDATE			
	mysql> GRANT ALL ON db_name.tbl_name TO username@'host' IDENTIFIED BY 'password';
		_：任意单个字符
		%：任意长度的任意字符		
	管理员账号：root/localhost, 密码默认为空；	
	配置文件：/etc/my.cnf, /etc/my.cnf.d/*.cnf
		[mysqld]
		innodb_file_per_table = ON
		skip_name_resolve = ON	
		datadir=/var/lib/mysql
					添加字符集	
	安装完成后，建议运行 mysql_secure_installation一次；
##解决方案：编译安装httpd-2.4, php-5.3.3+
	
		CentOS 7：
		httpd-2.4：rpm包默认编译安装了fcgi模块；
		php-fpm：php-fpm-version.rpm包
			/usr/lib64/httpd/modules/mod_proxy_fcgi.so
	
	配置文件：
		服务进程配置文件：/etc/php-fpm.conf, /etc/php-fpm.d/*.conf
		解释器配置文件：/etc/php.ini, /etc/php.d/*.ini 
	启动程序：systemctl start php-fpm.service	
	服务进程配置文件：
		[global]：全局配置
		[pool]：连接池配置
			listen = 127.0.0.1:9000 外部通信地址（172.16.174.160：9000）
			listen.backlog = -1（无限）  后援队列（定义大小100.队列100个以后的将被抛弃）
			
			listen.allowed_clients = 127.0.0.1（允许哪些主机当客户端连接 此服务）
			
			user = 	（以哪个身份运行）
			group = 
			
			pm = dynamic|static（静态：启动时启动固定进程-只pm.max_children有限  动态：随机分配：针对以下设置）
			pm.max_children
			pm.start_servers
			pm.min_spare_servers
			pm.max_spare_servers
			pm.max_requests 
			注：pm.status_path = /pm-status
		pm方式的php进程存储session的路径：
			php_value[session.save_handler] = files
			php_value[session.save_path] = /var/lib/php/session (此文件pm的执行者有写权限)
			
			# mkdir /var/lib/php/session
			# chown apache.apache /var/lib/php/session
			
		配置示例：如何将用户请求反代到 fcgi
			<VirtualHost *:80>
				ServerName www1.magedu.com
				DocumentRoot /data/vhosts/www1 静态资源
				ProxyRequests Off
				DirectoryIndex index.php								
				ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/data/vhosts/www1/$1 
				<Directory "/data/vhosts/www1">
					Options None
					AllowOverride None
					Require all granted
				</Directory>
			</VirtualHost>			
##编译安装amp：
		httpd：httpd-2.4
		mariadb：mariadb-5.5
		php5：php-5.4 
	
	注意：被依赖到的每个组件，在编译时主要用到的是其开发库和头文件；通常由NAME-devel-VERSION.rpm；
	
	开发包组：Development Tools, Server Platform Development
	
	CentOS 6专用：
	(1) apr
		# ./configure --prefix=/usr/local/apr
		# make && make install
	(2) apr-util
		# ./configure --prefix=/usr/local/apr-util  --with-apr=/usr/local/apr
		# make && make install
		
	CentOS 7专用：
		# yum install apr-devel apr-util-devel prce-devel openssl-devel libevent-devel
	
	
	安装httpd-2.4：
	
			# yum install pcre-devel openssl-devel  libevent-devel 
			
			# ./configure --prefix=/usr/local/apache24 --sysconfdir=/etc/httpd24 --enable-so --enable-ssl --enable-cgi --enable-rewrite --enable-modules=all --enable-mpms-shared=all --with-mpm=prefork --with-pcre --with-zlib --with-apr=/usr --with-apr-util=/usr

				--prefix=/usr/local/apache24
				 --sysconfdir=/etc/httpd24 
				--enable-so 
				--enable-ssl 
				--enable-cgi 
				--enable-rewrite 
				--enable-modules=all 
				--enable-mpms-shared=all 
				--with-mpm=prefork 
				--with-pcre 
				--with-zlib 
				--with-apr=/usr 
				--with-apr-util=/usr
			# make -j # 
			# make install 
			配置文件为安装目录的 httpd.conf 和 extra目录下的.conf文件 ，original是原始默认的配置文件
			httpd的程序安装位置为 --prefix=/usr/local.apache24 --sysconfdir= 配置文件的路径
				bin目录为 工具程序 
				built 编译目录 config.nice rules.mk 编译时所构建用到的指令
			cgi-bin 放cgi脚本的 cgi的网页要放在此目录下
			error 错误页
			htdocs 默认网页的存储位置 documentroot
			icons 图标
			include 头文件  二次开发 基于他编译其他文件
			logs日志文件
			man帮助文档
			manual手册页（网页格式）
			modules（编译的模块）
		编译安装后的工作：
			（1）（导出环境变量）vim /etc/profile.d
				vim httpd24.sh  export PATH=/usr/local/apache24/bin:$PATH
				./etc/profile.d/httpd24.sh
			（2）man手册 man.config 加
			（3）库文件 ldd -p ldd -v 如果没有就不用导出
					vim /etc/ld.so.conf/httpd24.conf /usr/local/apache24/lib
			（4）include文件
					ln -sv /usr/local/apache24/include /usr/include
		启动 httpd 命令 也可以 apachectl
MariaDB(mysql)：
		
	MySQL AB  --> MySQL
		Solaris：二进制版本；		
		www.mysql.com
	ariaDB的特性：
		插件式存储引擎：存储管理器有多种实现版本，彼此间的功能和特性可能略有区别；用户可根据需要灵活选择； 	
		
		存储引擎也称为“表类型”；
		
		(1) 更多的存储引擎；
			MyISAM：不支持事务；
			MyISAM --> Aria
			InnoDB --> XtraDB 
				：支持事务；
- 
- 
		(2) 诸多扩展和新特性；
		(3) 提供了较多的测试组件；
		(4) truly open source；
		
	MySQL的发行机制：
		Enterprise：提供了更丰富的功能；
		Community：
		
	安装和使用MariaDB：
		
		安装方式：
			(1) rpm包；
				(a) 由OS的发行商提供；
				(b) 程序官方提供；
			(2) 源码包；
			(3) 通用二进制格式的程序包；
			
		通用二进制格式安装MariaDB：
			(1) 准备数据目录；
				以/mydata/data目录为例；
			(2) 安装配置mariadb						
				# useradd  -r  mysql
				# tar xf  mariadb-VERSION.tar.xz  -C  /usr/local
				# cd /usr/local
				# ln  -sv  mariadb-VERSION  mysql
				# cd  /usr/local/mysql
			### chown  -R  root:mysql  ./*
				# scripts/mysql_install_db  --user=mysql  -datadir=/mydata/data
				# cp  support-files/mysql.server   /etc/init.d/mysqld 解压的ln 后的mysql
				# chkconfig   --add  mysqld
			(3) 提供配置文件
				cp support-files/mysql-large.cnf /etc/mysql/my.cnf
				mkdir /etc/mysql/my.cnf  
					skip_file_per_talbe = ON
					skip_name_resolve = ON
				ini格式的配
			注意：可能会启动失败：mv/etc/my.cnf{,.bak},重新启动
			配置：导出 PATH ld.so.conf mysql.sh ln sv man 手册
	置文件；各程序均可通过此配置文件获取配置信息；
					[program_name]
									
				OS Vendor提供mariadb rpm包安装的服务的配置文件查找次序：
					/etc/mysql/my.cnf  --> /etc/my.cnf  --> --default-extra-file=/PATH/TO/CONF_FILE  --> ~/.my.cnf
					
				通用二进制格式安装的服务程序其配置文件查找次序：
					/etc/my.cnf  --> /etc/mysql/my.cnf  --> --default-extra-file=/PATH/TO/CONF_FILE  --> ~/.my.cnf
					
				获取其读取次序的方法：
					mysqld  --verbose  --help
					
				# cp  support-files/my-large.cnf  /etc/my.cnf
				
				添加三个选项：
					datadir = /mydata/data
					innodb_file_per_table = ON
					skip_name_resolve = ON
					
			(4) 启动服务
				chkconfig --add mysqld
				chkconfig --level 35 mysqld on




				# service  mysqld  start
	
	安装php-5.4：
		# yum install libxml2-devel gd-devel freetype-devel libmcrypt-devel
			gd动态绘图
			xml广泛应用于网络数据交换，配置文件、Web服务
		# ./configure --prefix=/usr/local/php54 --with-mysql=/usr/local/mysql --with-openssl --with-mysqli=/usr/local/mysql/bin/mysql_config --enable-mbstring --enable-xml --enable-sockets --with-freetype-dir --with-gd --with-libxml-dir=/usr --with-zlib --with-jpeg-dir --with-png-dir --with-mcrypt --with-apxs2=/usr/local/apache24/bin/apxs --with-config-file-path=/etc/php54.ini --with-config-file-scan-dir=/etc/php54.d
		 ./检查编译环境是否符合，符合将结合makefile.in生成makefile文件
		# make -j #
		# make install
		包with进来才能调用其他的库完成高级的功能
			--with-apxs2=/usr/local/apache24/bin/编译成模块 prefork phplib5.so worker event phplib5-zts.so
		    --with-mysqli=/usr/local/mysql/bin/mysql_config mysql编程接口
			--with-mysql=/usr/local/mysql 不是默认安装的要指明路径
			--enable-mbstring支持多字节字符串
			--enable-xml 支持xml
			--enable-sockets 支持socket通信 本机的
			--with-freetype-dir （rpm包安装，默认usrlocal）
			--with-gd 动态绘图
			--with-libxml-dir=/usr 
			--with-zlib 
			--with-jpeg-dir 
			--with-png-dir 
			--with-mcrypt 支持加密库
			--with-apxs2=/usr/local/apache24/bin/apxs 编译成模块
			--with-config-file-path=/etc/php54.ini 	配置文件路径
			--with-config-file-scan-dir=/etc/php54.d 片段配置文件
	注意：如果httpd使用了线程式MPM，则编译php时应该额外使用--enable-maintainer-zts; 
	
	配置httpd：
		配置文件： cd php54 cp php.ini.production /etc/php54.ini
	检查模块：	/usr/local/apache24/modules
	使用要load模块：/etc/httpd24/httpd.conf 找到配置模块的位置
		LoadModule php5_module modules/libphp5.so 模块位置
		AddType application/x-httpd-php .php  .php
		DirectoryIndex index.php index.html if<IfModule dir_module>	位置添加
	默认位置：添加一个主页 
		/usr/local/apache24/htdocs下
	改下时区：/etc/php54.ini date.timezone = Asia/Shanghai
	部署phpMyAdmin:
		Web GUI，用于管理mysql数据库；
	重载 一下 

##编译amp：
		httpd-2.4, php-5.5.40, mariadb-5.5.46 
			
		
		LAMP(4):
	
	opcode加速器：
		APC，eAccelerator, Xcache, ...
	并不是优化了性能；而是通过缓存opcode可供多进程共享；
	
		xcache：
		http://xcache.lighttpd.org
		
		# /usr/local/php54/bin/phpize
		# ./configure --enable-xcache --with-php-config=/usr/local/php54/bin/php-config
		# make && make install
		
		# mkdir /etc/php54.d/
		# cp xcache.ini  /etc/php54.d/
	
	
	
	rsyslog：
	
	日志：历史事件记录；
	事件：时间，事件，事件级别
	
		syslog：
		klogd：kernel
		syslogd：system(application)
	
	事件记录格式：
	日期时间 	主机 	进程[pid]: 事件内容
	
	C/S架构：通过TCP或UDP提供日志记录服务；
	
		rsyslog：
		rsyslogd特性：
	多线程；
		UDP，TCP，SSL，TLS，RELP；
	存储日志信息于MySQL，PGSQL，Oracle等RDBMS；
	强大的过滤器，实现过滤日志信息中任何部分的内容；
	自定义输出格式；
	
	elk stack：elasticsearch, logstash, kibina
	
	rsyslog日志收集器的重要术语：
	facility：设施，从功能或日志信息来源的程序对日志收集进行分类：
	auth, authpriv, cron, daemon, kern, lpr, mail, mark, news, security, user, uucp, syslog, local0-local7 
	priority：优先级
	debug, info, notice, warn(warning), err(error), crit(critical), alert, emerg(panic)
	
	程序包：rsyslog
	程序环境：
	配置文件：/etc/rsyslog.conf, /etc/rsyslog.d/*.conf
	主程序：rsyslogd 
	C6：service rsyslogd {start|stop|restart|status}
	C7: systemctl start|stop|restart|status rsyslog.service
	
	配置文件格式：
	由三部分组成：
		MODULES
		GLOBAL DRICTIVES
		RULES：定义日志记录规则
		
	RULES:
		facility.priority	target 
		
		priority：
			*：所有级别
			none：没有级别，不记录日志；
			PRIORITY：此级别（含）及其上所有级别的信息；
			=PRIORITY：仅记录指定的级别的日志信息；
			
		facility：
			*：所有设施；
			f1,f2,...：指定的列表；
			
		target：
			文件：将日志信息记录到指定的文件中；-表示异步写入；
			用户：将日志事件通知给指定的用户；通过将信息发送给登录到系统上的用户终端进行；
			日志服务器：@host，把日志通过网络送往指定的服务器；
			管道：| COMMAND
			
	运行rsyslog为服务器：
	# Provides UDP syslog reception
	$ModLoad imudp
	$UDPServerRun 514
	
	# Provides TCP syslog reception
	$ModLoad imtcp
	$InputTCPServerRun 514
	
	其它的日志文件：
	/var/log/btmp：当前系统上，用户尝试登录失败相关的日志；
		lastb命令查看
		
	/var/log/wtmp：当前系统上，用户正常 登录的日志；
		last命令查看
			-n #：仅查看最近#条记录；
			
		lastlog命令：用于查看每个用户最近一次登录时间；
	
	/var/log/dmesg：系统引导过程中的日志信息；
		文本文件查看工具；
		也可以使用dmesg命令查看；
		
	rsyslog记录日志于mysql：
	(1) 安装rsyslog连接至mysql的模块；
	# yum install rsyslog-mysql 
	(2) 在mysql中创建专用于rsyslog-mysql的用户账号；
	mysql> GRANT ALL ON Syslog.* TO 'USERNAME'@'HOST' IDENTIFIED BY 'PASSWORD'
	(3) 生成所需的数据库和表；
	# mysql  -uUSERNAME -hHOST -p < /usr/share/doc/rsyslog-7.4.7/mysql-createDB.sql
	(4) 配置rsyslog使用ommysql模块
	### MODULES ###
	$ModLoad  ommysql 
	
	(5) 配置RULES，将某facility的日志发往mysql
	### RULES ###
	facility.priority 	:ommysql:DBHOST,DB,DBUSERNAME,DBUSERPASS
	
	(6) 重启rsyslog进程
	
	(7) 提供Web GUI工具loganalyzer
	(a) 配置amp组合；
	(b) 安装loganalyzer
		# tar xf loganalyzer-3.6.6.tar.gz
		# cd  loganalyzer-3.6.6
		# cp -r loganalyzer-3.6.6/src  /var/www/html/loganalyzer
		# cp contrib/*.sh  /var/www/html/loganalyzer
		# cd  /var/www/html/loganalyzer
		# chmod +x *.sh
		# ./configure.sh
		# ./secure.sh
		# chmod 666 config.php
		
	通过浏览器访问：
		http://HOST/loganalyzer/install.php 
		
			MySQL Native, Syslog Fiel
			
			Table type要选择“Monitorware”


http协议、httpd(apache)、php、SQL（MariaDB/PGSQL）、NoSQL（Memcached, redis）；nfs/cifs/ftp、iptables；

程序=指令+数据；
程序=算法+数据结构

lamp：

http：

http协议：
http/0.9：原型版本；
http/1.0：cache, MIME(multipupose internet Mail Extensions)
method：GET、POST、HEAD、PUT、DELETE、TRACE、OPTIONS、...		
http/1.1：缓存功能大大增强
speedy：spdy
http/2.0
		
80/tcp


https协议：
443/tcp

IANA:
0-1023：众所周知的，永久地分配给固定的应用使用；特权端口（仅root可用）；
1024-41951：注册端口，但要求不是特别严格，分配给程序注册为某应用使用；
41952+：客户端程序使用的随机端口，动态端口，或称为私有端口；/proc/sys/net/ipv4/ip_local_port_range；

BSD Socket：IPC一种实现，允许位于不同主机之上的进程之间互相通信的解决方案之一；
Socket API：
SOCK_STREAM：tcp套接字；
SOCK_DGRAM：udp套接字；
SOCK_RAW：裸套接字；

根据套按使用的地址格式：
AF_INET：ipv4地址家族；
AF_INET6：ipv6
AF_UNIX：Unix_sock；

TCP Finite State Machine：

TCP协议的特性：
建立连接：三次握手；
将数据打包成段：校验和（CRC32）
确认、重传及超时；
排序：逻辑序号；
流量控制：滑动窗口；
拥塞控制：慢启动及拥塞避免算法；

http：hyper text tranfer protocol， 超文本传输协议；

html：hyper text mark language，超文本标记语言；

<html>
<head>
	<title></title>
</head>
<body>
	
</body>
</html>

css
js

工作模式：request/response
一次完整的http事务：请求<-->响应；

web资源：
	一个html文档；
	一个图片；
	一个mp3文件片断；
	...
	
	URL：资源标识，用于描述服务器上某特定资源的位置；
		Uniform Resource Locator
			scheme://Server[:port]/PATH/TO/SOME_RESOURCE
			
	资源的种类：
		静态资源：.jpg, .gif, .png, .html, .txt, 
		动态资源：
			服务器端技术：.php, .jsp, ...
			客户端技术：.js
	
一次完整的http请求的处理过程：
(1) 建立或处理连接：接收请求或拒绝请求；
(2) 接收请求：接收客户端发来的具体请求报文；
(3) 处理请求：对请求报文进行解析；
(4) 访问资源：通过存储IO获取用户请求的资源； 
(5) 构建响应报文：
(6) 发送响应报文 ：
(7) 记录于日志中：

并发响应模型：
单进程I/O模型：串行响应；
多进程I/O模型：同时启动多个进程，每个进程响应一个请求；
复用的I/O模型：一个进程响应多个请求；
	多线程模型：一个进程生成多个线程，每个线程响应一个请求；
	事件驱动：一个进程直接响应多个请求； 
复用的多进程I/O结构：启动m个进程，每个进程生成n个线程，每个线程响应一个请求；
	
资源映射：
chroot:
	/var/test/a/b/index.html
	
	chroot /var/test, 
		/a/b/index.html

	例如：/var/www/html/
			images/logo.jpg
		http://www.magedu.com:80/images/log.jpg
		
	DocumentRoot
	
	web服务器的资源映射机制：
		(a) DocumentRoot
		(b) alias 
		(c) 虚拟主机的docroot
		(d) 用户的docroot
		...
		
http请求处理中的连接方式：
保持连接：长连接，keepalive
非保持连接：短连接, 

时间：
数量：

http协议的实现：
简单的基本http协议服务器：
httpd (apache)
nginx
lighttpd

application server：动态服务器技术；
iis, tomcat, jetty, resin, ...
weblogic, websphere, jboss, glassfish, ...

httpd：
www.netcraft.com

ASF：apache software foundation

apache，a patchy server, httpd

httpd的特性：
高度模块化：core + modules
DSO: Dynamic shared objects
	支持动态装载和卸载；
MPM：multipath processing modules
	prefork：一个主进程，多个子进程；一个进程响应一个请求；
		主进程：管理子进程；创建套接字；接收用户请求，并派发给某子进程处理；...
		子进程：处理请求、响应请求；
	worker：多进程多线程模型；一个线程响应一个请求； 
		主进程：管理子进程；创建套接字；接收用户请求，并派发给某子进程处理；...
		子进程：负责管理线程； 
		线程：处理并响应请求； 
	event：事件驱动模型，多进程模型，每个进程响应多个请求；
		主进程：管理子进程；创建套接字；接收用户请求，并派发给某子进程处理；...
		子进程：处理并响应请求； 
		
		httpd-2.2：event为测试模型；
			CentOS 6：MPM不支持DSO机制；
		httpd-2.4：production ready；支持DSO机制；
			CentOS 7：
			
httpd的版本：
	httpd-1.3：官方已经停止维护；
	httpd-2.0：
	httpd-2.2：
	httpd-2.4：
	
	httpd.apache.org
	
httpd的功能特性：
	CGI：common gateway interface；
	虚拟主机：IP， PORT， HOSTNAME
	反向代理 
	负载均衡：bytraffic, bybusiness, byrequest
	路径别名
	丰富的用户认证机制
		basic:
		digest:
	支持第三方模块 
	...
	
安装httpd：
rpm包：CentOS base源；
编译安装：定制新功能，或其它原因；

CentOS 6：httpd-2.2
sysinit脚本：/etc/rc.d/init.d/httpd
程序环境：
	配置文件：
		/etc/httpd/conf/httpd.conf
		/etc/httpd/conf.d/*.conf
	程序文件：
		/usr/sbin/httpd
		/usr/sbin/httpd.event
		/usr/sbin/httpd.worker
		
		脚本配置文件：/etc/sysconfig/httpd
	日志文件：
		/var/log/httpd 
			access_log：访问日志
			error_log：错误日志
	站点文档根目录：
		/var/www/html
	模块文件路径：
		/usr/lib64/httpd/modules
		
chkconfig httpd on|off
	
CentOS 7：httpd-2.4
Systemd Unit File：/usr/lib/systemd/system/httpd.service

程序环境：
	配置文件：
		/etc/httpd/conf/httpd.conf
		/etc/httpd/conf.modules.d/*.conf
		/etc/httpd/conf.d/*.conf
	程序文件：
		/usr/sbin/httpd

		MPM支持DSO机制，所以各为一个独立的模块；
		
	日志文件：
		/var/log/httpd 
			access_log：访问日志
			error_log：错误日志
	站点文档根目录：
		/var/www/html
	模块文件路径：
		/usr/lib64/httpd/modules	
		
systemctl enable httpd.service


回顾：http, httpd
http: 协议
http/1.1, http/2.0
html

httpd：
MPM：
prefork：一个进程响应一个请求；
worker：一个线程响应一个请求；
event：一个进程响应多个请求；

http(2)

httpd-2.2的基础配置

/etc/httpd：ServerRoot
conf/httpd.conf、conf.d/*.conf：配置文件
logs：日志文件
modules：模块文件

主配置文件：/etc/httpd/conf/httpd.conf
directive value
	directive：不区分字符大小写；例如：ServerRoot；
	value：除了文件路径这外，大多数不区分字符大小写；
	
### Section 1: Global Environment
### Section 2: 'Main' server configuration
### Section 3: Virtual Hosts	

修改后生效：
	reload
	restart：通常仅修改了监听的地址和端口；
	
1、修改监听的地址端口；
Listen [IP:]PORT
	(1) 可定义多次；
		Listen 80
		Listen 8080
	(2) 省略IP，表示0.0.0.0；
	
2、持久连接
persistent connection：tcp连接建立后，资源获取完成不会断开连接，而是继续等待请求其它资源； 
	如何断开？
		数量限制 
		时间限制
		
	相关指令：
		KeepAlive On|Off
		MaxKeepAliveRequests 100
		KeepAliveTimeout 15
		
请求测试：
	$ telnet SERVER_IP PORT
	GET /test1.html HTTP/1.1     
	Host: SERVER_IP
	
	HTTP/1.1 200 OK
	Date: Tue, 14 Jun 2016 06:04:36 GMT
	Server: Apache/2.2.15 (CentOS)
	Last-Modified: Tue, 14 Jun 2016 03:34:43 GMT
	ETag: "60102-10-53534af955806"
	Accept-Ranges: bytes
	Content-Length: 16
	Connection: close
	Content-Type: text/html; charset=UTF-8

	test1.html page
	Connection closed by foreign host.
	
3、MPM 
multipath processing module：多路处理模块；

	httpd-2.2的MPM机制不支持DSO机制，event为测试；
		httpd：prefork
		httpd.worker: worker
		httpd.event: event
		
		/etc/sysconfig/httpd
		HTTPD=/usr/sbin/httpd|httpd.worker|httpd.event
		
	查看httpd程序的模块列表：
		查看静态编译的模块：
			httpd -l
		查看编译的所有模块：
			httpd -M
			
	80, 500ms, 256
		512*86400/80=55W PV
			Page View
		UV: User View 
		
4、DSO 
LoadModule指令
	LoadModule  Mod_Name  modules/Module_File.so
	
相对路径，是相对于ServerRoot指令所定义的路径而言；

5、	‘Main' Server

定义一个主机的基本指令：
	ServerName  FQDN：PORT
	DocumentRoot /var/www/html
	
		http://www.magedu.com/
	
6、站点资源访问控制

基于文件系统进行
	<Directory "/PATH/TO/SOME_DIR">
	</Directory>
	
	<File "">
	</File>
	
	<FileMatch "PATTERN">
	</FileMatch>

基于url路径进行
	<Location "/PATH/TO/SOME_URL">
	</Location>
	
	<LocationMatch "URL_PATTERN">
	...
	</LocationMatch>
	
目录中的常用指令：
	(1) Options：用于定义资源的展示方式；后跟以空白字符分隔的“选项”列表； 
		Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews None All
			Indexes：允许索引；
			FollowSymLinks：允许跟踪符号链接；
			SymLinksifOwnerMatch
			ExecCGI：允许执行CGI脚本；
			
	(2) AllowOverride
		httpd允许在网页文档的各目录下使用.htaccess文件实现单目录资源的访问控制；表示哪此指令可以存放于.htaccess文件中；
			/data/web/
				.htaccess
				admin: .htaccess 
				images: .htaccess
				
	(3) order和allow/deny from
		基于IP地址的访问控制；
		
		order用于定义allow和deny的生效次序；
		allow from IP/NETWORK/FQDN
		deny from IP/NETWORK/FQDN 
			来源地址格式：
				IP
				NetAddr：
					172.16
					172.16.0.0
					172.16.0.0/16
					172.16.0.0/255.255.0.0
				FQDN
				DAMAIN
				
			来源请求遵循最佳匹配法则机制；
				
		order allow, deny 
		
7、定义站点主页面：
	DirectoryIndex file1 file2 ...
	
8、定义路径别名：
	
	Alias  /URL/  "/PATH/TO/SOME_DIR/"
	
	DocumentRoot "/data/web"
		
		http://www.magedu.com/images/logo.jpg <-- /data/web/images/logo.jpg
		
	Alia  /images/ "/webdata/pictures/"
		http://www.magedu.com/images/logo.jpg <-- /webdata/pictures/logo.jpg
		
		alias指定的URL右侧的“/”相当于后面的路径右侧的“/”
		
9、日志设定

	错误日志：
		ErrorLog logs/error_log
		LogLevel warn
		
	访问日志：
		LogFormat：定义日志信息格式；
		CustomLog：指明日志文件路径及日志格式；
		
	格式：
		%h	Remote host
		%l	Remote logname (from identd, if supplied).
		%u	Remote user (from auth; may be bogus if return status (%s) is 401)
		%t	Time the request was received (standard english format)
		%r	First line of request
		%s	Status. For requests that got internally redirected, this is the status of the *original* request --- %>s for the last.
		%b	Size of response in bytes, excluding HTTP headers.
		%{Foobar}i	The contents of Foobar: header line(s) in the request sent to the server. 
		
		LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
		
10、httpd-manual
	# yum install httpd-manual
	
	配置文件：/etc/httpd/conf.d/manual.conf
	
	# service httpd reload
	
	访问路径：
		http://SERVER_IP/manual/
		
11、基于用户的访问控制机制

	http协议的认证功能：
		认证质询：
			WWW-Authenticate：响应码为401，拒绝客户端请求，并说明要求客户端提供账号和密码；
			
		认证：
			Authorization：客户端填入账号和密码后再次发送请求报文；认证通过后，服务端发送响应资源； 
			
	认证的方式有两种：
		basic：明文
		digest：消息摘要认证
		
	虚拟账号：仅用于访问某服务的账号和密码；
		存储于何处（httpd要有相应的适配模块）？
			文本文件
			SQL数据库
			ldap目录数据库
			
	安全域：需要用户认证后方能访问的资源集合；通常基于名称对其进行标识；
	
	basic认证的配置示例：
		(1) 定义安全域
			<Directory "/PATH/TO/SOME_DIR">
				Options  None
				AllowOverride None
				AuthType Basic
				AuthName "SOME_STRING_HERE"
				AutuUserFile "/PATH/TO/HT_PASSWD_FILE"
				Require user user1 user2 ...
			</Directory>
			
				Require valid-user
		
		(2) 创建账号文件
			htpasswd [options]  /PATH/TO/HT_PASSWD_FILE  USERNAME
				-c：创建文件；
				-m：md5加密密码； 
				-s: SHA加密密码；
				-D：删除指定用户
				
		也可于基于组账号进行认证：
			(1) 定义安全域
				<Directory "/PATH/TO/SOME_DIR">
					Options  None
					AllowOverride None
					AuthType Basic
					AuthName "SOME_STRING_HERE"
					AuthUserFile "/PATH/TO/HT_PASSWD_FILE"
					AuthGroupFile "/PATH/TO/HT_GROUP_FILE"
					Require group grp1 grp2 ...
				</Directory>
			
			(2) 创建用户账号密码文件完成后，还得将用户定义为组（创建组账号文件）
				组文件：每行定义一个组
					GROUP_NAME: username1 username2 ...
					
12、虚拟主机 
	多个站点基于不同的信息进行标识；
	
	站点标识：
		IP相同，端口不同；
		IP不同，端口相同；
		FQDN不同；
		
	虚拟主机就有了三种实现方式：
		基于IP的虚拟主机：
			每个虚拟主机使用一独有的IP地址；
		基于PORT的虚拟主机：
			每个虚拟主机使用一个独有的PORT；
		基于FQDN的虚拟主机：
			每个虚拟主机使用一个独有的FQDN；
			
	注意：虚拟主机与“主服务器”不能够同时使用；
		
		虚拟主机的配置方法：
			<VirtualHost IP:PORT>
				ServerName
				DocumentRoot 
			</VirtualHost>
			
			其它常用指令：
				Errorlog
				CusomLog
				Alias
				ServerAlias
				...
				
		基于IP的虚拟主机示例：
			前提：本机需要配置了用到的所有IP地址；
			
			<VirtualHost 172.16.100.71:80>
				ServerName www1.magedu.com
				DocumentRoot /data/vhosts/www1
			</VirtualHost>
			<VirtualHost 172.16.100.72:80>
				ServerName www2.magedu.com
				DocumentRoot /data/vhosts/www2
			</VirtualHost>
			
		基于端口的虚拟主机示例：
			前提：httpd监听了用到的所有端口；
			
			<VirtualHost 172.16.100.71:80>
				ServerName www1.magedu.com
				DocumentRoot /data/vhosts/www1
			</VirtualHost>
			<VirtualHost 172.16.100.71:8080>
				ServerName www2.magedu.com
				DocumentRoot /data/vhosts/www2
			</VirtualHost>						
					
		基于FQDN的虚拟主机示例：
			NameVirtualHost 172.16.100.71:80
			
			<VirtualHost 172.16.100.71:80>
				ServerName www1.magedu.com
				DocumentRoot /data/vhosts/www1
			</VirtualHost>
			<VirtualHost 172.16.100.71:80>
				ServerName www2.magedu.com
				DocumentRoot /data/vhosts/www2
			</VirtualHost>					
			
课外作业：
	写一个脚本，批量生成10个虚拟主机配置：
		/etc/httpd/conf.d/vhosts#.conf
			主机名：www#
			目录：/data/vhosts/www#
			访问日志：logs/www#-access_log 
			
		接受命令行参数，作为命令和主机名传递；
			
		使用函数：
			列出：list [-a|vhost_name]
			创建：create vhost_name
			删除：delete [-a|vhost_name]

深度学习：						
			
回顾：
httpd配置：
listen [address:]port
keepalive
MPM：
prefork, workder, event
DSO：
LoadModule mod_name mod_path
httpd -l|-M
DirectoryIndex 
DocumentRoot
Alias
<Directory "">
...
</Directory>
<Location "">
...
<Location>
ErrorLog, LogLevel
CustomLog, LogFormat
%{foo}i
基于用户的访问控制机制
虚拟主机：
IP，PORT，FQDN

httpd(3)

http协议：http/1.0, http/1.1, http/2.0

http协议的事务：request/response
stateless：无状态
cookie：
	Set-Cookie：
	Cookie：
session

url：Uniform Resource Locator 
URL scheme：协议，http, https, ftp, 
服务器地址：IP:PORT
资源路径：/path/to/resource
DocumentRoot

http://www.magedu.com/images/logo.jpg

基本语法格式：
<scheme>://<user>:<password>@<host>[:<port>]/<path>;<params>?<query>#<frag>
	params：参数
		http://www.magedu.com/admin/admin.php;username=admin
	query：查询
		http://www.magedu.com/admin/admin.php?logintime=12/07/2018
	frag：
		http://www.magedu.com/admin/readme.html#ch1
		
http协议报文格式：
通用格式：
起始行
Name: Value
Name: Value
...

主体 


request：
<method> <request-URL> <version>
<headers>

<body>

response：
<version> <status> <reason-phrase>
<headers>

<body>

method：请求方法，标明客户端希望服务器对资源执行的动作
GET、HEAD、POST
WebDAV：PUT、DELETE
OPTIONS、TRACE

协议查看或分析工具：tcpdump, wireshark, tshark, ...

status：状态码，status code
三位数字，1xx, 2xx, ..., 5xx
标明请求处理过程的结果状态； 

1xx: 100-101, 信息提示；
2xx：200-206，成功类的响应码，例如200；
3xx：300-305，重定向类的响应码，例如301, 302, 304等 ；
4xx：400-415, 错误类信息，客户端错误，例如 401, 404, 403等 ；
5xx：500-505, 服务器端错误，例如500，502等；

headers：
媒体格式：MIME
	major/minor
		images/jpeg
		text/html
		text/plaintext
		text/xml
		...
	
格式：
	Name: Value
	
首部分类：
	通用首部
	请求首部
	响应首部
	实体首部
	扩展首部
	
通用首部：请求、响应报文均可使用；
	Date：报文的创建时间；
	Connction：连接状态，keep-alive, close；
	Via：经由，报文传输经由的代理节点；
	Cache-Control：缓存控制
	Pragma：
	
	
	
请求首部：
	Accept：可接受的媒介类型；MIME
	Accept-Charset：接受字符集格式；
	Accept-Encoding：接受的编码格式，deflate, gzip, ...
	Accept-language：接受的语言；
	
	Client-IP：
	Host：请求的服务器名称和端口；
	Referer：包含当前正在请求的资源的上一级资源； 
	User-Agent：客户端代理；
	
	条件式请求首部：
		Expect：
		If-Modified-Since:
		If-Unmodified-Since:
		If-None-Match：本地缓存中存储的文件的ETag的值是否与服务器对应的资源的不相同；
		If-Match:
		
	安全请求首部：
		Authrization：向服务器发送账号和密码；
		Cookie：向服务器端发送Cookie信息；
		Cookie2：
		
响应首部：
	信息性首部：
		Age：响应持续时长
		Server：服务器端软件程序的名称、版本、...
		
	协商首部：
		Accept-Ranges：服务器端可接受的请求类型范围；
		Vary：服务器查看的其它首部列表； 
		
	安全响应首部：
		Set-Cookie：为客户端设定Cookie
		Set-Cookie2:
		WWW-Authenticat：认证质询
		
实体首部：
	Allow：对此实体可使用的请求方法；
	Location：资源的真正地址；
	
	Content-Encoding:
	Content-language:
	Content-Length:
	Content-Location:
	Content-Type
	
	缓存相关：
		Etag:
		Expires：实体过期时间；
		Last-Modified: 最近一次的修改时间；

扩展首部：
	X-Forwared-For
	...
	
《HTTP权威指南》前四章；


httpd-2.2基本配置

13、status页面
	LoadModule  status_module  modules/mod_status.so
	
	<Location /server-status>
		SetHandler server-status
		Order allow,deny
		Allow from 172.16
	</Location>	
	
	ExtendedStatus {On|Off}

14、curl命令

curl是基于URL语法在命令行方式下工作的文件传输工具，它支持FTP, FTPS, HTTP, HTTPS, GOPHER, TELNET, DICT, FILE及LDAP等协议。curl支持HTTPS认证，并且支持HTTP的POST、PUT等方法， FTP上传， kerberos认证，HTTP上传，代理服务器， cookies， 用户名/密码认证， 下载文件断点续传，上载文件断点续传, http代理服务器管道（ proxy tunneling）， 甚至它还支持IPv6， socks5代理服务器,，通过http代理服务器上传文件到FTP服务器等等，功能十分强大。			

curl  [options]  [URL...]

curl的常用选项：

    -A/--user-agent <string> 设置用户代理发送给服务器
    --basic 使用HTTP基本认证
    --tcp-nodelay 使用TCP_NODELAY选项
    -e/--referer <URL> 来源网址
    --cacert <file> CA证书 (SSL)
    --compressed 要求返回是压缩的格式
    -H/--header <line>自定义首部信息传递给服务器
    -I/--head 只显示响应报文首部信息
    --limit-rate <rate> 设置传输速度
    -u/--user <user[:password]>设置服务器的用户和密码
    -0/--http1.0 使用HTTP 1.0	

用法：curl [options] [URL...]

另一个工具：elinks
	elinks  [OPTION]... [URL]...
		-dump: 不进入交互式模式，而直接将URL的内容输出至标准输出； 

15、user/group

指定以哪个用户的身份运行httpd服务进程；
	User apache
	Group apache

SUexec
		
16、使用mod_deflate模块压缩页面优化传输速度

适用场景：
	(1) 节约带宽，额外消耗CPU；同时，可能有些较老浏览器不支持；
	(2) 压缩适于压缩的资源，例如文件文件；

SetOutputFilter DEFLATE

# mod_deflate configuration


# Restrict compression to these MIME types
AddOutputFilterByType DEFLATE text/plain 
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/x-javascript
AddOutputFilterByType DEFLATE text/javascript
AddOutputFilterByType DEFLATE text/css

c

17、https,  http over ssl 

SSL会话的简化过程
	(1) 客户端发送可供选择的加密方式，并向服务器请求证书；
	(2) 服务器端发送证书以及选定的加密方式给客户端；
	(3) 客户端取得证书并进行证书验正：
		如果信任给其发证书的CA：
			(a) 验正证书来源的合法性；用CA的公钥解密证书上数字签名；
			(b) 验正证书的内容的合法性：完整性验正
			(c) 检查证书的有效期限；
			(d) 检查证书是否被吊销；
			(e) 证书中拥有者的名字，与访问的目标主机要一致；
	(4) 客户端生成临时会话密钥（对称密钥），并使用服务器端的公钥加密此数据发送给服务器，完成密钥交换；
	(5) 服务用此密钥加密用户请求的资源，响应给客户端；

	注意：SSL会话是基于IP地址创建；所以单IP的主机上，仅可以使用一个https虚拟主机；

回顾几个术语：PKI，CA，CRL，X.509 (v1, v2, v3)

配置httpd支持https：
	(1) 为服务器申请数字证书；
		测试：通过私建CA发证书
			(a) 创建私有CA
			(b) 在服务器创建证书签署请求
			(c) CA签证
			
	(2) 配置httpd支持使用ssl，及使用的证书；
		# yum -y install mod_ssl

		配置文件：/etc/httpd/conf.d/ssl.conf
			DocumentRoot
			ServerName
			SSLCertificateFile
			SSLCertificateKeyFile
			
	(3) 测试基于https访问相应的主机；
		# openssl  s_client  [-connect host:port] [-cert filename] [-CApath directory] [-CAfile filename]

思考（课外作业）：
1、只提供https服务；如何处理用户习惯地对http的请求？

回顾：
http协议：
url、status、method、headers
httpd-2.2：
mod_status、user/group、mod_ssl(https)、mod_deflate

httpd应用(4)

18、httpd自带的工具程序

htpasswd：basic认证基于文件实现时，用到的账号密码文件生成工具；
	htdbm/htdigest；
apachectl：httpd自带的服务控制脚本，支持start和stop；

apxs：由httpd-devel包提供，扩展httpd使用第三方模块的工具；APache eXtenSion tool

rotatelogs：日志滚动工具；
	access.log -->
		access.log, access.1.log  -->
			access.log, acccess.1.log, access.2.log
			
suexec：访问某些有特殊权限配置的资源时，临时切换至指定用户身份运行；

ab： apache bench

19、httpd的压力测试工具

ab, webbench, http_load,  seige

jmeter, loadrunner

tcpcopy：网易，复制生产环境中的真实请求，并将之保存下来；

ab  [OPTIONS]  URL
	-n：总请求数；
	-c：模拟的并发数；
	-k：以持久连接模式 测试；
	
httpd-2.4：

新特性：
(1) MPM支持DSO机制，以模块方式加载；
(2) event MPM生产可用；
(3) 异步读写机制；
(4) 支持每模块及每目录单独日志级别定义；
(5) 每请求相关的专用配置；
(6) 增强版的表达式分析器；
(7) 毫秒级的持久连接时长；
(8) 基于FQDN的虚拟主机不再需要专门的指令NameVirtualHost；
(9) 支持用户自定义变量；
(10) 更低内在的开销； 

新模块：
(1) mod_proxy_fcgi
(2) mod_proxy_scgi
(3) mod_remoteip

安装httpd-2.4：

apr: apache porable runtime

依赖于apr-1.4+， apr-utils-1.4+, [apr-iconv]

CentOS 6：
	前提：开发包组；
	
	编译安装httpd-2.4；
	
CentOS 7：
	# yum install httpd
	
	配置环境：
		/etc/httpd/conf/httpd.conf
		模块配置文件：/etc/httpd/conf.modules.d/*.conf
		分段配置文件：/etc/httpd/conf.d/*.conf 
		
	特有配置：
		(1) 切换MPM；
			编辑配置文件/etc/httpd/conf.modules.d/00-mpm.conf
				LoadModule mpm_MPMNAME_module modules/..
				
		(2) 基于IP地址的访问控制
			统一使用Require指令进行，不再支持使用allow, deny等 ；
			
			允许所有主机访问：Require all granted
			拒绝所有主机访问：Require all denied
			
			<RequireAll>
				Require ip 172.16.100.67
				Require all denied
			</RequireAll>
			
			控制特定的客户端IP地址访问：
				Require ip IPADDR：授权
				Require not ip IPADDR：拒绝
				
			控制特定的主机名访问：
				Require host HOSTNAME
				Require not host HOSTNAME
					HOSTNAME：
						FQDN：单个主机名
						domain.tld：域名内的所有主机
				
		(3) 虚拟主机
			基于FQDN的虚拟主机不再需要专门的指令NameVirtualHost; 
			
				<VirtualHost *:80>
					ServerName www1.magedu.com
					DocumentRoot /data/vhosts/www1
					<Directory "/data/vhosts/www1">
						Options None
						AllowOverride None
						Require all granted
					</Directory>
				</VirtualHost>						
	
		(4) 毫秒级长连接
			KeepAliveTimeout #ms

			/etc/httpd/conf.d/ka.conf
				KeepAlive On
				KeepAliveTimeout 500ms
				MaxKeepAliveRequests 100						

		(5) https
		
博客作业：分别使用httpd-2.2和httpd-2.4实现
1、建立httpd服务，要求：
(1) 提供两个基于名称的虚拟主机www1, www2；有单独的错误日志和访问日志； 
(2) 通过www1的/server-status提供状态信息，且仅允许tom用户访问；
(3) www2不允许192.168.0.0/24网络中任意主机访问；

2、为上面的第2个虚拟主机提供https服务；

LAMP组合：

Web资源的类型：
静态资源：原始形式与响应结果一致；
动态资源：原始形式通常为程序文件，需要运行后将运行结果呈现给用户；
	客户端技术：js
	服务器端技术：php, jsp
	
CGI：Common Gateway Interface
可以让一个客户端，从客户端代理向运行在网络服务器上程序传输数据；CGI描述了客户端和服务器程序之间传输数据的一种标准；

请求流程：
	Client --(http) --> httpd --> (cgi) --> application process (code) --> (mysql) --> mysqld (mariadb)

程序=指令+数据
	数据模型：层次、网状、关系；
	
	指令：代码文件；
	数据：数据存储系统、文件、...
	
php：脚本编程语言、专为web开发而设计、将代码嵌入到html中；

LAMP：
A: httpd
M: mysql或mariadb
P: php/perl/python/ruby
		
	
		
关于PHP

一、PHP简介
	
PHP是通用服务器端脚本编程语言，其主要用于web开发以实现动态web页面，它也是最早实现将脚本嵌入HTML源码文档中的服务器端脚本语言之一。同时，php还提供了一个命令行接口，因此，其也可以在大多数系统上作为一个独立的shell来使用。

Rasmus Lerdorf于1994年开始开发PHP，它是初是一组被Rasmus Lerdorf称作“Personal Home Page Tool” 的Perl脚本， 这些脚本可以用于显示作者的简历并记录用户对其网站的访问。后来，Rasmus Lerdorf使用C语言将这些Perl脚本重写为CGI程序，还为其增加了运行Web forms的能力以及与数据库交互的特性，并将其重命名为“Personal Home Page/Forms Interpreter”或“PHP/FI”。此时，PHP/FI已经可以用于开发简单的动态web程序了，这即是PHP 1.0。1995年6月，Rasmus Lerdorf把它的PHP发布于comp.infosystems.www.authoring.cgi Usenet讨论组，从此PHP开始走进人们的视野。1997年，其2.0版本发布。

1997年，两名以色列程序员Zeev Suraski和Andi Gutmans重写的PHP的分析器(parser)成为PHP发展到3.0的基础，而且从此将PHP重命名为PHP: Hypertext Preprocessor。此后，这两名程序员开始重写整个PHP核心，并于1999年发布了Zend Engine 1.0，这也意味着PHP 4.0的诞生。2004年7月，Zend Engine 2.0发布，由此也将PHP带入了PHP 5时代。PHP5包含了许多重要的新特性，如增强的面向对象编程的支持、支持PDO(PHP Data Objects)扩展机制以及一系列对PHP性能的改进。

二、PHP Zend Engine

Zend Engine是开源的、PHP脚本语言的解释器，它最早是由以色列理工学院(Technion)的学生Andi Gutmans和Zeev Suraski所开发，Zend也正是此二人名字的合称。后来两人联合创立了Zend Technologies公司。

Zend Engine 1.0于1999年随PHP 4发布，由C语言开发且经过高度优化，并能够做为PHP的后端模块使用。Zend Engine为PHP提供了内存和资源管理的功能以及其它的一些标准服务，其高性能、可靠性和可扩展性在促进PHP成为一种流行的语言方面发挥了重要作用。

Zend Engine的出现将PHP代码的处理过程分成了两个阶段：首先是分析PHP代码并将其转换为称作Zend opcode的二进制格式(类似Java的字节码)，并将其存储于内存中；第二阶段是使用Zend Engine去执行这些转换后的Opcode。

三、PHP的Opcode

Opcode是一种PHP脚本编译后的中间语言，就像Java的ByteCode,或者.NET的MSL。PHP执行PHP脚本代码一般会经过如下4个步骤(确切的来说，应该是PHP的语言引擎Zend)：
1、Scanning(Lexing) —— 将PHP代码转换为语言片段(Tokens)
2、Parsing —— 将Tokens转换成简单而有意义的表达式
3、Compilation —— 将表达式编译成Opocdes
4、Execution —— 顺次执行Opcodes，每次一条，从而实现PHP脚本的功能

	hhvm

	扫描-->分析-->编译-->执行

四、php的加速器

基于PHP的特殊扩展机制如opcode缓存扩展也可以将opcode缓存于php的共享内存中，从而可以让同一段代码的后续重复执行时跳过编译阶段以提高性能。由此也可以看出，这些加速器并非真正提高了opcode的运行速度，而仅是通过分析opcode后并将它们重新排列以达到快速执行的目的。

常见的php加速器有：

1、APC (Alternative PHP Cache)
遵循PHP License的开源框架，PHP opcode缓存加速器，目前的版本不适用于PHP 5.4。项目地址，http://pecl.php.net/package/APC。

2、eAccelerator
源于Turck MMCache，早期的版本包含了一个PHP encoder和PHP loader，目前encoder已经不在支持。项目地址， http://eaccelerator.net/。

3、XCache
快速而且稳定的PHP opcode缓存，经过严格测试且被大量用于生产环境。项目地址，http://xcache.lighttpd.net/

4、Zend Optimizer和Zend Guard Loader
Zend Optimizer并非一个opcode加速器，它是由Zend Technologies为PHP5.2及以前的版本提供的一个免费、闭源的PHP扩展，其能够运行由Zend Guard生成的加密的PHP代码或模糊代码。 而Zend Guard Loader则是专为PHP5.3提供的类似于Zend Optimizer功能的扩展。项目地址，http://www.zend.com/en/products/guard/runtime-decoders

5、NuSphere PhpExpress
NuSphere的一款开源PHP加速器，它支持装载通过NuSphere PHP Encoder编码的PHP程序文件，并能够实现对常规PHP文件的执行加速。项目地址，http://www.nusphere.com/products/phpexpress.htm

五、PHP源码目录结构

PHP的源码在结构上非常清晰。其代码根目录中主要包含了一些说明文件以及设计方案，并提供了如下子目录：

1、build —— 顾名思义，这里主要放置一些跟源码编译相关的文件，比如开始构建之前的buildconf脚本及一些检查环境的脚本等。
2、ext —— 官方的扩展目录，包括了绝大多数PHP的函数的定义和实现，如array系列，pdo系列，spl系列等函数的实现。 个人开发的扩展在测试时也可以放到这个目录，以方便测试等。
3、main —— 这里存放的就是PHP最为核心的文件了，是实现PHP的基础设施，这里和Zend引擎不一样，Zend引擎主要实现语言最核心的语言运行环境。
4、Zend —— Zend引擎的实现目录，比如脚本的词法语法解析，opcode的执行以及扩展机制的实现等等。
5、pear —— PHP 扩展与应用仓库，包含PEAR的核心文件。
6、sapi —— 包含了各种服务器抽象层的代码，例如apache的mod_php，cgi，fastcgi以及fpm等等接口。
7、TSRM —— PHP的线程安全是构建在TSRM库之上的，PHP实现中常见的*G宏通常是对TSRM的封装，TSRM(Thread Safe Resource Manager)线程安全资源管理器。
8、tests —— PHP的测试脚本集合，包含PHP各项功能的测试文件。
9、win32 —— 这个目录主要包括Windows平台相关的一些实现，比如sokcet的实现在Windows下和*Nix平台就不太一样，同时也包括了Windows下编译PHP相关的脚本。
				
httpd与php结合方式：
CGI 
module （把php编译成为httpd的扩展模块）
	MPM：
		prefork: libphp5.so
		event, worker: libphp5-zts.so
FastCGI：
	fpm
	
LAMP的实现方式：
	httpd(prefork)+libphp5.so+mysql
	httpd(event)+libphp5-zts.so+mysql
	httpd+fpm(php)+mysql
	
CentOS 6：
	mysql-server-5.1
CentOS 7
	mariadb-server-5.5
	
MySQL的命令行客户端程序：mysql
	-uUSERNAME
	-hHOST
	-pPASSWORD
	
	支持SQL语句对数据完成管理：
		DDL，DML
			DDL：CREATE, ALTER, DROP
			DML：INSERT，DELETE，SELECT，UPDATE
			
	mysql> GRANT ALL ON db_name.tbl_name TO username@'host' IDENTIFIED BY 'password';
		_：任意单个字符
		%：任意长度的任意字符
		
	管理员账号：root/localhost, 密码默认为空；
	
	配置文件：/etc/my.cnf, /etc/my.cnf.d/*.cnf
		[mysqld]
		innodb_file_per_table = ON
		skip_name_resolve = ON
		
	安装完成后，建议运行 mysql_secure_installation一次；

php的安装：
	httpd(prefork)
	# yum install php 
	
	配置文件：
		/etc/php.ini, /etc/php.d/*.ini 
	
	测试php：
		<?php 
			phpinfo();
		?>
	
	测试mysql连接性：
		<?php
			$conn = mysql_connect('172.16.100.71','testuser','testpass');
			if($conn)
				echo "OK";
			else
				echo "Failure";
		?>				
	
CentOS 6：
	# yum install httpd php php-mysql mysql-server
	# service mysqld start
	# service httpd start 
	
CentOS 7：
	# yum install httpd php php-mysql mariadb-server
	# systemctl start mariadb.service httpd.service 
	
实践作业：使用两个虚拟主机，分别部署安装wordpress, phpwind;

	
	



回顾： httpd, lamp

httpd: mod_deflate, https, 
amp： 
静态资源：Client -- http --> httpd
动态资源：Client -- http --> httpd --> libphp5.so ()
动态资源：Client -- http --> httpd --> libphp5.so () -- mysql --> MySQL server

httpd+php:
modules: 把php编译成为httpd的模块
cgi：
fastcgi：

php：zend engine
编译：Opcode是一种PHP脚本编译后的中间语言
执行：

Scanning --> Parsing --> Compilation --> Execution

加速器：APC，eAccelerator, Xcache

LAMP(2) 

快速部署amp：
CentOS 7:
Modules：程序包，httpd, php, php-mysql, mariadb-server
FastCGI：程序包，httpd, php-fpm, php-mysql, mariadb-server
CentOS 6：
httpd, php, php-mysql, mysql-server

php：
脚本语言解释器
配置文件：/etc/php.ini,  /etc/php.d/*.ini 

配置文件在php解释器启动时被读取，因此，对配置文件的修改如何生效？
	Modules：重启httpd服务；
	FastCGI：重启php-fpm服务；

ini：
	[foo]：Section Header
	directive = value
	
	注释符：较新的版本中，已经完全使用;进行注释；
	#：纯粹的注释信息
	;：用于注释可启用的directive
	
	php.ini的核心配置选项文档：  http://php.net/manual/zh/ini.core.php
	php.ini配置选项列表：http://php.net/manual/zh/ini.list.php
	
<?php 
...php code...
?>

mariadb(mysql)：

数据模型：层次模型、网状模型、关系模型

关系模型：
	二维关系：
		表：row, column
		索引：index
		视图：view
		
	SQL接口：Structured Query Language
		类似于OS的shell接口；也提供编程功能；
		
		ANSI： SQL标准，SQL-86, SQL-89, SQL-92, SQL-99, SQL-03, ...
			xml
			
		DDL：Data Defined Language
			CREATE, ALTER, DROP
		DML: Data Manapulating Language
			INSERT, DELETE, UPDATE, SELECT
			
		编程接口：选择、循环；
		
		SQL代码：
			存储过程：procedure
			存储函数：function
			触发器：trigger
			事件调度器：event scheduler
			
			例程：routine
			
	用户和权限：
		用户：用户名和密码；
		权限：管理类、数据库、表、字段
		
DBMS：DataBase Management System
	RDBMS：Relational
		
	MySQL：单进程，多线程 
		用户连接：通过线程来实现；
			线程池：

事务(Transaction)：组织多个操作为一个整体，要么全部都执行，要么全部都不执行；
“回滚”， rollback

Bob:8000, 8000-2000
Alice:5000, 5000+2000

一个存储系统是否支持事务，测试标准：
	ACID：
		A：原子性；
		C：一致性；
		I：隔离性；
		D：持久性；

SQL接口：分析器和优化器
存储引擎：


补充材料：RDMBS设计范式基础概念

设计关系数据库时，遵从不同的规范要求，设计出合理的关系型数据库，这些不同的规范要求被称为不同的范式，各种范式呈递次规范，越高的范式数据库冗余越小。

目前关系数据库有六种范式：第一范式（1NF）、第二范式（2NF）、第三范式（3NF）、巴德斯科范式（BCNF）、第四范式(4NF）和第五范式（5NF，又称完美范式）。满足最低要求的范式是第一范式（1NF）。在第一范式的基础上进一步满足更多规范要求的称为第二范式（2NF），其余范式以次类推。一般说来，数据库只需满足第三范式(3NF）就行了。

(1) 第一范式（1NF）

所谓第一范式（1NF）是指在关系模型中，对域添加的一个规范要求，所有的域都应该是原子性的，即数据库表的每一列都是不可分割的原子数据项，而不能是集合，数组，记录等非原子数据项。即实体中的某个属性有多个值时，必须拆分为不同的属性。在符合第一范式（1NF）表中的每个域值只能是实体的一个属性或一个属性的一部分。简而言之，第一范式就是无重复的域。

说明：在任何一个关系数据库中，第一范式（1NF）是对关系模式的设计基本要求，一般设计中都必须满足第一范式（1NF）。不过有些关系模型中突破了1NF的限制，这种称为非1NF的关系模型。换句话说，是否必须满足1NF的最低要求，主要依赖于所使用的关系模型。

(2) 第二范式(2NF)

第二范式（2NF）是在第一范式（1NF）的基础上建立起来的，即满足第二范式（2NF）必须先满足第一范式（1NF）。第二范式（2NF）要求数据库表中的每个实例或记录必须可以被唯一地区分。选取一个能区分每个实体的属性或属性组，作为实体的唯一标识。

第二范式（2NF）要求实体的属性完全依赖于主关键字。所谓完全依赖是指不能存在仅依赖主关键字一部分的属性，如果存在，那么这个属性和主关键字的这一部分应该分离出来形成一个新的实体，新实体与原实体之间是一对多的关系。为实现区分通常需要为表加上一个列，以存储各个实例的唯一标识。简而言之，第二范式就是在第一范式的基础上属性完全依赖于主键。

(3) 第三范式（3NF）

第三范式（3NF）是第二范式（2NF）的一个子集，即满足第三范式（3NF）必须满足第二范式（2NF）。简而言之，第三范式（3NF）要求一个关系中不能包含已在其它关系已包含的非主关键字信息。简而言之，第三范式就是属性不依赖于其它非主属性，也就是在满足2NF的基础上，任何非主属性不得传递依赖于主属性。

数据库：数据集合
表：为了满足范式设计要求，将一个数据集分拆为多个；

约束：constraint，向数据表插入的数据要遵守的限制规则；
	主键：一个或多个字段的组合，填入主键中的数据，必须不同于已存在的数据；不能为空；
	外键：一个表中某字段中能插入的数据，取决于另外一张表的主键中的数据；
	惟一键：一个或多个字段的组合，填入惟一键中的数据，必须不同于已存在的数据；可以为空；
	检查性约束：取决于表达式的要求；
	
索引：将表中的某一个或某些字段抽取出来，单独将其组织一个独特的数据结构中；
	常用的索引类型：
		树型：
		hash：
		
	注意：有助于读请求，但不利于写请求；
	
关系运算：
	选择：挑选出符合条件的行；
	投影：挑选出符合需要的列；
	连接：将多张表关联起来；
		
数据抽象：
	物理层：决定数据的存储格式，即如何将数据组织成为物理文件；
	逻辑层：描述DB存储什么数据，以及数据间存在什么样的关系；
	视图层：描述DB中的部分数据；
	
关系模型的分类：
	关系模型
	实体-关系模型
	基于对象的关系模型
	半结构化关系模型
	
MariaDB(mysql)：

Unireg

MySQL AB  --> MySQL
	Solaris：二进制版本；
	
	www.mysql.com
	
c

MariaDB的特性：
	插件式存储引擎：存储管理器有多种实现版本，彼此间的功能和特性可能略有区别；用户可根据需要灵活选择； 
	
	存储引擎也称为“表类型”；
	
	(1) 更多的存储引擎；
		MyISAM：不支持事务；
		MyISAM --> Aria
		InnoDB --> XtraDB 
			：支持事务；
	(2) 诸多扩展和新特性；
	(3) 提供了较多的测试组件；
	(4) truly open source；
	
MySQL的发行机制：
	Enterprise：提供了更丰富的功能；
	Community：
	
安装和使用MariaDB：
	
	安装方式：
		(1) rpm包；
			(a) 由OS的发行商提供；
			(b) 程序官方提供；
		(2) 源码包；
		(3) 通用二进制格式的程序包；
		
	通用二进制格式安装MariaDB：
		(1) 准备数据目录；
			以/mydata/data目录为例；
		(2) 安装配置mariadb						
			# useradd  -r  mysql
			# tar xf  mariadb-VERSION.tar.xz  -C  /usr/local
			# cd /usr/local
			# ln  -sv  mariadb-VERSION  mysql
			# cd  /usr/local/mysql
			# chown  -R  root:mysql  ./*
			# scripts/mysql_install_db  --user=mysql  -datadir=/mydata/data
			# cp  support-files/mysql.server   /etc/init.d/mysqld
			# chkconfig   --add  mysqld
		(3) 提供配置文件
			ini格式的配

置文件；各程序均可通过此配置文件获取配置信息；
				[program_name]
								
			OS Vendor提供mariadb rpm包安装的服务的配置文件查找次序：
				/etc/mysql/my.cnf  --> /etc/my.cnf  --> --default-extra-file=/PATH/TO/CONF_FILE  --> ~/.my.cnf
				
			通用二进制格式安装的服务程序其配置文件查找次序：
				/etc/my.cnf  --> /etc/mysql/my.cnf  --> --default-extra-file=/PATH/TO/CONF_FILE  --> ~/.my.cnf
				
			获取其读取次序的方法：
				mysqld  --verbose  --help
				
			# cp  support-files/my-large.cnf  /etc/my.cnf
			
			添加三个选项：
				datadir = /mydata/data
				innodb_file_per_table = ON
				skip_name_resolve = ON
				
		(4) 启动服务
			# service  mysqld  start
			
回顾：MariaDB的基础
关系：二维关系，表（行、列）
设计范式：
第一范式：字段是原子性的；
第二范式：存在可用主键；
第三范式：任何都不应该依赖于其它表的非主属性；
约束：主键、惟一键、外键、检查性约束；
MariaDB安装方式：
包管理器(rpm, deb)
通用二进制格式；
源码编译安装；
SQL：
数据库、表、索引、视图、存储过程、存储函数、触发器、事件调度器、用户和权限；
元数据数据库：mysql

DDL, DML
DDL: CREATE, ALTER, DROP
DML:  INSERT, DELETE, UPDATE, SELECT
DCL: GRANT, REVOKE

MariaDB的基础(2)

MariaDB程序的组成：
C：Client
mysql：CLI交互式客户端程序；
mysqldump：备份工具；
mysqladmin：管理工具；
mysqlbinlog：
...
S：Server
mysqld
mysqld_safe：建议运行服务端程序；
mysqld_multi：多实例；

三类套接字地址：
	IPv4, 3306/tcp
	Unix Sock：/var/lib/mysql/mysql.sock, /tmp/mysql.sock
		C <--> S: localhost, 127.0.0.1
		
命令行交互式客户端程序：mysql
mysql 
mysql [OPTIONS] [database]

常用选项：
	-uUSERNAME：用户名，默认为root；
	-hHOST：远程主机（即mysql服务器）地址，默认为localhost; 
	-p[PASSWORD]：USERNAME所表示的用户的密码； 默认为空；
	
	注意：mysql的用户账号由两部分组成：'USERNAME'@'HOST'; 其中HOST用于限制此用户可通过哪些远程主机连接当前的mysql服务；
		HOST的表示方式，支持使用通配符：
			%：匹配任意长度的任意字符；
				172.16.%.%,  172.16.0.0/16
			_：匹配任意单个字符；
			
	-Ddb_name：连接到服务器端之后，设定其处指明的数据库为默认数据库；
	-e 'SQL COMMAND;'：连接至服务器并让其执行此命令后直接返回；
	
命令：
	客户端命令：本地执行
		mysql> help
			\u db_name：设定哪个库为默认数据库
			\q：退出；
			\d CHAR：设定新的语句结束符；
			\g：语句结束标记；
			\G：语句结束标记，结果竖排方式显式；
			\s：
	服务端命令：通过mysql连接发往服务器执行并取回结果；
		DDL， DML， DCL 
		
		注意：每个语句必须有语句结束符，默认为分号(;)
		
数据类型：
	表：行和列
		创建表：定义表中的字段；
		
	定义字段时，关键的一步即为确定其数据类型；
		用于确定：数据存储格式、能参与运算种类、可表示的有效的数据范围；
		
	字符型：字符集 
		码表：在字符和二进制数字之间建立映射关系；
		
	种类：
		字符型：
			定长字符型：
				CHAR(#)：不区分字符大小写
				BINARY(#)：区分字符大小写
			变长字符型：
				VARCHAR(#)
				VARBINARY(#)
			对象存储：
				TEXT
				BLOB
			内置类型：
				SET 
				ENUM						
		数值型：
			精确数值型：
				INT（TINYINT，SMALLINT，MEDIUMINT，INT，BIGINT） 
			近似数值型：
				FLOAT
				DOBULE
		日期时间型：
			日期型：DATE
			时间型：TIME
			日期时间型：DATETIME
			时间戳：TIMESTAMP
			年份：YEAR(2), YEAR(4)
			
	数据类型有修饰符：
		UNSIGNED：无符号；
		NOT NULL：非空；
		DEFAULT  value：默认值；
		
服务器端命令：
DDL：数据定义语言，主要用于管理数据库组件，例如表、索引、视图、用户、存储过程
	CREATE、ALTER、DROP
DML：数据操纵语言，主要用管理表中的数据，实现数据的增、删、改、查；
	INSERT， DELETE， UPDATE， SELECT
	
获取命令帮助：
	mysql> help  KEYWORD
	
数据库管理：
	创建：
		CREATE  {DATABASE | SCHEMA}  [IF NOT EXISTS]  db_name;
			[DEFAULT]  CHARACTER SET [=] charset_name
			[DEFAULT]  COLLATE [=] collation_name
			
		查看支持的所有字符集：SHOW CHARACTER SET 
		查看支持的所有排序规则：SHOW  COLLATION
		
	修改：
		ALTER {DATABASE | SCHEMA}  [db_name]
			[DEFAULT]  CHARACTER SET [=] charset_name
			[DEFAULT]  COLLATE [=] collation_name
			
	删除：
		DROP {DATABASE | SCHEMA} [IF EXISTS] db_name
		
	查看：
		SHOW DATABASES LIKE  ’‘;
			
表管理：
	创建：
		CREATE TABLE  [IF NOT EXISTS]  tbl_name  (create_defination)  [table_options]
		
		create_defination:
			字段：col_name  data_type
			键：
				PRIMARY KEY (col1, col2, ...)
				UNIQUE KEY  (col1, col2,...)
				FOREIGN KEY (column)
			索引：
				KEY|INDEX  [index_name]  (col1, col2,...)
				
		table_options：
			ENGINE [=] engine_name
			
		查看数据库支持的所有存储引擎类型：
			mysql> SHOW  ENGINES;
			
		查看某表的存储引擎类型：
			mysql> SHOW  TABLES  STATUS  [LIKE  'tbl_name']
		
	修改：
		ALTER [ONLINE | OFFLINE] [IGNORE] TABLE tbl_name  [alter_specification [, alter_specification] ...]
		
		alter_specification:
			字段：
				添加：ADD  [COLUMN]  col_name  data_type  [FIRST | AFTER col_name ]
				删除：DROP  [COLUMN] col_name 
				修改：
					CHANGE [COLUMN] old_col_name new_col_name column_definition  [FIRST|AFTER col_name]	
					MODIFY [COLUMN] col_name column_definition  [FIRST | AFTER col_name]
			键：
				添加：ADD  {PRIMARY|UNIQUE|FOREIGN}  KEY (col1, col2,...)
				删除：
					主键：DROP PRIMARY KEY
					外键：DROP FOREIGN KEY fk_symbol
			索引：
				添加：ADD {INDEX|KEY} [index_name]  (col1, col2,...)
				删除：DROP {INDEX|KEY}  index_name
			表选项：
				ENGINE [=] engine_name
			
		查看表上的索引的信息：
			mysql> SHOW INDEXES FROM tbl_name;
			
	删除：
		DROP  TABLE  [IF EXISTS]   tbl_name [, tbl_name] ...
		
	表的引用方式：
		tbl_name
		db_name.tbl_name

	第二种创建方式：
		复制表结构；
		
	第三种创建方式：
		复制表数据；

索引管理：
	索引是特殊的数据结构；
	
	索引：要有索引名称；
	
	创建：
		CREATE  [UNIQUE|FULLTEXT|SPATIAL] INDEX  index_name  [BTREE|HASH]  ON tbl_name (col1, col2,,...)
	
	删除：
		DROP  INDEX index_name ON tbl_name
		
DML：INSERT， DELETE， UPDATE， SELECT
	
	INSERT INTO：
		INSERT  [INTO]  tbl_name  [(col1,...)]  {VALUES|VALUE}  (val1, ...),(...),...
		
		注意：
			字符型：引号；
			数值型：不能用引号；
			
	SELECT：
		(1) SELECT  *  FROM  tbl_name;
		(2) SELECT  col1, col2, ...  FROM  tbl_name;
			显示时，字段可以显示为别名；
				col_name  AS  col_alias
		(3)  SELECT  col1, ...  FROM tbl_name  WHERE clause; 
			WHERE clause：用于指明挑选条件；
				col_name 操作符 value：
					age > 30; 
					
				操作符(1) ：
					>, <, >=, <=, ==, !=
					
				组合条件：
					and 
					or
					not
					
				操作符(2) ：
					BETWEEN ...  AND ...
					LIKE 'PATTERN'
						通配符：
							%：任意长度的任意字符；
							_：任意单个字符；
					RLIKE  'PATTERN'
						正则表达式对字符串做模式匹配；
					IS NULL
					IS NOT NULL
		(4) SELECT col1, ... FROM tbl_name  [WHERE clause]  ORDER BY  col_name, col_name2, ...  [ASC|DESC];
			ASC: 升序；
			DESC： 降序；
			
	DELETE：
		DELETE   FROM  tbl_name  [WHERE where_condition]  [ORDER BY ...]  [LIMIT row_count]
		
		(1) DELETE  FROM  tbl_name  WHERE where_condition 
		(2) DELETE  FROM  tbl_name  [ORDER BY ...]  [LIMIT row_count]
		
	UPDATE：
		UPDATE [LOW_PRIORITY] [IGNORE] table_reference  SET col_name1=value1 [, col_name2=value2] ... [WHERE where_condition]  [ORDER BY ...] [LIMIT row_count]
		
用户账号及权限管理：
	
	用户账号：'username'@'host'
		host：此用户访问当前mysql服务器时，允许其通过哪些主机远程创建连接；
			表示方式：IP，网络地址、主机名、通配符(%和_)；
			
		禁止检查主机名：my.cnf
			[mysqld]
			skip_name_resolve = ON
			
	创建用户账号：
		CREATE  USER   'username'@'host'  [IDENTIFIED BY  'password'];
		
	删除用户账号：
		DROP USER  ’user‘@’host' [, user@host] ...

	授权：
		权限级别：管理权限、数据库、表、字段、存储例程；
		
		GRANT  priv_type,...  ON  [object_type]  db_name.tbl_name  TO  'user'@'host'  [IDENTIFIED BY  'password'];
			
			priv_type： ALL  [PRIVILEGES]
			db_name.tbl_name：
				*.*：所有库的所有表；
				db_name.*：指定库的所有表；
				db_name.tbl_name：指定库的特定表；
				db_name.routine_name：指定库上的存储过程或存储函数；
			
			[object_type]
				TABLE
				FUNCTION
				PROCEDURE
			
		查看指定用户所获得的授权：
			SHOW GRANTS FOR  'user'@'host'
			
			SHOW GRANTS FOR CURRENT_USER;
			
		回收权限：
			REVOKE  priv_type, ...  ON  db_name.tbl_name  FROM  'user'@'host';
			
		注意：MariaDB服务进程启动时，会读取mysql库的所有授权表至内存中；
			(1) GRANT或REVOKE命令等执行的权限操作会保存于表中，MariaDB此时一般会自动重读授权表，权限修改会立即生效；
			(2) 其它方式实现的权限修改，要想生效，必须手动运行FLUSH PRIVILEGES命令方可；
			
加固mysql服务器，在安装完成后，运行mysql_secure_installation命令；
	
图形管理组件：
	phpMyAdmin
		运行于lamp；
	Navicat
	Mysql-Front
	ToadForMySQL
	SQLyog




		
	
	
LAMP(3)

php：编程语言；普遍需求，通用的功能模块；扩展，extensions；
程序：分解多个包
主包：php-5.3.3-
支包：name-subname-version-release.rpm 

CentOS 6：
CentOS 7：
EPEL：

性能扩展方式：
向上扩展：Scale Up
向外扩展：Scale Out 

httpd与php结合方式：
CGI：Common Gateway Interface
Module：php，编译成httpd的扩展模块；
prefork：libphp5.so
worker, event：libphp5-zts.so 
fastCGI：C/S架构，socket；

php-fpm：
CentOS 6：
httpd-2.2：默认不支持fcgi模块，需要自行编译扩展；
php-5.3.3：不支持fpm机制，需要自行打补丁编译安装；

解决方案：编译安装httpd-2.4, php-5.3.3+

CentOS 7：
httpd-2.4：rpm包默认编译安装了fcgi模块；
php-fpm：php-fpm-version.rpm包

配置文件：
	服务进程配置文件：/etc/php-fpm.conf, /etc/php-fpm.d/*.conf
	解释器配置文件：/etc/php.ini, /etc/php.d/*.ini 
	
服务进程配置文件：
	[global]：全局配置
	[pool]：连接池配置
		listen = 127.0.0.1:9000
		listen.backlog = -1
		
		listen.allowed_clients = 127.0.0.1
		
		user = 
		group = 
		
		pm = dynamic|static
		pm.max_children
		pm.start_servers
		pm.min_spare_servers
		pm.max_spare_servers
		pm.max_requests 

	pm方式的php进程存储session的路径：
		php_value[session.save_handler] = files
		php_value[session.save_path] = /var/lib/php/session
		
		# mkdir /var/lib/php/session
		# chown apache.apache /var/lib/php/session
		
	配置示例：
		<VirtualHost *:80>
			ServerName www1.magedu.com
			DocumentRoot /data/vhosts/www1
			ProxyRequests Off
			DirectoryIndex index.php
			ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/data/vhosts/www1/$1
			<Directory "/data/vhosts/www1">
				Options None
				AllowOverride None
				Require all granted
			</Directory>
		</VirtualHost>			

编译安装amp：
	httpd：httpd-2.4
	mariadb：mariadb-5.5
	php5：php-5.4 

注意：被依赖到的每个组件，在编译时主要用到的是其开发库和头文件；通常由NAME-devel-VERSION.rpm；

	开发包组：Development Tools, Server Platform Development

	CentOS 6专用：
	(1) apr
		# ./configure --prefix=/usr/local/apr
		# make && make install
	(2) apr-util
		# ./configure --prefix=/usr/local/apr-util  --with-apr=/usr/local/apr
		# make && make install
		
	CentOS 7专用：
	# yum install apr-devel apr-util-devel
	
	
	安装httpd-2.4：
	
	# yum install pcre-devel openssl-devel  libevent-devel
	
	# ./configure --prefix=/usr/local/lamp/apache24 --sysconfdir=/etc/lamp/httpd24 --enable-so --enable-ssl --enable-cgi --enable-rewrite --enable-modules=most --enable-mpms-shared=all --with-mpm=prefork --with-pcre --with-zlib --with-apr=/usr/local/lamp/apr  --with-apr-util=/usr/local/lamp/apr-util
	# make -j # 
	# make install 
				collect2: ld returned 1 exit status
				make[1]: *** [httpd] Error 1
				make[1]: Leaving directory `/testdir/httpd-2.4.6'
				make: *** [install-recursive] Error 1
	配置文件为安装目录的 httpd.conf 和 extra目录下的.conf文件 ，original是原始默认的配置文件
			httpd的程序安装位置为 --prefix=/usr/local.apache24 --sysconfdir= 配置文件的路径
				bin目录为 工具程序 
				built 编译目录 config.nice rules.mk 编译时所构建用到的指令
				cgi-bin 放cgi脚本的 cgi的网页要放在此目录下
				error 错误页
				htdocs 默认网页的存储位置 documentroot
				icons 图标
				include 头文件  二次开发 基于他编译其他文件
				logs日志文件
				man帮助文档
				manual手册页（网页格式）
				modules（编译的模块）
					vim /etc/
	
	安装php-5.4：
	# yum install libxml2-devel gd-devel freetype-devel libmcrypt-devel libpng-devel、
			gd 在Scientific support中安装依赖关系会安装 php support中也会有一下插件
	
	# ./configure --prefix=/usr/local/php54 --with-mysql=/usr/local/mysql --with-openssl --with-mysqli=/usr/local/mysql/bin/mysql_config --enable-mbstring --enable-xml --enable-sockets --with-freetype-dir --with-gd --with-libxml-dir=/usr --with-zlib --with-jpeg-dir --with-png-dir --with-mcrypt --with-apxs2=/usr/local/apache24/bin/apxs --with-config-file-path=/etc/php54.ini --with-config-file-scan-dir=/etc/php54.d
	# make -j #
	# make install
./configure --prefix=/usr/local/lamp/php55  --enable-mbstring --enable-xml --enable-sockets --with-freetyppe-dir --with-gd --with-libxml-dir=/usr --with-zlib --with-jpep-dir --with-png-dir  --with-mcrypt --with-apxs2=/usr/local/apache/bin/apxs --with-config-file-path=/etc/php55.ini --with-config-file-scan-dir=/etc/php55.d

	
	注意：如果httpd使用了线程式MPM，则编译php时应该额外使用--enable-maintainer-zts; 

	ldconfig -v 重读库文件
	/etc/ld.so.conf.d/php55.sh
	配置httpd：
	LoadModule php5_module modules/libphp5.so
	AddType application/x-httpd-php .php 
	DirectoryIndex index.php index.html
	
	部署phpMyAdmin:
	Web GUI，用于管理mysql数据库；
	
	回顾：
	
	php-fpm：
	httpd:
	ProxyRequests Off
	ProxyPassMatch
	DirectoryIndex index.php 
	
	php-fpm：
	/etc/php-fpm.conf
	/etc/php-fpm.d/*.conf
	
	systemctl start php-fpm.service 
	
	编译amp：
	httpd-2.4, php-5.5.40, mariadb-5.5.46
	
	LAMP(4):
	
	opcode加速器：
	APC，eAccelerator, Xcache, ...
	并不是优化了性能；而是通过缓存opcode可供多进程共享；
	
	xcache：
	http://xcache.lighttpd.org
	
	# /usr/local/php54/bin/phpize
	# ./configure --enable-xcache --with-php-config=/usr/local/php54/bin/php-config
	# make && make install
	
	# mkdir /etc/php54.d/
	# cp xcache.ini  /etc/php54.d/
	
	博客作业：
	(1) CentOS 7, apm+xcache, rpm包, php module;
		a) 一个虚拟主机提供phpMyAdmin，另一个虚拟主机提供wordpress；
		b) 为phpMyAdmim提供https服务；
		
	(2) CentOS 7, amp + xcache， rpm包，php-fpm；
		a) httpd, php, mariadb分别部署在一个单独的主机上；
		b) 一个虚拟主机提供phpMyAdmin，另一个虚拟主机提供wordpress；
		c) 为phpMyAdmim提供https服务；
		
	(3) CentOS 7, amp + xcache，编译安装，php-fpm；
		a) 分别深度：httpd, php, mariadb分别部署在一个单独的主机上，以及都在同一主机；
		b) 一个虚拟主机提供phpMyAdmin，另一个虚拟主机提供wordpress；
		c) 为phpMyAdmim提供https服务；
		
	(4) 对以上所有部署做压力测试，并对比测试结果，写出测试报告；
	
	
	rsyslog：
	
	日志：历史事件记录；
	事件：时间，事件，事件级别
	
	syslog：
	klogd：kernel
	syslogd：system(application)
	
	事件记录格式：
	日期时间 	主机 	进程[pid]: 事件内容
	
	C/S架构：通过TCP或UDP提供日志记录服务；
	
	rsyslog：
	rsyslogd特性：
	多线程；
	UDP，TCP，SSL，TLS，RELP；
	存储日志信息于MySQL，PGSQL，Oracle等RDBMS；
	强大的过滤器，实现过滤日志信息中任何部分的内容；
	自定义输出格式；
	
	elk stack：elasticsearch, logstash, kibina
	
	rsyslog日志收集器的重要术语：
	facility：设施，从功能或日志信息来源的程序对日志收集进行分类：
	auth, authpriv, cron, daemon, kern, lpr, mail, mark, news, security, user, uucp, syslog, local0-local7 
	priority：优先级
	debug, info, notice, warn(warning), err(error), crit(critical), alert, emerg(panic)
	
	程序包：rsyslog
	程序环境：
	配置文件：/etc/rsyslog.conf, /etc/rsyslog.d/*.conf
	主程序：rsyslogd 
	C6：service rsyslogd {start|stop|restart|status}
	C7: systemctl start|stop|restart|status rsyslog.service
	
	配置文件格式：
	由三部分组成：
		MODULES
		GLOBAL DRICTIVES
		RULES：定义日志记录规则
		
	RULES:
		facility.priority	target 
		
		priority：
			*：所有级别
			none：没有级别，不记录日志；
			PRIORITY：此级别（含）及其上所有级别的信息；
			=PRIORITY：仅记录指定的级别的日志信息；
			
		facility：
			*：所有设施；
			f1,f2,...：指定的列表；
			
		target：
			文件：将日志信息记录到指定的文件中；-表示异步写入；
			用户：将日志事件通知给指定的用户；通过将信息发送给登录到系统上的用户终端进行；
			日志服务器：@host，把日志通过网络送往指定的服务器；
			管道：| COMMAND
			
	运行rsyslog为服务器：
	# Provides UDP syslog reception
	$ModLoad imudp
	$UDPServerRun 514
	
	# Provides TCP syslog reception
	$ModLoad imtcp
	$InputTCPServerRun 514
	
	其它的日志文件：
	/var/log/btmp：当前系统上，用户尝试登录失败相关的日志；
		lastb命令查看
		
	/var/log/wtmp：当前系统上，用户正常 登录的日志；
		last命令查看
			-n #：仅查看最近#条记录；
			
		lastlog命令：用于查看每个用户最近一次登录时间；
	
	/var/log/dmesg：系统引导过程中的日志信息；
		文本文件查看工具；
		也可以使用dmesg命令查看；
		
	rsyslog记录日志于mysql：
	(1) 安装rsyslog连接至mysql的模块；
	# yum install rsyslog-mysql 
	(2) 在mysql中创建专用于rsyslog-mysql的用户账号；
	mysql> GRANT ALL ON Syslog.* TO 'USERNAME'@'HOST' IDENTIFIED BY 'PASSWORD'
	(3) 生成所需的数据库和表；
	# mysql  -uUSERNAME -hHOST -p < /usr/share/doc/rsyslog-7.4.7/mysql-createDB.sql
	(4) 配置rsyslog使用ommysql模块
	### MODULES ###
	$ModLoad  ommysql 
	
	(5) 配置RULES，将某facility的日志发往mysql
	### RULES ###
	facility.priority 	:ommysql:DBHOST,DB,DBUSERNAME,DBUSERPASS
	
	(6) 重启rsyslog进程
	
	(7) 提供Web GUI工具loganalyzer
	(a) 配置amp组合；
	(b) 安装loganalyzer
		# tar xf loganalyzer-3.6.6.tar.gz
		# cd  loganalyzer-3.6.6
		# cp -r loganalyzer-3.6.6/src  /var/www/html/loganalyzer
		# cp contrib/*.sh  /var/www/html/loganalyzer
		# cd  /var/www/html/loganalyzer
		# chmod +x *.sh
		# ./configure.sh
		# ./secure.sh
		# chmod 666 config.php
		
	通过浏览器访问：
		http://HOST/loganalyzer/install.php 
		
			MySQL Native, Syslog Fiel
			
			Table type要选择“Monitorware”
		
	





		

























