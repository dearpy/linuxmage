zabbix：

	监控系统：硬件、软件、业务指标
		sensor：数据指标
			采样：				
			存储：
				数据：历史数据、趋势数据
			报警：
				脚本：
				媒介：
			展示：Visual

			
		监控数据采集通道
			SNMP：Simple Network Management Protocol
			ssh/telnet
			IPMI Intelligent Platform Management Interface
			agent:
				master/agent
			
		NMS:
			网络监控系统		
			
			开监控工具：
				cacti, nagios, zabbix, ganglia
					cacti， nagios
					zabbix
					ganglia
				
			数据：
				历史数据：NVPS
				趋势数据
					
				存储系统：
					关系型数据库：MySQL
					rrd：roundrobin database
					NoSQL：redis/mongodb
						时间序列存储
						
		SNMP协议：
			nms/agent
				nms: cli/gui
				agent: service
				
			v1
			v2c: community，public
			v3
			
			MIB, OID, ...
			
	ZABBIX：
		LTS：2.2， 3.0
			long time support
			
		特性：
			数据采样：
				snmp, ssh/telnet, agent, ipmi, jmx(java Management eXtensions)
				自定义采样机制：UserPrameter
			告警
				升级：
					script
					notification
			数据存储:
				数据存储：mysql/pgsql
			展示：
				实时绘图：graph, screen, slide show, map
			
			支持模板：
			网络自动发现：
			分布式监控：
				Server <--> Proxy <--> agent/ssh/ipmi 
			API
			
		zabbix程序的组件：
			zabbix_server：服务端守护进程；
			zabbix_agentd：agent守护进程；
			zabbix_proxy：代理服务器，可选组件；
			zabbix_get：命令行工具，手动测试向agent发起数据采集请求；
			zabbix_sender：命令行工具，运行于agent端，手动向server端发送数据；
			zabbix_java_gateway: java网关；
			zabbix_database：MySQL或PostgreSQL；
			zabbix_web：Web GUI
		
自整理：
zabbix 安装 配置好 epel base zabbix 源
服务器： zabbix-server-mysql zabbix-get  zabbix-agent（自监控使用） （zabbix-server旧版）mariadb（可独立） 	（zabbix-web zabbix-web-mysql httpd php ）可独立
agent：	zabbix-agent zabbix-sender 

注意：script 警告 添加media types中添加script User中添加Media时 tpye 为 script的名字 类型scripts action中 还是要选择sendmessage 并通过脚本实现发送内容
		 脚本要有执行权限 
有问题 多看日志

		zabbix逻辑组件：
			主机组
			主机 
			监控项(item)
				key：实现获取监控的目标上的数据的命令或脚本的名称；
			应用(application)：同一类监控项的集合；
			触发器(trigger)：表达式；PROBLEM， OK；
			事件(event)：
			动作(action)：由条件(condition)和操作(operation)组件；
			媒介(media)：发送通知的通道；
			通知(notification)：
			远程命令(remote command)：
			报警升级()：
			模板(template)：快速定义被监控主机的各监控项的预设项目集合；
			图形(graph)：用于展示历史数据或趋势数据的图像；
			屏幕(screen)：由多个graph组成；
			
		zabbix安装：
			设置ZBX DB：
				mysql> CREATE DATABASE zabbix CHARSET 'utf8';
				mysql> GRANT ALL ON zabbix.* TO ...;
				
			安装服务端：
				~]# yum install zabbix-server-mysql-3.0.2-1.el7.x86_64.rpm zabbix-get-3.0.2-1.el7.x86_64.rpm
				
				注意：CentOS 7.0和7.1需要升级trousers程序包；
				
			安装web GUI：
				~]# yum install httpd php php-mysql php-mbstring php-gd php-bcmath php-ldap php-xml
				~]# yum install zabbix-web-3.0.2-1.el7.noarch.rpm zabbix-web-mysql-3.0.2-1.el7.noarch.rpm 
				
			安装agent端：
				 ~]# yum install zabbix-agent-3.0.2-1.el7.x86_64.rpm zabbix-sender-3.0.2-1.el7.x86_64.rpm
				 
		服务端数据库初始化：
			2.x：三个sql脚本；
			3.x：一个sql脚本；
				~]# gzip  -d  create.sql.gz 
				~]# mysql -uzbxuser -h127.0.0.1 -p zabbix < create.sql 
				
		zabbix server配置启动：
			配置文件：/etc/zabbix/zabbix_server.conf
			配置段：
				~]# grep "^#####" zabbix_server.conf
				############ GENERAL PARAMETERS #################
				############ ADVANCED PARAMETERS ################
				####### LOADABLE MODULES #######
				####### TLS-RELATED PARAMETERS #######	
				
			通用参数：
				ListenPort=10051
				SourceIP=
				LogType=file
				LogFile=/var/log/zabbix/zabbix_server.log
				LogFileSize=0
				DebugLevel=3
				
				DBHost=localhost
				DBName=zabbix
				DBUser=zabbix
				DBPassword=
				DBSocket=/tmp/mysql.sock
				DBPort=3306
				
		配置zabbix-web:
			配置php的时区设定：
				(1) /etc/php.ini 
				(2) /etc/httpd/conf.d/zabbix.conf 
					php_value date.timezone 
			
			访问URL
				http://HOST/zabbix 
					
			安装生成的配置文件：/etc/zabbix/web/zabbix.conf.php
			
			登录：
				admin/zabbix 
				
			Monitoring
			Inventory
			Reports
			Configuration
			Administration
			
回顾：
	监控系统：
		采样、存储、展示、报警
		
		采样：ipmi, agent, ssh/telnet，snmp, jmx, ...
		存储：历史数据、趋势数据
			rrd、sql、nosql 
		展示：Web GUI、app 
		报警：发送通知 
			媒介
			
	zabbix：
		程序组成部分：
			zabbix-server-mysql
			zabbix-get 
			
			zabbix-web 
			zabbix-web-mysql 
			
			zabbix-agent 
			zabbix-sender
			
			zabbix-proxy
			
		被监控主机的可用接口：
			agent：
			IPMI：
			snmp:
			jmx:
			
ZABBIX(2)

	agent端的配置：
		~]# yum  install zabbix-agent-3.0.2-1.el7.x86_64.rpm  zabbix-sender-3.0.2-1.el7.x86_64.rpm
		
		Unit file： zabbix-agent.service 
		配置文件：/etc/zabbix/zabbix_agentd.conf 
			############ GENERAL PARAMETERS #################
			##### Passive checks related
				被动监控相关配置
			##### Active checks related
				主动监控相关配置，agent端主动向server周期性发送数据；
			############ ADVANCED PARAMETERS #################
			####### USER-DEFINED MONITORED PARAMETERS #######
				用户自定义参数
			####### LOADABLE MODULES #######
			####### TLS-RELATED PARAMETERS #######			
		
		##### Passive checks related
			Server=IP1, IP2, ...
			ListenPort=10050
			ListenIP=0.0.0.0
			StartAgents=3
			
		##### Active checks related
			ServerActive=IP1[:port], IP2[:port], ...
			Hostname=Unique_HOSTNAME
				必须与服务器配置的监控主机的主机名称保持一致；
			
		启动服务：
			systemctl start zabbix-agent.service 
			
	监控配置：
		术语：host groups --> host --> application --> item --> trigger --> action (conditions, operations)
			    graph: 
				simple: 每个item定义完成后自动生成 
				customed：用于将多个item的数据整合于一个图形中展示
				
		主机：
		
		
		item: key+parameter 
			key: 
				zabbix内建：
					type: 
						agent 
						agent(active)
						snmp v1
						...
				用户自定义(UserParameter)
				
			采集到的数据的类型：
				数值：
					整数
					浮点数 
				字符串：
					字符串
					文本
			字段意思：items
Name：监控项的名字
Type：监控agent使用的类型 一旦类型确定，能使用的key也就确定了
host interface：被监控主机的IP地址+端口号
type of information：
data type D 十进制 
history 历史数据存储时间
trend 趋势数据存储时间	
update interval 每隔多长时间同步一次数据

测试：zabbix_get -s 172.16.174.177 -k "net.if.in[eno16777736,packets]"
			存储的值：
				As is：不对数据做任何处理
				Delta：（simple change)，本次采样减去前一次采样的值的结果
				Delta：（speed per second)，本次采样减去前一次采样的值，再除以经过的时长；
			
		trigger：
			逻辑表达式，阈值；通常用于定义数据的不合理区间；
				OK：正常 状态 --> 较老的zabbix版本，其为FALSE；
				PROBLEM：非正常 状态 --> 较老的zabbix版本，其为TRUE；
				
				OK --> PROBLEM 
				Recovery：PROBLEM --> OK 
				
			触发器存在可调用的函数：
				nodata()
				last()
				
			Severity
				Not classified
				Information
				Warning
				Average
				High
				Disaster
				
			触发器表达式：
				{hostname:key[paramters].function(arguments) 
					>, <, =, #（not equal）...
					+, -, *, /
					&, |
				
				{n1.magedu.com:net.if.in[eno16777736,packets].last(#1)}>15
				
			trigger间存在依赖关系：
			
		Media：媒介
			告警信息的传递通道；
			类型：
				Email：邮件
				Script：自定义脚本
				SMS：短信
				Jabber：
				Ez Texting：
				
			接收信息的目标为zabbix用户：
				需要用户上定义对应各种媒介通道的接收方式；
				
		Action：
			conditions：	
				多个条件之间存在逻辑关系；
			operations：
				条件满足时触发的操作；
					
				send message：
					(1) Media type：传递信息的通道；
						(a) Email
						(b) Script：报警脚本；
							脚本放置路径：zabbix_server.conf配置文件中AlertScriptsPath参数定义的路径下；
								/usr/lib/zabbix/alertscripts/
							zabbix服务器在调用脚本时，会向其传递三个参数：
								$1：经由此信道接收信息的目标；
								$2：subject
								$3：body
					(2) 信息接收人：
						(a) User Groups
						(b) Users
					
				remote command
					功能：
						在agent所在的主机上运行用户指定的命令或脚本；例如：
							重启服务；
							通过IPMI重启服务器；
							任何用户自定义脚本中定义的操作； 
							
					可执行的命令类型：
						IPMI	Intelligent Platform Management Interface
						ssh 
						telnet 
						Custom Script
						Global Script
						
					前提：
						在agent需要完成的配置：
							(1) zabbix用户拥有所需要的管理权限；
								编辑/etc/sudoers文件，注释如下行；
								Defaults requiretty
								Defaults zabbix !requiretty (此用户不需要tty)
								添加如下行：
								zabbix  ALL=(ALL)  NOPASSWD: ALL
								
							(2) agent进程要允许执行远程命令； 
								编辑/etc/zabbix/zabbix_agentd.conf，设置如下配置：
								EnableRemoteCommands=1
								
								重启服务生效；
								
		总结：
			host groups --> host --> application --> item (key) --> trigger --> action
				
				(1) media type
				(2) user group/user
				
			action operations: 可定义为升级方式；
				send message 
				remote command
				
回顾：

	zabbix：
		host group --> host
			host：
				agent
				IPMI
				SNMP
				JMX 
		
		item：监控项
			key + parameters
			
		trigger：表达式 
			TRUE：PROBLEM
			FALSE：OK
			
			(1) 依赖关系；
			(2) 一个item之上可以存在多个trigger；
			
			结果：trigger events 
		
		action：conditions + operations
			operations：
				send message 
					email、scripts 
						$1: contact, {ALERT.SENDTO}
						$2: subject, {ALERT.SUBJECT}
						$3: message body, {ALERT.MESSAGE}
				remote command
				
ZABBIX(03)

	展示接口：
		graph: simple, custom
		screen：把多个graph整合于同一屏幕进行展示；
		slide show：把多个screen以slide show的方式进行展示
		
	模板：
		主机配置模板：用于链接至目标主机实现快速监控管理；
			link, unlink, unlink and clear 
			
		模板可继承；
		
	宏：macro，预设的文本替换模式；
		级别：
			全局：Administration --> General --> Macros 
			模板：编辑模板 --> Macros
			主机：编辑主机 --> Macros 
			
		类型：
			内建：{MACRO_NAME}
			自定义：{$MACRO_NAME}
				命名方式：大写字母、数字和下划线；
				
	网络发现：
		zabbix server扫描指定网络范围内的主机；
		
			发现方式：
				ip地址范围；
					可用服务（ftp, ssh, http, ...）
					zabbix_agent的响应；
					snmp_agent的响应； 
					
			分两个阶段：
				discovery 
				actions 
				
			发现：--> discovery events 
				Service, Host 
				
				UP/DOWN, DICOVERED/LOST 
				
			可采取的动作：
				send message, remote command
				add/remove host 
				enable/disable host 
				add host to group
				link  template to host
				...
				
	自定义key：
		
		item type： 不同的类型适用的接口有可能不同；有些key仅能用在指定的接口之上；
			agent
			agent(active)
			simple
			snmpv1
			snmpv2
			snmpv3
			ssh 
			...
			
		接口类型：agent, ipmi, snmp, jmx
			
		自定义key：在zabbix agent端的配置文件上由用户通过UserParameter指令定义的key；
			zabbix_agentd.conf文件中
				UserParameter=<key>,<command>
				
			UserParameter=nginx.active,curl -s http://localhost/status | awk '/^Active/{print $3}'
			UserParameter=nginx.accepts,curl -s http://localhost/status | awk '/^[[:space:]]*[0-9]/{print $1}'
			UserParameter=nginx.handled,curl -s http://localhost/status | awk '/^[[:space:]]*[0-9]/{print $2}'
			UserParameter=nginx.requests,curl -s http://localhost/status | awk '/^[[:space:]]*[0-9]/{print $3}'			
					
		实践：
			监控lvs director上web服务的相关统计数据及速率数据；
			
			制定出lvs director的监控模板：
				items, trigger, graph 
				
		
回顾：
	zabbix的展示：
		graph, screen, slide show; 
		map; 
		
	template：主机级别的配置的集合；
		items, triggers, graphs, ...
		link/unlink/unlink and clear
		
	macro：
		全局、模板、主机；
		文本替换模式；
	
	UserParameter：
		agent interface 
		UserParameter=<key>[*],<command>
		
	network discovery：
		Service, Host 
		UP/DOWN；DISCOVERED/LOST 
		
		发现 --> actions 
			add/remove hosts
			enable/disable hosts
			link/unlink templates
			...
			
ZABBIX(4)

	Web监控：
		监控指定的站点的资源下载速度，及页面响应时间；
		
	主动/被动 检测：
		被动检测：相对于agent而言；agent,  server向agent请求获取配置的各监控项相关的数据，agent接收请求、获取数据并响应给server；
		主动检测：相对于agent而言；agent(active),agent向server请求与自己相关监控项配置，主动地将server配置的监控项相关的数据发送给server；
			agent端所需要基本配置：
				ServerActive=
				Hostname=
				
	基于SNMP监控：
	
		SNMP：简单网络管理协议；
			agent/nms
			
			读（get, getnext）、写（set）、trap（陷阱）；
			
			161/udp
			162/udp
			
		SNMP:
			v1: 1989
			v2: 1993
			v3: 1998 
			
		MIB：Management Information Base
		OID：Object ID
		
		Linux启用snmp的方法：
			# yum install net-snmp net-snmp-utils 
			配置文件：
				/etc/snmp/snmpd.conf
				定义ACL 
				
			启动服务：
				systemctl  start  snmpd.service 
				
		测试工具：
			# snmpget -v 2c  -c  public  HOST  OID
			# snmpwalk  -v 2c -c public  HOST  OID 
			-v 版本snmp -c community   HOST 172.16.174.201 OID :system.sysDescr.0 sysName.0
				更改 /etc/snmp/snmpd.conf 授权 
				snmpwalk -v 1 localhost -c public .1.3.6.1.4.1.2021.10
		Key	<Unique string to be used as reference to triggers> For example, “my_param”.
		
	Zabbix Proxy的配置：
		server-node-agent 
		server-proxy-agent
注意：更改代理后 应该将server 指向代理服务器
		1、配置proxy主机：
			(1) 安装程序包 
				zabbix-proxy-mysql zabbix-get 
				zabbix-agent zabbix-sender 
				
			(2) 准备数据库
				创建、授权用户、导入schema.sql；
				
			(3) 修改配置文件
				Server=
					zabbix server主机地址；
				Hostname=
					当前代理服务器的名称；在server添加proxy时，必须使用此处指定的名称；
					需要事先确保server能解析此名称；
				DBHost=
				DBName=
				DBUser=
				DBPassword=
				
				ConfigFrequency=10
				DataSenderFrequency=1
				
		2、在server端添加此Porxy
			Administration --> Proxies 
			
		3、在Server端配置通过此Proxy监控的主机；
		
	zabbix performace tuning：
		nvps：new values per second 
			100w/m, 15000/s
			
		调优：
			Database：
				历史数据不要保存太长时长；
				尽量让数据缓存在数据库服务器的内存中；
			触发器表达式：减少使用min(), max(), avg()；尽量使用last()，nodata()；
			数据收集：polling较慢(减少使用SNMP/agentless/agent）；尽量使用trapping（agent(active））；
			数据类型：文本型数据处理速度较慢；尽量少收集类型为text或string类型的数据；多使用类型为numeric的；
			
		zabbix服务器的器：
			(1) 服务器组件的数量；
				alerter, discoverer, escalator, http poller, hourekeeper, icmp pinger, ipmi polller, poller, trapper, configration syncer, ...
				
				StartPollers=60
				StartPingers=10
				...
				StartDBSyncer=5
				...
				
			(2) 数据库优化
				分表：
					history_*
					trends*
					events*
					
实例：		
		nginx status 开启方法：
		server {
			...
			location /status {
				stub_status on;
				access_log off;
				allow 123.123.123.123; # 允许访问的 IP
				allow 127.0.0.1;
				deny all;
			}
		}
		
		状态页面各项数据的意义：
		active connections – 当前 Nginx 正处理的活动连接数。
		serveraccepts handled requests — 总共处理了 233851 个连接 , 成功创建 233851 次握手 (证明中间没有失败的 ), 总共处理了 687942 个请求 ( 平均每次握手处理了 2.94 个数据请求 )。
		reading — nginx 读取到客户端的 Header 信息数。
		writing — nginx 返回给客户端的 Header 信息数。
		waiting — 开启 keep-alive 的情况下，这个值等于 active – (reading + writing)， 意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接。
				
				
					
之前笔记


https://www.zabbix.com/documentation/2.4/manual/config/items/itemtypes/zabbix_agent

各key的获取位置






源码安装：

下载地址：http://www.zabbix.com/download.php

# tar -zxvf zabbix-2.0.0.tar.gz

创建用户：
# groupadd zabbix
# useradd -g zabbix zabbix

注意：同时安装了server和agent的节点上，建议其运行用户不要相同。



创建数据库：

server和proxy的运行都依赖于数据库，agent则不需要。

以MySQL数据库为例：
mysql> CREATE DATABASE zabbix CHARACTER SET utf8 COLLATE utf8_bin;
mysql> GRANT ALL ON zabbix.* TO zbuser@'%' IDENTIFIED BY 'zbpass';
# 请按需要修改用户名和密码；
shell> mysql -u<username> -p<password> zabbix < database/mysql/schema.sql
# 如果仅为proxy创建数据库，只导入schema.sql即可；否则，请继续下面的步骤；

shell> mysql -u<username> -p<password> zabbix < database/mysql/images.sql
shell> mysql -u<username> -p<password> zabbix < database/mysql/data.sql



编译安装zabbix:

同时安装server和agent，并支持将数据放入mysql数据中，可使用类似如下配置命令：
./configure --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-ssh2

如果仅安装server，并支持将数据放入mysql数据中，可使用类似如下配置命令：
./configure --enable-server --with-mysql --with-net-snmp --with-libcurl

如果仅安装proxy，并支持将数据放入mysql数据中，可使用类似如下配置命令：
./configure --prefix=/usr --enable-proxy --with-net-snmp --with-mysql --with-ssh2

如果仅安装agent，可使用类似如下配置命令：
./configure --enable-agent


而后编译安装zabbix即可：
# make
# make install




配置zabbix：

server的配置文件为zabbix_server.conf，至少应该为其配置数据库等相关的信息；

agent的配置文件为zaabix_agentd.conf，至少应该为其指定server的IP地址；

proxy的配置文件为zabbix_proxy.conf，至少应该为其指定proxy的主机名和server的IP，以及数据库等相关的配置信息；




启动zabbix:

server:  zabbix_server

agent: zabbix_agentd

proxy: zabbix_proxy






安装frontend: 

# cp -a  frontend/php/  /var/www/html/zabbix

启动lamp或lnmp后，通过浏览器访问http://<server_ip_or_name>/zabbix即可进行安装。






如果使用rpm安装，zabbix关于mysql的数据库脚本文件路径为：
# cd /usr/share/doc/zabbix-server-mysql-2.0.6/create/


zabbix: 
	数据采集-->数据存储-->数据展示和分析-->报警

	数据采集：
		SNMP
		agent
		ICMP/SSH/IPMI

	数据存储：
		cacti: rrd
		nagios: , mysql
		zabbix: mysql/pgsql/oracle

	数据展示（Web）：
		java
		php
		移动app

	报警：
		mail(smtp)
		Chat Message
		SMS


zabbix:
	zabbix agent
	agent(active)
	SNMP
	SSH

zabbix: 
	用RDBMS保存；

数据展示：
	php, web gui

报警：
	报警升级

如何确定zabbix的监控对象：
	手动添加
	自动发现

	hosts, host group
	item, application
		item: key
	graph, screen
	trigger, event (discovery)
	action (notification, operation, condition)



zabbix仅运行在触发器上定义依赖关系；

























模板是一系列配置的集合，它可以方便地快速部署在某监控对象上，并支持重复应用。模板可包含多种类型的条目：
	items
	triggers
	graphs
	applications
	screens (since Zabbix 2.0)
	low-level discovery rules (since Zabbix 2.0)


将模板应用至某主机上时，其定义的所有条目都会自动添加。因此，模板通常用于为某监控的服务或应用程序整合一组条目并将之分别应用于对应的运行了相应服务的主机上。此外，模板的另一个好处在于，必要时，修改了模板，被应用的主机都会相应的作出修改。






UserParameter：

UserParameter=<key>,<command>



剩余的内容：
	discovery
	web monitoring
	jmx
	分布式监控








Delta (speed per second)：保存为(value-prev_value)/(time-prev_time的计算结果，即当前值减去前一次获取的数据值，除以当前时间戳减去前一次值获取时的时间戳得到的结果；如果当前值小于前一次的值，其将会被丢弃；
Delta (simple change)：保存为 (value-prev_value)的计算结果；



用户自定义参数：


/etc/zabbix
	zabbix_agentd.conf, zabbix_agentd.d/*.conf











nginx status 开启方法：
server {
	...
	location /status {
		stub_status on;
		access_log off;
		allow 123.123.123.123; # 允许访问的 IP
		allow 127.0.0.1;
		deny all;
	}
}

状态页面各项数据的意义：
active connections – 当前 Nginx 正处理的活动连接数。
serveraccepts handled requests — 总共处理了 233851 个连接 , 成功创建 233851 次握手 (证明中间没有失败的 ), 总共处理了 687942 个请求 ( 平均每次握手处理了 2.94 个数据请求 )。
reading — nginx 读取到客户端的 Header 信息数。
writing — nginx 返回给客户端的 Header 信息数。
waiting — 开启 keep-alive 的情况下，这个值等于 active – (reading + writing)， 意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接。





/usr/lib/zabbix/externalscripts
/etc/zabbix/externalscripts



自动发现
web监控
分布式环境中使用zabbix






UserParameter=Nginx.active[*], /usr/bin/curl -s "http://$1:$2/status" | awk '/^Active/ {print $NF}'
UserParameter=Nginx.reading[*], /usr/bin/curl -s "http://$1:$2/status" | grep 'Reading' | cut -d" " -f2
UserParameter=Nginx.writing[*], /usr/bin/curl -s "http://$1:$2/status" | grep 'Writing' | cut -d" " -f4
UserParameter=Nginx.waiting[*], /usr/bin/curl -s "http://$1:$2/status" | grep 'Waiting' | cut -d" " -f6
UserParameter=Nginx.accepted[*], /usr/bin/curl -s "http://$1:$2/status" | awk '/^[ \t]+[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/ {print $$1}'
UserParameter=Nginx.handled[*], /usr/bin/curl -s "http://$1:$2/status" | awk '/^[ \t]+[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/ {print $$2}'
UserParameter=Nginx.requests[*], /usr/bin/curl -s "http://$1:$2/status" | awk '/^[ \t]+[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/ {print $$3}'








UserParameter=nginx.access_countaccess, /usr/lib/zabbix/externalscripts/logcheck_nginx.accesslog totalaccess
UserParameter=nginx.access_count200, /usr/lib/zabbix/externalscripts/logcheck_nginx.accesslog 200access
UserParameter=nginx.access_count202, /usr/lib/zabbix/externalscripts/logcheck_nginx.accesslog 202access
UserParameter=nginx.access_count4xx, /usr/lib/zabbix/externalscripts/logcheck_nginx.accesslog 4xxaccess
UserParameter=nginx.access_count3xx, /usr/lib/zabbix/externalscripts/logcheck_nginx.accesslog 3xxaccess
UserParameter=nginx.access_count5xx, /usr/lib/zabbix/externalscripts/logcheck_nginx.accesslog 5xxaccess











UserParameter=varnish.stat[*], /usr/lib/zabbix/externalscripts/varnishstatus varnish_stat $1
UserParameter=varnish.count[*], /usr/lib/zabbix/externalscripts/varnishstatus varnish_count $1
UserParameter=varnish.hitrate, /usr/lib/zabbix/externalscripts/varnishstatus varnish_hitrate



































# This is a config file for Zabbix Agent (Unix)
# To get more information about Zabbix, visit http://www.zabbix.com

############ GENERAL PARAMETERS #################

### Option: PidFile
# Name of PID file.
#
# Mandatory: no
# Default:
PidFile=/var/log/zabbix/zabbix_agentd.pid

### Option: LogFile
# Name of log file.
# If not set, syslog is used.
#
# Mandatory: no
# Default:
# LogFile=

LogFile=/var/log/zabbix/zabbix_agentd.log

### Option: LogFileSize
# Maximum size of log file in MB.
# 0 - disable automatic log rotation.
#
# Mandatory: no
# Range: 0-1024
# Default:
# LogFileSize=1
LogFileSize=20

### Option: DebugLevel
# Specifies debug level
# 0 - no debug
# 1 - critical information
# 2 - error information
# 3 - warnings
# 4 - for debugging (produces lots of information)
#
# Mandatory: no
# Range: 0-4
# Default:
# DebugLevel=3


### Option: SourceIP
# Source IP address for outgoing connections.
#
# Mandatory: no
# Default:
# SourceIP=

### Option: EnableRemoteCommands
# Whether remote commands from Zabbix server are allowed.
# 0 - not allowed
# 1 - allowed
#
# Mandatory: no
# Default:
# EnableRemoteCommands=0

### Option: LogRemoteCommands
# Enable logging of executed shell commands as warnings.
# 0 - disabled
# 1 - enabled
#
# Mandatory: no
# Default:
# LogRemoteCommands=0

##### Passive checks related

### Option: Server
# List of comma delimited IP addresses (or hostnames) of Zabbix servers.
# Incoming connections will be accepted only from the hosts listed here.
# No spaces allowed.
# If IPv6 support is enabled then '127.0.0.1', '::127.0.0.1', '::ffff:127.0.0.1' are treated equally.
#
# Mandatory: no
# Default:
# Server=

Server=172.16.10.15,127.0.0.1

### Option: ListenPort
# Agent will listen on this port for connections from the server.
#
# Mandatory: no
# Range: 1024-32767
# Default:
# ListenPort=10050

### Option: ListenIP
# List of comma delimited IP addresses that the agent should listen on.
# First IP address is sent to Zabbix server if connecting to it to retrieve list of active checks.
#
# Mandatory: no
# Default:
# ListenIP=0.0.0.0

### Option: StartAgents
# Number of pre-forked instances of zabbix_agentd that process passive checks.
# If set to 0, disables passive checks and the agent will not listen on any TCP port.
#
# Mandatory: no
# Range: 0-100
# Default:
# StartAgents=3
StartAgents=2

##### Active checks related

### Option: ServerActive
# List of comma delimited IP:port (or hostname:port) pairs of Zabbix servers for active checks.
# If port is not specified, default port is used.
# IPv6 addresses must be enclosed in square brackets if port for that host is specified.
# If port is not specified, square brackets for IPv6 addresses are optional.
# If this parameter is not specified, active checks are disabled.
# Example: ServerActive=127.0.0.1:20051,zabbix.domain,[::1]:30051,::1,[12fc::1]
#
# Mandatory: no
# Default:
# ServerActive=

ServerActive=172.16.10.15

### Option: Hostname
# Unique, case sensitive hostname.
# Required for active checks and must match hostname as configured on the server.
# Value is acquired from HostnameItem if undefined.
#
# Mandatory: no
# Default:
# Hostname=

# Hostname=Zabbix server

### Option: HostnameItem
# Item used for generating Hostname if it is undefined.
# Ignored if Hostname is defined.
#
# Mandatory: no
# Default:
HostnameItem=system.hostname

### Option: RefreshActiveChecks
# How often list of active checks is refreshed, in seconds.
#
# Mandatory: no
# Range: 60-3600
# Default:
# RefreshActiveChecks=120
RefreshActiveChecks=60

### Option: BufferSend
# Do not keep data longer than N seconds in buffer.
#
# Mandatory: no
# Range: 1-3600
# Default:
# BufferSend=5
BufferSend=10

### Option: BufferSize
# Maximum number of values in a memory buffer. The agent will send
# all collected data to Zabbix Server or Proxy if the buffer is full.
#
# Mandatory: no
# Range: 2-65535
# Default:
# BufferSize=100
BufferSize=1000

### Option: MaxLinesPerSecond
# Maximum number of new lines the agent will send per second to Zabbix Server
# or Proxy processing 'log' and 'logrt' active checks.
# The provided value will be overridden by the parameter 'maxlines',
# provided in 'log' or 'logrt' item keys.
#
# Mandatory: no
# Range: 1-1000
# Default:
# MaxLinesPerSecond=100
MaxLinesPerSecond=200

### Option: AllowRoot
# Allow the agent to run as 'root'. If disabled and the agent is started by 'root', the agent
#       will try to switch to user 'zabbix' instead. Has no effect if started under a regular user.
# 0 - do not allow
# 1 - allow
#
# Mandatory: no
# Default:
# AllowRoot=0

############ ADVANCED PARAMETERS #################

### Option: Alias
# Sets an alias for parameter. It can be useful to substitute long and complex parameter name with a smaller and simpler one.
#
# Mandatory: no
# Range:
# Default:

### Option: Timeout
# Spend no more than Timeout seconds on processing
#
# Mandatory: no
# Range: 1-30
# Default:
Timeout=20

### Option: Include
# You may include individual files or all files in a directory in the configuration file.
# Installing Zabbix will create include directory in /usr/local/etc, unless modified during the compile time.
#
# Mandatory: no
# Default:
# Include=

# Include=/usr/local/etc/zabbix_agentd.userparams.conf
Include=/usr/local/zabbix/etc/zabbix_agentd.conf.d/

####### USER-DEFINED MONITORED PARAMETERS #######

### Option: UnsafeUserParameters
# Allow all characters to be passed in arguments to user-defined parameters.
# 0 - do not allow
# 1 - allow
#
# Mandatory: no
# Range: 0-1
# Default:
# UnsafeUserParameters=0

### Option: UserParameter
# User-defined parameter to monitor. There can be several user-defined parameters.
# Format: UserParameter=<key>,<shell command>
# See 'zabbix_agentd' directory for examples.
#
# Mandatory: no
# Default:
# UserParameter=










			
			
			
			
		
				
			
			
			
				
				
				
				
				
				
			
			
			
			
		
			
			
				
		
		
	
		
			
		
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			  

			  
			  
			  
			  
			  
			  
			  
			  
			  
			  

			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  






		  
			  
			  
			  
			  