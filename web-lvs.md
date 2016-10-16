
Linux Cluster:
	
	Cluster：计算机集合、为解决某个特定问题组合起来；
	
	系统扩展的方式：
		Scale Up：向上扩展；
		Scale Out：向外扩展； 
	
	Linux Cluster类型：
		LB：Load Balancing，负载均衡；
			SPoF, 瓶径

		HA：High Availiablity, 高可用；
			A=MTBF/（MTBF+MTTR）
				(0,1), 90%，95%,    
		HP：High Performance，高性能；
			
			www.top500.org
			
		DS：distributed system
		
LB Cluster：

	LB Cluster的实现：
		硬件：
			F5 BIG-IP
			Citrix Netscaler
			A10 A10
			
		软件：
			lvs: Linux Virtual Server
			nginx：
			haproxy:
			ats: apache traffic server
			perlbal：
			pound:
			
		基于工作的协议层次划分：
			传输层：
				lvs
				nginx, haproxy
			应用层：
				http: nginx, haproxy, httpd
				fastcgi: nginx, httpd
				mysql：mysql-proxy, ...
				...
			
		站点指标：
			PV： Page View
			UV：Unique Vistor
			IP：
			
	lvs：
		Linux Virtual Server
			  Real Server
			  
		会话保持			
			Session持久机制：
		 			1、session绑定：session sticky;始终将来自同一个源IP的请求定向至同一个RS；没有容错能力；有损均衡效果；
		 			2、session复制：session replication; (session cluster)在RS之间同步session，每个RS拥有集群中的所有的session；对规模集群不适用；
		 			3、session服务器：session serve利用单独部署的服务器来统一管理集群中的session；
			
		后端Real Server的health check机制
		
		作者：章文嵩；didi，alibaba
		
		l4：四层路由，或四层交换；
			ip:port；根据请求报文的目标IP和目标协议及端口将其调度转发至某RealServer；挑选RealServer将根据调度方法实现； 
			
		iptables:
			iptables: 用户空间规则管理工具；
			netfilter：内核空间让规则生效的组件；
				PREROUTING --> INPUT
				PREROUTING --> FORWARD --> POSTROUTING
				OUTPUT --> POSTROUTING
				
			DNAT：PREROUTING；
			
			ipvs：INPUT链接；
			
		lvs:
			ipvsadm/ipvs
				ipvsadm：用户的空间的命令行工具，规则管理器，用于管理集群服务及RealServer；
				ipvs：工作于内核空间的netfilter的INPUT钩子之上的框架，可接收来ipvsadm的管理命令；
				
				支持基于TCP、UDP、SCTP、AH、ESP、AH_ESP等 协议进行调度；
				
		lvs集群的常用术语：
			vs: virutal server, Director, Dispatcher, Balancer
			rs: real server, backend server, upstream server
			
			CIP: Client IP
			VIP: Virtual Server IP
			DIP: Director IP
			RIP: Real Server IP
			
			CIP <--> VIP <--> DIP <--> RIP
			
		lvs集群类型：
			lvs-nat
			lvs-dr
			lvs-tun
			lvs-fullnat
			
			lvs-nat：
				多目标IP的DNAT，通过将请求报文中的目标地址和目标端口修改为某挑出的RS的RIP和PORT实现转发；
				
				(1) RIP和DIP必须在同一个IP网络，且应该使用私网地址；RS的网关要指向DIP；
				(2) 请求报文和响应报文都必须经由Director转发；Director易于成为系统瓶颈；
				(3) 支持端口映射；VIP上的端口和RIP上的端口不必非得是同一个；
				(4) vs必须是Linux系统，rs可以是任意系统； 
				
			lvs-dr：
				Direct Routing：直接路由；
				
				通过为请求报文重新封装一个MAC首部进行转，源MAC是DIP所在的接口的MAC，目标MAC是某挑选出的RS的RIP所在接口的MAC地址；源IP/PORT，以及目标IP/PORT均保持原状；
				
				(1) 确保前端路由器将目标IP为VIP的请求报文发往Director：
					(a) 在前端路由器绑定；
					(b) 在RS使用arptables; 
					(c) 在RS上修改内核参数限制通告及应答级别；
						arp_announce
						arp_ignore
				(2) RS的RIP可以私网地址，也可是公网地址；RIP与DIP在同一个IP网络；
				(3) RS跟Director要在同一个物理网络，以实现基于MAC地址的转发；
				(4) 请求报文要经由Director，但响应不能经由Director；
				(5) 不支持端口映射；
				
			lvs-tun：tunnel，隧道；
				转发方式：不修改请求报文的IP首部（源IP为CIP，目标IP为VIP），而是在IP报文之外再封装一个IP首部（源IP是DIP，目标IP是RIP），将报文发往挑选出的目标RS；
				
				(1) DIP, VIP, RIP应该都是公网地址；
				(2) RS网关必须不可能指向DIP；
				(3) 请求报文要经由Director，但响应不能经由Director，而是直接发往CIP；
				(4) 不支持端口映射；
				(5) RS必须支持隧道功能；
				
			lvs-fullnat：
				通过同时修改请求报文的源IP地址和目标IP地址进行转发；
					cip --> dip 
					vip --> rip 
					
				(1) VIP是公网地址，RIP和DIP是私网地址，且通常不在同一IP网络；
				(2) RS收到的请求报文源地址为DIP，因响应给DIP即可；
				(3) 请求和响应报文都经由Director；
				(4) 支持端口映射；
				
				注意：默认不支持；
				
	ipvs scheduler：
		根据其调度时是否考虑各RS当前的负载状态，可分为静态方法和动态方法；
		静态方法：仅根据算法本身进行调度，注重起点公平；
			RR：roundrobin，轮询；
			WRR：Weighted RR，加权轮询；
			SH：Source Hashing，源地址哈希，将来自于同一个IP地址的请求始终发往第一次挑中的RS，从而实现了会话绑定；
			DH：Destination Hashing，目标地址哈希，将发往同一个目标地址的请求，始终转发至第一次挑中的RS；
		动态方法：主要根据每RS当前的负载状态进行调度，注重结果公平；
			Overhead = 
		
			LC：least connections，最少连接
				Overhead = activeconns*256+inactiveconns
			WLC：Weighted LC, 加权最少连接
				Overhead = （activeconns*256+inactiveconns）/weight 
			SED：Shortest Expection Delay，最短期望延迟；
				Overhead=(activeconns+1)*256/weight
			NQ：Never Queue
			LBLC：Locality-Based Least connections，基于本地的最少连接；动态的DH算法； 
			LBLCR：LBLC with Replication，
				
					
回顾：
	Linux Cluster:
		LB、HA、HP
		
		LB：
			传输层：lvs，haproxy, nginx 
			应用层：nginx, haproxy, httpd, ats, ...
			
		会话保持：
			session sticky
			session replication
			session server
			
		lvs类型：
			lvs-nat, lvs-dr, lvs-tun, lvs-fullnat
			
		lvs调度方法：
			静态方法：rr, wrr, sh, dh
			动态方法：lc, wlc, sed, nq, lblc, lblcr 
	
		lvs:
			ipvsadm
			ipvs (INPUT)
				
ipvs(2)

	ipvs：内核中TCP/IP协议栈上；
		~]# grep -i -C 10 "ipvs" /boot/config-3.10.0-327.el7.x86_64
		
		集群：
			Service：
			RS：
			
			定义集群：
				定义Service ：把哪一个地址的哪一个端口当做集群服务（一个director可以定义多个服务的集群）
				向Service添加RS：
			管理集群：
				管理service
				管理rs
				查看集群
									
	ipvsadm：
				
		ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]] [-M netmask] [--pe persistence_engine] [-b sched-flags]
		ipvsadm -D -t|u|f service-address
		ipvsadm -C
		ipvsadm -R
		ipvsadm -S [-n]
		ipvsadm -a|e -t|u|f service-address -r server-address [options]
		ipvsadm -d -t|u|f service-address -r server-address
		ipvsadm -L|l [options]
		ipvsadm -Z [-t|u|f service-address]
		ipvsadm --set tcp tcpfin udp
		ipvsadm --start-daemon state [--mcast-interface interface] [--syncid sid]
		ipvsadm --stop-daemon state
		ipvsadm -h	
		
		管理集群服务：增、删、改、查 添加一个director 调度器（vip dip）
			增、改：
				ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]]
					-A：追加；
					-E：编辑；
				注： -s 调度程序算法 rr wrr 等   -p, --persistent [timeout] 持久连接默认300s（或360s）
			删除：
				ipvsadm -D -t|u|f service-address
				
			service-address：
				-t: TCP协议，VIP:TCP_PORT
				-u: UDP协议，VIP:UDP_PORT
				-f：FWM，结合iptables的MARK功能，FWM
				
		管理RealServer：增、删、改、查
			增、改：
				ipvsadm -a|e -t|u|f service-address -r server-address [-g|m|i] [-w weight]
	
			删除：
				ipvsadm -d -t|u|f service-address -r server-address 
				
			server-address:
				rip[:port]
	
			lvs-type：
				-g： gateway，dr，默认； 
				-m: masquerade，nat；
				-i: ipip， tun；
				
		查看：
			ipvsadm -L|l [options]
				-n, --numeric：数值格式显示IP和PORT；
				--exact：精确值；
				
				-c, --connection：显示当前的IPVS连接；
				--stats：显示连接的统计数据；
				--rate：显示IPVS的速率数据；
				
		清空IPVS：
			ipvsadm -C
			
		保存和重载：
			保存：
				ipvsadm-save > /PATH/TO/SOME_RULE_FILE
				ipvsadm -S  > /PATH/TO/SOME_RULE_FILE
				
			重载：
				ipvsadm-restore < /PATH/FROM/SOME_RULE_FILE
				ipvsadm -R < /PATH/FROM/SOME_RULE_FILE
				
		保存规则并于开机时自动载入：
			CentOS 6：
				/etc/sysconfig/ipvsadm :将配置保持在此处
				
				chkconfig  ipvsadm on 开机运行 ipvsadm
			开机启动流程：开机会检查对应的 /etc/rc.d/对应rc#.d的文件 -> init.d/的启动脚本 
				制作启动文件 chkconfig格式的文件
				  		chkconfig [--list] [--type <type>] [name]
				         chkconfig --add <name>
				         chkconfig --del <name>
				         chkconfig --override <name>
         chkconfig [--level <levels>] [--type <type>] <name> <on|off|reset|resetpriorities>
				/etc/sysconfig/ipvsadm-config
				/usr/lib/systemd/system/ipvsadm.service
				/usr/sbin/ipvsadm
			CentOS 7:
				/etc/sysconfig/ipvsadm 
				
				systemctl enable ipvsadm.service
				
				注：检查 /etc/systemd/system 下面的文件 ln -s '/usr/lib/systemd/system/ipvsadm.service' '/etc/systemd/system/multi-user.target.wants/ipvsadm.service
		清空计数器：
			ipvsadm -Z [-t|u|f service-address]
	
	负载均衡的设计要点：
		(1) 会话保持；
		(2) 数据共享；
			共享存储：
				NAS：Network Attached Storage, 文件服务器，访问接口是文件级别；
					NFS， CIFS
				SAN：Storage Area Network，访问接口为块级别；借助于常见的网络技术传输SCSI协议报文的存储解决方案，常见的iSCSI协议；
				DS：distributed storage, 访问接口通常为文件级别，有些存储不兼容FS posix API，不支持挂载；
			数据同步：
				rsync+inotify
				
				课外作业（博客作业）：rsync+inotify实现数据同步；
				
		
		lvs-nat：
		
			设计要点：
				(1) RIP与DIP在同一个IP网络，RIP的网关要指向DIP；
				(2) 支持端口映射；
				
			实践作业（博客）：负载均衡一个php应用；
				测试：(1) 是否需要会话保持；
					    (2) 是否需要共享存储；
					    
		lvs-dr：direct routing 
		
			dr模型中，各主机上均需要配置VIP；解决地址冲突的方式有三种：
				(1) 在前端网关做静态绑定；
				(2) 在各RS使用arptables；
				(3) 在各RS修改内核参数，来限制arp响应和通告的级别；
					限制响应级别：arp_ignore
						0：使用本地任意接口上配置的地址进行响应； 
						1：仅在请求的目标IP配置在本地主机的接收报文接口上时，才给予响应； 只响应请求报文是本地接口的IP的报文时 才响应
						2-8：
					限制通告级别：arp_announce
						0：默认，把本机所有接口的信息向每个接口上的网络通告； 
						1：尽量改名向非本网络通告；
						2：必须避免向非本网络通告；
						
				
				
				设置脚本示例：	
					#!/bin/bash
					#
					vip=172.16.100.71
					mask=255.255.255.255

					case $1 in
					start)
						echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
						echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
						echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
						echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce

						ifconfig lo:0 $vip netmask $mask broadcast $vip up
						route add -host $vip dev lo:0

						;;
					stop)
						ifconfig lo:0 down

						echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
						echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
						echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
						echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
						;;
					*)
						echo "Usage: $(basename $0) start|stop"
						;;
					esac		
			
		博客作业：lvs详细应用；
			讲明白类型、调度方法；并且给nat和dr类型的设计及实现； 
			
	FWM：firewall mark 
		
		在netfilter上，为某些匹配规则匹配到报文添加标记：mangle表；
			打标记方法：
				iptables -t mangle -A PREROUTING -d $vip -p $protocol --dport $port -j MARK --set-mark 6
				
			基于标记定义集群服务：
				ipvsadm -A -f 6 -s rr
				
	lvs persistence：持久连接
	
		基于持久连接模板，实现无论使用任何算法，在一段时间将来自同一个地址的请求始终发往同一个RS；
		
		Port Affinity：
			每端口持久：仅在一段时间内，将对来自于同一个IP地址的访问相同服务的请求发往后端的同一个RS；
				$vip:PORT
			每客户端持久：仅在一佒时间内，将对所有端口的请求始终发往后端的一同一个RS；
				$vip:0 				
			每防火墙标记持久：在一段时间内，将对绑定为同一个FWM的请求发往同一个后端RS；
				FWM 
				
	考虑：
		(1) Diretor不可用，整个系统将不再不用；SPoF；
			高可用：
				keepalived
				heartbeat/corosync
					ldirectord
		(2) 某RS不可用时，Director是否仍将请求调度至此主机？会。
			完整的实现，应该 周期性对各RS的健康状态做检测；
				健康：添加至RS列表，或启用此RS；
				失败：从各RS中移除，或禁用此RS；
		
				检测方式：
					(a) 网络层检测；
					(b) 传输层检测：端口探测；
					(c) 应用层检测：请求某关键资源； 
					