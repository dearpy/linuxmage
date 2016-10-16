#IPTABLES-小结
	
##相关概念
	带状态检测的包过滤型防火墙；连接追踪（connection tracking）；
	IPTABLES实现防火墙原理：通过对表中响应的链添加规则已达到对经由的包进行过滤控制修改的一系列操作；
		表tables：filter，nat，mangle，raw
		链chain：PREROUTING,INPUT,FORWARD,OUTPUT,POSTROUTING
	添加规则：就是在表（t）添加在链上（INPUT）的一系列规则	
		iptables -t filter -A INPUT -d 172.16.174.100 -p tcp --dport 80 -j ACCEPT	
		Packets Filter Firewall 包过滤型防火墙:firewall 为iptables的前身；	
	Firewall：隔离工具；工作于主机或网络的边缘，对经由的报文根据预先定义的规则（识别条件）进行检测，对于能够被规则匹配到的报文实行某预定义的处理机制的一套组件；
			硬件防火墙：在硬件级别能部分防火墙，另一部分功能基于软件实现； 
			软件防火墙：应用软件处理逻辑运行通用硬件实现的防火墙；
			
			主机防火墙：服务范围为当前主机；
			网络防火墙：服务范围为局域网；
					
	iptables/netfilter：软件实现的主机或网络防火墙； 
			iptables：规则编写工具；
			netfilter：防火墙框架；framework；
		netfilter 组件也称为内核空间（kernelspace），是内核的一部分，由一些信息包过滤表组成，这些表包含内核用来控制信息包过滤处理的规则集。
		iptables 组件是一种工具，也称为用户空间（userspace），它使插入、修改和除去信息包过滤表中的规则变得容易。除非您正在使用 Red Hat Linux 7.1 或更高版本，否则需要下载该工具并安装使用它。
##链chain 
	系统中五个钩子函数定义了5个链chain：PREROUTING,INPUT,FORWARD,OUTPUT,POSTROUTING
##表tables
	filter：过滤，防火墙；
	nat：network address translation 用于修改报文的原地址或目标地址，甚至端口号
	mangle；拆解报文，做出修改，并重新封装起来
	raw：关闭nat上启用的连接追踪机制
	
	优先级次序（由高到低）：
		raw->mangle->nat->filter
	功能<->钩子
		raw：PREROUTING,OUTPUT
		mangle:PREOUTING,INTPUT,FORWARD,OUTPUT,POSTROUTING
		nat:PREROUTING,INPUT,OUTPUT,POSTROUTING
		filter:INPUT,FORWARD,OUTPUT
		报文流向：
			由本级到某个进程：PREROUTING->INPUT
			由本机转发的报文：PREROUTING->FORWARD->POSTROUTING
			由本级某个进程转发的：OUTPUTING->POSTROUTING
	规则的组成部分：
		匹配条件：-m match
			网络层首部属性值：ip
			传输层首部属性值：port
			附加的条件
		
			基本匹配条件：简单检查IP、TCP、UDP等报文的某属性进行匹配的机制；					
			扩展匹配条件：需要借助于扩展模块进行的匹配条件指定即为扩展匹配；
		icmp: internet control messaging protocol
		处理动作：-j target
			基本动作：ACCEPT，DROP, ...
			扩展动作：需要借助扩展模块进行的动作；
		注：systemctl disable firewalld.service
		配置文件：/etc/sysconfig/iptables-config
	添加规则之时需要考量的问题：
			(1) 报文的流经路径，判断添加规则至哪个链上；
			(2) 确定要实现的功能，判断添加规则至哪个表上；
			(3) 要指定的匹配条件，以用于匹配目标报文；
##iptables 
	命令的格式：
	格式：iptables [-t table] COMMAND chain CRETIRIA -j TARGET
		table（表）：默认filter。nat，mangle，raw ...
		COMMAND：
		chain（链）：系统自己带的链：PREROUTING,INPUT,FORWARD,OUTPUT,POSTROUTING 也可以通过-N 新建自定义链
		CRETIRIA 匹配条件：match
		-j 处理动作
	COMMAND:
		链：
			-P：policy，策略，定义默认策略； 一般有两种选择，ACCEPT和DROP；
			-N:new，新建一条自定义的规则链；被内建链上的规则调用才能生效；[-j  chain_name]；
			-X:drop删除自定义的引用计数为0的空链
			-F:flush：flush，清空指定的链
			-E:重命名自定义的引用计数和为0的链；
		规则：
			-A:append,追加，在指定链的尾部追加一条规则；
			-I:insert,插入，在指定的位置（省略位置时表示链首）插入一条规则；
			-D:delete,删除，删除指定的规则；
			-R:replace,替换，将指定的规则替换为新规则；不能仅修改规则中的部分，而是整条规则完全替换；
		查看：
			-L：list，列出表中的链上的规则；
				-n：numeric，以数值格式显示；
				-v：verbose，显示详细格式信息； 
					-vv, -vvv
				-x：exactly，计数器的精确结果；
				--line-numbers：显示链中的规则编号；
		计数器：
					规则，以及默认策略有专用的计数器；
					记录被当前规则所匹配到的：
						(1) 报文个数；
						(2) 字节总数；
					
		重置规则计数器：
			-Z：zero，置0；
	匹配条件：
		多重条件：逻辑关系为“与”；
		基本匹配条件：
			[!] -s, --source address[/mask][,...]：检查报文中的源IP地址是否符合此处指定的地址或范围；
			[!] -d, --destination address[/mask][,...]：检查报文中的目标IP地址是否符合此处指定的地址或范围；
			[!] -p, --protocol protocol：
				protocol：{tcp|udp|icmp}
			[!] -i, --in-interface name：数据报文的流入接口；INPUT, FORWARD  and  PREROUTING 
			[!] -o, --out-interface name：数据报文的流出接口； FORWARD, OUTPUT and POSTROUTING
		扩展匹配条件
			隐式扩展：不用-m选项指出matchname即可使用此match的专用选项进行匹配；
				-p tcp：隐含了-m tcp；
					[!] --source-port,--sport port[:port]：匹配报文中传输层的源端口；
					[!] --destination-port,--dport port[:port]：匹配报文中传输层的目标端口；
					[!] --tcp-flags mask comp
						SYN，ACK，FIN，RST，URG，PSH；	
						
						mask：要检查的标志位列表，以逗号分隔；
						comp：必须为1的标志位，余下的出现在mask列表中的标志位则必须为0；
						
						--tcp-flags  SYN,ACK,FIN,RST  SYN 
					[!] --syn：
						相当于--tcp-flags  SYN,ACK,FIN,RST  SYN 
				-p udp：隐含了-m udp：
					[!] --source-port,--sport port[:port]：匹配报文中传输层的源端口；
					[!] --destination-port,--dport port[:port]：匹配报文中传输层的目标端口；
				-p icmp：隐含了-m icmp:
					 [!] --icmp-type {type[/code]|typename}
						8：echo-request
						0：echo-reply
		显式扩展：必须使用-m选项指出matchname，有的match可能存在专用的选项；
						
			获取帮助：
				CentOS 7：man iptables-extensions
				CentOS 6：man iptables
				
			1、multiport扩展
				以离散或连续的方式定义多端口匹配条件；
				
				 [!] --source-ports,--sports port[,port|,port:port]...：指定多个源端口；
				 [!] --destination-ports,--dports port[,port|,port:port]...：指定多个目标端口；
				 [!] --ports port[,port|,port:port]...：指定多个端口；
				  
			2、iprange扩展
				以连续的ip地址范围指明连续的多地址匹配条件；
				
				[!] --src-range IP[-IP]：源IP地址；
				[!] --dst-range IP[-IP]：目标IP地址；
				
			3、string扩展
				对报文中的应用层数据做字符串匹配检测；
			
				[!] --string pattern：要检测字符串模式；
				[!] --hex-string pattern：要检测的字符串模式，16进制编码；
				--algo {bm|kmp}
				
			4、time扩展
				根据报文到达的时间与指定的时间范围进行匹配度检测；
				
				--datestart YYYY[-MM[-DD[Thh[:mm[:ss]]]]]：起始日期时间；
				--datestop YYYY[-MM[-DD[Thh[:mm[:ss]]]]]：结束日期时间；
				
				--timestart hh:mm[:ss]
				--timestop  hh:mm[:ss]
				
				[!] --monthdays day[,day...]
				[!] --weekdays day[,day...]
				
				~]# iptables -I INPUT -d 172.16.100.67 -p tcp --dport 23 -m time --timestart 09:00:00 --timestop 18:00:00 --weekdays Tue,Thu,Sat -j ACCEPT
				
			5、connlimit扩展
				根据每客户端IP做并发连接数匹配；
				
				--connlimit-upto n：连接数数量小于等于n，此时应该允许；
				--connlimit-above n：连接数数量大于n，此时应该拒绝；
				
				~]# iptables -A INPUT -d 172.16.100.67 -p tcp --dport 23 -m connlimit --connlimit-upto 2 -j ACCEPT
				
			6、limit扩展
				基于收发报文的速率进行匹配；
				
				--limit rate[/second|/minute|/hour|/day]：平均速率
				--limit-burst number：峰值速率
				
			7、state扩展
				状态检测；连接追踪机制（conntrack）；
				
				--state INVALID：无法识别的状态； 
				 ESTABLISHED：已建立的连接；
				 NEW：新连接； 
				 RELATED：相关联的连接；
				 UNTRACKED：未追踪的连接；
				 
				nf_conntrack内核模块；
					追踪到的连接：/proc/net/nf_conntrack文件中；
					
					能追踪的最大连接数量定义在：/proc/sys/net/nf_conntrack_max
						此值可自行定义，建议必要时调整到足够大；
						
					不同的协议的连接追踪的时长：
						/proc/sys/net/netfilter/
						
				[!] --state STATE
				
				如何开放被模式的ftp服务： 
					(1) 装载追踪ftp协议的模块；
						# modprobe nf_conntrack_ftp
						
					(2) 放行命令连接
						~] # iptables -A INPUT -d 172.16.100.67 -p tcp -m state --state ESTABLISHED -j ACCEPT
						~] # iptables -A INPUT -d 172.16.100.67 -p tcp --dport 21 -m state --state NEW -j ACCEPT
					
					(3) 放行数据连接
						~] iptables -A INPUT -d 172.16.100.67 -p tcp -m state --state RELATED -j ACCEPT
									
																			
			处理动作（目标）
				-j targetname [per-target-options]
				
				targetname：
					ACCEPT：接受；
					DROP：丢弃；
					REJECT：拒绝；
					
	保存和重载规则：
		iptables-save > /PATH/TO/SOME_RULE_FILE 
		iptables-restore < /PATH/FROM/SOME_RULE_FILE
		
		CentOS 6：
			保存规则：
				service iptables save
					自动保存规则至/etc/sysconfig/iptables文件中；
			重载规则：
				server iptables restore
					从/etc/sysconfig/iptables文件中重载规则； 
					
	规则优化：
		(1) 可安全放行所有入站及出站，且状态为ESTABLISHED的连接；
		(2) 服务于同一类功能的规则，匹配条件严格的放前面，宽松放后面；
		(3) 服务于不同类功能的规则，匹配报文可能性较大扩前面，较小放后面；
		(4) 设置默认策略；
			(a) 最后一条规则设定；
			(b) 默认策略设定； 
					
		
iptables/netfilter(3)

	iptables/netfilter网络防火墙：
		在FORWARD链上定义规则，注意以下几个问题：
			(1) 请求-响应均经由FORWARD链，要注意规则的方向性；
			(2) 如果可以启用conntrack机制，建议将双方向的状态为ESTABLISHED的报文直接放行；
			
	处理动作：
		ACCEPT，DROP，REJECT
		
		自定义链；
			iptables -N chain_name 新建链
			iptables -X chian_name 删除链
			iptables -E old_name new_name 重命名连接
			
			-j chain_name 引用自定义链，自定义链引用后才会生效
			
			RETURN  执行完自定义链返回主链继续执行 -j RETURN 放在最后一条
	nat表	
		REDIRECT：端口重定向；
			This  target  is only valid in the nat table, in the PREROUTING and OUTPUT chains, and user-defined chains which are  only called from those chains.
			
			--to-ports port[-port]：映射至哪个目标端口； 
			
			~]# iptables -t nat -A PREROUTING -d 172.16.100.67 -p tcp --dport 80 -j REDIRECT --to-port 8088 所有到80端口的请求转到8080
			
		SNAT：
			This  target  is only valid in the nat table, in the POSTROUTING and INPUT chains, and user-defined chains which are only called from those chains. 
			
			 --to-source  [ipaddr[-ipaddr]][:port[-port]] 当访问外网时把本地网络转换成指定地址进行访问
			 
		 MASQUERADE：外网地址为动态获取时
			This target is only valid in the nat table, in the POSTROUTING chain.  It  should  only  be  used  with  dynamically assigned  IP (dialup) connections: if you have a static IP address, you should use the SNAT target.
			
		DNAT：
			This target is only valid in the nat table, in the PREROUTING and OUTPUT chains, and user-defined chains  which  are only  called from those chains.
			
			--to-destination [ipaddr[-ipaddr]][:port[-port]] 需要让互联网中主机访问公司内部某一服务器时可做DNAT.当访问公网IP：port时转发到内部主机进行响应。
			
		LOG：针对某些重要的数据进行日志记录
			--log-prefix
			--log-ip-options
			使用方法：-j LOG --log-prefix “to ping” 注释信息
tcp_wrapper
	库文件：libwrap.so
		tcp包装器；
		工作模式：当访问带有libwrap.so共享库的程序时，会第一时间对/etc/host,allow进行匹配，之后deny文件，完全都不匹配放行通过。
		
		配置文件：/etc/hosts.allow, /etc/hosts.deny
		
		判断一个服务是否能够由tcp_wrapper进行访问控制的方法：
			(1) 动态链接至libwrap.so库：
				ldd - print shared library dependencies
			(2) 静态编译库文件至应用程序中：string命令查看应用程序文件中的字符串中，是否出现了/etc/hosts.allow，/etc/hosts.deny；
				strings - print the strings of printable characters in files.
				
		
	配置的方法格式：
		获取帮助信息：
			'man 5 hosts_options' and 'man 5 hosts_access'
		
		daemon_list: client_list [:options]
			
			daemon_list：
				注意：要给出应用程序文件名；
				
				(1) 单个应用程序文件名称；
				(2) 程序文件名列表，以逗号分隔；
				(3) ALL：所有受tcp_wrapper控制的应用程序；
				
			client_list：
				(1) 单个IP地址或主机名；
				(2) 网络地址：如果有掩码，则必须使用完整格式的掩码；也可以使用简短格式的网络地址，例如172.16.；
				(3) 内建的ACL:
					ALL: 所有主机；
					KNOWN：
					UNKNOWN：
					PARANOID：
					
				练习：设定vsftpd服务仅开放给172.16.网络中的主机访问；
					/etc/hosts.allow
						vsftpd: 172.16.
						
					/etc/hosts.deny
						vsftpd: ALL 
			
				EXCEPT：除了 
			
			[:options]
				deny：拒绝，主要用于hosts.allow文件，定义拒绝访问规则；
				allow：允许，主要用于hosts.deny文件，定义允许访问的规则； 
				spawn：生成，发起，运行；
				
				练习：设定vsftpd服务仅开放给172.16.网络中的主机访问，但不包含172.16.200.2；
				
				练习：判断写在/etc/hosts.deny文件的中下面规则的意义
					vsftpd: ALL EXCEPT 10. EXCEPT 10.10. EXCEPT 10.10.10. EXCEPT 10.10.10.10
				
			
				sshd, vsftpd: ALL : spawn /bin/echo $(date) login attempt from %c to %s >> /var/log/tcpwrapper.log
			 
				%c     Client  information: user@host, user@address, a host name, or just an address, depending on how much information is available.
				%s     Server information: daemon@host, daemon@address, or just a daemon name, depending on how much information  is available.
练习：
	监听协议上的数据：
		 tcpdump -i eth0 -nn 'icmp' icmp：协议可以替换
实例：
iptables -t filter -R INPUT 1 -d 172.16.174.160 -s 172.16.0.0/16 -p icmp -j ACCEPT
	ping   请求：--icmp-type 8  响应：-icmp-type 0
iptables -t filter -I INPUT 1 -d 172.16.174.160 -s 172.16.0.0/16 -p tcp -m multiport --dport 22,80 -j ACCEPT  ?

 iptables -I  INPUT -d 172.16.174.160 -s 172.16.0.0/16 -p icmp --icmp-type 8 -j ACCEPT

iptables -I INPUT -d 172.16.174.160 -p tcp --dport 80 -m string --string admin --algo kmp -j DROP

iptables -I INPUT -d 172.16.174.160 -p icmp --icmp-type 8 -m time --timestart 05:19:00 --timestop 05:30:00 --weekdays Sat -j DROP
	与正常的cst时间 少8小时 utc 时间 是 标准统一时间 中国东八区 +0800 
iptables -I INPUT -d 172.16.174.160 -p icmp --icmp-type 8 -m time --timestart 05:19:00 --timestop 05:30:00 ! --weekdays Sat -j DROP
iptables -A OUTPUT -s 172.16.174.160 -p icmp --icmp-type 0 -m time --timestart 05:09:00 --timestop 06:00:00 --weekdays Sat -j DROP

iptables -t filter -I INPUT -d 172.16.174.177 -p icmp --icmp-type 8 -j LOG --log-prefix "to ping "

iptables -t nat -A PREROUTING -d 172.16.174.177 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.11

iptables -t nat -A PREROUTING -d 172.16.174.177 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.11：8080

iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j SNAT --to-source 172.16.174.177


#如果想要自己访问要加一条规则
	iptables -t filter -I INPUT 5 -i lo -j ACCEPT (OUTPUT)