HA Cluster:
自整理：错误的资源 要先看提示后 按提示操作  unmanage 后 delete	
	Linux Cluster: LB/HA/HP
		
	HA: 
		A=MTBF/(MTBF+MTTR)
			MTBF: Mean  Time Between Failure
			MTTR: Mean Time to Repair
			
			冗余：
				Failover, Failback 
				
			1>A>0：百分比 
				95%, 99%, 99.5%, 99.9%, 99.99%, 99.999%
				
		故障场景：
			硬件：
				人为故障
				wear out 
				设计缺陷
				...
			软件：
				人为故障
				bug
				设计缺陷
				...
				
	HA解决方案：
		vrrp协议：keepalived
		SA Forum：AIS，OpenAIS(开放式应用接口标准)
			Hearbeat
			CMAN：Cluster MANager (RHCS)
			Corosync 
				group communication
				
		OpenAIS：
			Messaging Layer (Infrastructure Layer)
				ha-aware
			CRM：(Cluster Resource Manager)
				（非ha-aware）
				管理接口：
					CLI：命令行接口
					GUI：图形用户界面 
						WebGUI
						GUI 
				LRM: Local Resource Manager
			RA: (Resource Agent)
				start|stop|restart|status 
					status: 
						running
						stopped
						
			常见实现：
				Messaging Layer：
					heartbeat 
						v1, v2, v3
					cman 
					corosync
				CRM：
					heartbeat v1：haresources
						配置接口：haresources配置文件；
					heartbeat v2：crm
						运行方式：在集群听每个节点上运行一个crmd守护进程(5560/tcp)，提供API；
						配置接口：crmsh, hb_gui；
					heartbeat v3：pacemaker
						配置接口：crmsh, pcs；
						GUI：hawk(suse), LCMC, pacemaker-gui 
					cman: rgmanager
						配置接口：cluster.conf, system-config-cluster, conga(ricci/luci), cman_tool, ccs_tool, clustat, ...
					corosync: pacemaker
										
				组合方式	：
					heartbeat v1
					heartbeat v2
					heartbeat v3 + cluster-glue + pacemaker
					corosync + pacemaker
						corosync v1 + pacemaker (plugin) 
						corosync v2 (quromsystem) + pacemaker (standalone deamon)
					cman + rgmanager (RHCS)
					cman + corosync + pacemaker 
					
				RHCS（Cluster Suite）：
					RHEL5：cman + rgmanager + conga(ricci/luci)
					RHEL6:
						cman + rgmanager + conga(ricci/luci)
						corosync v1 + pacemaker + crmsh/pcs
						cman + corosync v1 + pacemaker
					RHEL7：
						corosync v2 + pacemaker + pcs
	
		quorum：
			
			failover：即资源所在的active node出现故障时，将其转移至其它可用节点的过程； failback 
			
			network partition: brain-split
			
			quorum：
				投票系统：
					node: vote
					
				with quorum: votes > total/2
				without quorum: votes <= total/2
					no_quorum_policy
						stop
						ignore 
						suicide
						freeze
						
					fencing：
						node level：	STONITH( shooting the other node in the head)
							stonith device：
								hardware
								software
								meatware
						resource level：
							fc switch								
				
				特殊场景：two nodes cluster 
					(1) no_quorum_policy
						ignore
					(2) quorum device
						(a) ping node
						(b) quorum disk
						...
						
			集群事务信息及心跳信息传递方式：
				(1) unicast
				(2) broadcast
				(3) multicast
				
				Messaging Layer
				
			Resource Allocation Layer：资源管理
				DC： Designated Coordinator 
				
				Component:
					DC:
						CRM, CIB, PE, LRM 
					非DC：
						CRM，CIB， LRM 
						
					CIB：Cluster Information Base 
						DC负责接收新配置，并传播给其它节点；
						
				资源类型：
					primitive：基本资源，主资源； 
						仅能运行一份，仅能运行于单个节点；
					group：组
						将resouce组合成一个service；
						功用：组合、次序（对称）
					clone：克隆
						同一资源在集群可出现多个副本；可以运行于多个节点；
					multi-state（master/slave）：
						是克隆类型资源的一特殊表现，存在多个副本，副本间存在主从关系；drbd
						
				集群的架构模型：
					Active/Passive, Active/Active
					N个节点：
						N-M：N个节点，运行M个服务，M<N, 备用节点（N-M）；
						N-N：N个节点，运行N个服务；
						
				资源倾向性：资源的约束关系；
					score (-oo, +oo)
					
					location：位置约束，资源对节点倾向性；
					colocation：排列约束，资源与资源在一起的倾向性；
					order：顺序约束，定义资源间的依赖关系；
					
			RA：资源代理 
				类别：
					LSB：/etc/rc.d/init.d/*
						支持start|stop|restart|status|reload|force-reload；
						注意：一定不能开机自启动；
					OCF：Open Cluster Framework
						/usr/lib/ocf/resource.d/provider/目录下，类似于LSB的脚本，但支持start, stop, status, monitor, meta-data等；
					systemd：unit file，/usr/lib/systemd/system/
						注意：一定要enable；
					STONITH：调用stonith设备的专用RA；
					service：
					
			资源的黏性：
				资源留在当前节点的倾向性；
					
			
回顾：
	HA Cluster：
		vrrp, ais 
		OpeAIS：Messaging Layer, Resource Allocation Layer, RA 
			Messaging Layer：
				heartbeat, cman, openais, corosync
			CRM：
				haresources, crm, pacemaker, rgmanager
			RA：
				LSB, OCF, systemd, STONITH, service
				
		Quorum：
			with quorum: votes > total/2
			without quorum：votes <= total/2
				no_quorum_policy: stop, ignore, suicide, freeze
				
			fencing：
				node：stonith
					hareward, software, meatware
				resource：
					fc switch 
					
		constraint：
			location：-oo, +oo
			colocation：-oo, +oo
			order：-oo, +oo
				mandatory, optional, ...
				
		Resource Type：
			primitive, group, clone, multi-state(master/slave)
			
HA Cluster（2）

	CentOS 7：corosync v2 + pacemaker (standalone deamon) + crmsh/pcs 
	
	安装配置：
		前提：时间同步、基于主机名互相通信、ssh的互信通信
			node1.magedu.com node1
			node2.magedu.com node2
		
		各节点安装相关的程序包：corosync, pacemaker
			# yum -y install corosync pacemaker
		
		corosync：
			程序环境：
				配置文件：/etc/corosync/corosync.conf
				密钥文件：/etc/corosync/authkey
				Unit File：corosync.service 
				
			配置文件格式：
				totem { }
					This top level directive contains configuration options for the totem protocol.
					
					totem协议：即节点间的通信协议，主要定义通信方式、通信协议版本、加密算法...
					interface {}：定义集群心跳信息及事务信息传递的接口，可以有多组；

				logging { }：日志系统； 
					This top level directive contains configuration options for logging.

				quorum { }：投票系统；
					This top level directive contains configuration options for quorum.

				nodelist { }：节点列表； 
					This top level directive contains configuration options for nodes in cluster.

				qb { } 
					This top level directive contains configuration options related to libqb.	
					
			生成密钥文件：
				# corosync-keygen [-l]
				
			配置示例：
				totem {
					version: 2
					crypto_cipher: aes128
					crypto_hash: md5
					interface {
						ringnumber: 0
						bindnetaddr: 172.16.0.0
						mcastaddr: 239.255.101.11
						mcastport: 5405
						ttl: 1
					}
				}
				logging {
					fileline: off
					to_stderr: no
					to_logfile: yes
					logfile: /var/log/cluster/corosync.log
					to_syslog: no
					debug: off
					timestamp: on
					logger_subsys {
						subsys: QUORUM
						debug: off
					}
				}
				quorum {
					provider: corosync_votequorum
				}
				nodelist {
					node {
						ring0_addr: node1.magedu.com
						nodeid: 1
					}
					node {
						ring0_addr: node1.magedu.com
						nodeid: 2
					}
				}			
		
			corosync-cfgtool：管理工具
				-S：显示当前节点ring相关的信息； 
				-R: 控制所有节点重载配置文件；
				-H：
				
			corosync-cmapctl  | grep members
				
		pacemaker：
			启动服务：systemctl  start pacemakerd.service 	
				Unit File： pacemaker.service 
				环境配置文件：/etc/sysconfig/pacemaker 
				
			# crm_mon
	crmsh安装	yum install crmsh-2.1.4-1.1.x86_64.rpm python-pssh-2.3.1-4.2.x86_64.rpm pssh-2.3.1-4.2.x86_64.rpm 	
		配置接口(crmsh/pcs)：
			crmsh：
				运行方式：
					交互式方式：
						crm(live)#
					命令方式：
						crm status
						crm node standby
						crm(live)# node standby
						crm(live)node# standby
			
				获取帮助：ls, help 
					help [KEYWORD]
				查看集群状态信息：
					status 
					
				设定和管理集群：
					cluster
					
				配置CIB：configure 
					Commands:
						acl_target     Define target access rights
						cd             Navigate the level structure
						_test          Help for command _test
						clone          Define a clone
						colocation     Colocate resources
						commit         Commit the changes to the CIB
						default-timeouts Set timeouts for operations to minimums from the meta-data
						delete         Delete CIB objects
						edit           Edit CIB objects
						erase          Erase the CIB
						fencing_topology Node fencing order
						filter         Filter CIB objects
						graph          Generate a directed graph
						group          Define a group
						help           Show help (help topics for list of topics)
						load           Import the CIB from a file
						location       A location preference
						ls             List levels and commands
						modgroup       Modify group
						monitor        Add monitor operation to a primitive
						ms             Define a master-slave resource
						node           Define a cluster node
						op_defaults    Set resource operations defaults
						order          Order resources
						primitive      Define a resource
						property       Set a cluster property
						ptest          Show cluster actions if changes were committed
						quit           Exit the interactive shell
						refresh        Refresh from CIB
						_regtest       Help for command _regtest
						rename         Rename a CIB object
						role           Define role access rights
						rsc_defaults   Set resource defaults
						rsc_template   Define a resource template
						rsc_ticket     Resources ticket dependency
						rsctest        Test resources as currently configured
						save           Save the CIB to a file
						schema         Set or display current CIB RNG schema
						show           Display CIB objects
						_objects       Help for command _objects
						tag            Define resource tags
						up             Go back to previous level
						upgrade        Upgrade the CIB to version 1.0
						user           Define user access rights
						verify         Verify the CIB with crm_verify
						xml            Raw xml
						cib            CIB shadow management
						cibstatus      CIB status management and editing
						template       Edit and import a configuration from a template				
				
				资源管理：resouce 
					Commands:
						cd             Navigate the level structure
						cleanup        Cleanup resource status
						demote         Demote a master-slave resource
						failcount      Manage failcounts
						help           Show help (help topics for list of topics)
						ls             List levels and commands
						maintenance    Enable/disable per-resource maintenance mode
						manage         Put a resource into managed mode
						meta           Manage a meta attribute
						migrate        Migrate a resource to another node
						param          Manage a parameter of a resource
						promote        Promote a master-slave resource
						quit           Exit the interactive shell
						refresh        Refresh CIB from the LRM status
						reprobe        Probe for resources not started by the CRM
						restart        Restart a resource
						scores         Display resource scores
						secret         Manage sensitive parameters
						start          Start a resource
						status         Show status of resources
						stop           Stop a resource
						trace          Start RA tracing
						unmanage       Put a resource into unmanaged mode
						unmigrate      Unmigrate a resource to another node
						untrace        Stop RA tracing
						up             Go back to previous level
						utilization    Manage a utilization attribute					
				
				节点管理：node
					Commands:
						attribute      Manage attributes
						cd             Navigate the level structure
						clearstate     Clear node state
						delete         Delete node
						fence          Fence node
						help           Show help (help topics for list of topics)
						ls             List levels and commands
						maintenance    Put node into maintenance mode
						online         Set node online
						quit           Exit the interactive shell
						ready          Put node into ready mode
						show           Show node
						standby        Put node into standby
						status         Show nodes' status as XML
						status-attr    Manage status attributes
						up             Go back to previous level
						utilization    Manage utilization attributes				
			
			查看资源代理相关属性：ra命令
				classes：类别列表
				list  CLASSES  [PROVIDER]：获取指定类别的资源代理列表
				info  [<class>:[<provider>:]]<type> 
				
				注意：lsb及systemd类别一般没有属性；
				使用：进入ra -> classes -> list classes->info ocf:heartbeat:IPaddr
			全局属性：
				property：
					stonith-enabled={true|false}
					no-quorum-policy={stop|suicide|freeze|ignore}
					default-resource-stickiness=
						
			
			定义一个资源：
				primitive：
					primitive <rsc> [<class>:[<provider>:]]<type>   [params   <attr>=<val> [<attr>=<val>...]]
						<rsc>：资源的ID，字符串数据；
						{[<class>:[<provider>:]]<type> 资源代理
							class：RA类别
							provdier：提供者
							type：资源代理
							
				group：
					group <name> <rsc> [<rsc>...]
					
			定义约束：
				位置约束：
					location <id> rsc  <score>: <node>
				
				排列约束：
					colocation <id> <score>: <rsc>[:<role>] <with-rsc>[:<role>]
					
				顺序约束：
					order <id> [{kind|<score>}:] first then [symmetrical=<bool>]
					
					kind :: Mandatory | Optional | Serialize
					
			配置示例：
				primitive webip IPaddr \
					params ip=172.16.100.99
				primitive webserver systemd:httpd
				primitive webstore Filesystem \
					params device="172.16.100.6:/webdata/html" directory="/var/www/html" fstype=nfs
				group webservice webstore webip webserver	
				
			资源监控：
				monitor
				primitive ... op monitor interval= timeout=
				
			课外实践（博客作业）：
				mariadb的高可用实现；
				
		corosync高可用ipvs Director的解决方案：corosync + ldirectord；
		
	pcs: 全生命周期管理工具；
		pcs/pcsd：
			pcsd：运行于HA Cluster的各节点；
			pcs：pcsd服务的命令行客户端；
				
		 pcs命令：

			Usage: pcs [-f file] [-h] [commands]...
			Control and configure pacemaker and corosync.	
			
			Commands:
				cluster     Configure cluster options and nodes
				resource    Manage cluster resources
				stonith     Configure fence devices
				constraint  Set resource constraints
				property    Set pacemaker properties
				acl         Set pacemaker access control lists
				status      View cluster status
				config      View and manage cluster configuration
				pcsd        Manage pcs daemon			
					
			课外实践（博客作业）：
				haproxy的高可用实现；


HA：High Avaliablity
	
	HA: 平均无故障时间/(平均修复时间+平均无故障时间)
		提高系统可用性：
			缩短平均修复时间（冗余机制）
			延长平均平均无故障时间

		集群：
			手动切换
			自动切换

		切换：
			failover: 故障切换
			failback: 

		资源：
			vip: float ip
			ipvs规则

		集群服务：构建的方式
			分组：
			约束：

	解决方案：
		vrrp+script: keepalived
		ais:
			heartbeat
			corosync
			cman(openais)

		服务的类型：
			no ha-aware
			ha-aware

	HA的框架：
		Messaging Layer: 基础事务层
			heartbeat v1, v2, v3
			corosync(openAIS)
			cman(openAIS)

		CRM: Cluster Resource Manager
			heartbeat v1: 自带资源管理器haresources（配置接口：配置文件，文件名也叫haresources）
			heartbeat v2: 自带资源管理器crm （各节点运行crmd进程，配置接口：命令行客户端crmsh，GUI客户端hb-gui）
			heartbeat v3 = heartbeat + pacemaker + cluster-glue 
				packmaker: 
					CLI： crm(SuSE), pcs
					GUI: hawk, LCMC, pacemaker-mgmt
			cman + rgmanager:
				resource group manager：Failover Domain, node priority
				配置接口：
					clustat, cman_tool
					Conga: luci+ricci

		LRM: Local Resource Manager

		RA：Resource Agent
			heartbeat legacy: heartbeat和传统类型，通常是/etc/ha.d/haresources.d/目录下的脚本；
			LSB: /etc/init.d/*
			OCF(Open Cluster Framework)：
				provider:
			STONITH: 

		资源隔离机制：
			节点级别：STONITH
				电源交换机
				服务硬件管理模块
			资源级别：


		quorum： 法定票数(大于总票数的一半)
			用来判定集群分裂的场景中，某些节点是否可以继续以集群方式运行；

			仲裁设备：
				ping node
					ping node group
				quorum disk: qdisk

	CentOS或RHEL系统高可用集群的解决方案：

		CentOS 5:
			RHCS：cman+rgmanager
			选用第三方方案：corosync+pacemaker, heartbeat(v1或v2), keepalived

		CentOS 6:
			RHCS: cman+rgmanager
			corosync + rgmanager
			cman + pacemaker
			heartbeat v3 + pacemaker
			keepalived

	配置高可用集群的前提：（以两节点的heartbeat为例）
		1、时间必须保持同步
			使用ntp服务器
		2、节点必须名称互相通信
			解析节点名称
			/etc/host
			集群中使用的主机名为`uname -n`表示的主机名；
		3、ping node
			仅偶数节点才需要；
		4、ssh密钥认证进行通信；

	案例：heartbeat: httpd高可用
		需要几个资源？
			ip
			httpd

		node1.magedu.com: 172.16.100.7
		node2.magedu.com: 172.16.100.8

		ip: 172.16.100.23
		httpd: 
			(1) 安装好httpd程序
			(2) web资源做好准备
			(3) 测试启动并关闭；确保其开机不会自动启动；

	heartbeat v1的配置：
		主配置文件：ha.cf
		认证密钥：authkeys, 其权限必须为组和其它无权访问；
		用于资源的文件：haresources

	博客：基于heartbeat v1配置mysql的基于nfs共享数据的高可用集群；







	安装配置：

		# yum install perl-TimeDate net-snmp-libs libnet PyXML
		# rpm -ivh heartbeat-2.1.4-12.el6.x86_64.rpm heartbeat-pils-2.1.4-12.el6.x86_64.rpm heartbeat-stonith-2.1.4-12.el6.x86_64.rpm

		heartbeat:　694/udp



		ha.cf配置文件部分参数详解：

			autojoin    none
				#集群中的节点不会自动加入
			logfile /var/log/ha-log
				#指名heartbaet的日志存放位置
			keepalive 2
				#指定心跳使用间隔时间为2秒（即每两秒钟在eth1上发送一次广播）
			deadtime 30
				#指定备用节点在30秒内没有收到主节点的心跳信号后，则立即接管主节点的服务资源
			warntime 10
				#指定心跳延迟的时间为十秒。当10秒钟内备份节点不能接收到主节点的心跳信号时，就会往日志中写入一个警告日志，但此时不会切换服务
			initdead 120
				#在某些系统上，系统启动或重启之后需要经过一段时间网络才能正常工作，该选项用于解决这种情况产生的时间间隔。取值至少为deadtime的两倍。
			    
			udpport 694
				#设置广播通信使用的端口，694为默认使用的端口号。
			baud    19200
				#设置串行通信的波特率       
			#bcast   eth0        
				# Linux  指明心跳使用以太网广播方式，并且是在eth0接口上进行广播。
			#mcast eth0 225.0.0.1 694 1 0
				#采用网卡eth0的Udp多播来组织心跳，一般在备用节点不止一台时使用。Bcast、ucast和mcast分别代表广播、单播和多播，是组织心跳的三种方式，任选其一即可。
			#ucast eth0 192.168.1.2
				#采用网卡eth0的udp单播来组织心跳，后面跟的IP地址应为双机对方的IP地址
			auto_failback on
				#用来定义当主节点恢复后，是否将服务自动切回，heartbeat的两台主机分别为主节点和备份节点。主节点在正常情况下占用资源并运行所有的服务，遇到故障时把资源交给备份节点并由备份节点运行服务。在该选项设为on的情况下，一旦主节点恢复运行，则自动获取资源并取代备份节点，如果该选项设置为off，那么当主节点恢复后，将变为备份节点，而原来的备份节点成为主节点
			#stonith baytech /etc/ha.d/conf/stonith.baytech
				# stonith的主要作用是使出现问题的节点从集群环境中脱离，进而释放集群资源，避免两个节点争用一个资源的情形发生。保证共享数据的安全性和完整性。
			#watchdog /dev/watchdog
				#该选项是可选配置，是通过Heartbeat来监控系统的运行状态。使用该特性，需要在内核中载入"softdog"内核模块，用来生成实际的设备文件，如果系统中没有这个内核模块，就需要指定此模块，重新编译内核。编译完成输入"insmod softdog"加载该模块。然后输入"grep misc /proc/devices"(应为10)，输入"cat /proc/misc |grep watchdog"(应为130)。最后，生成设备文件："mknod /dev/watchdog c 10 130" 。即可使用此功能
			node node1.magedu.com  
				#主节点主机名，可以通过命令“uname –n”查看。
			node node2.magedu.com  
				#备用节点主机名
			ping 192.168.12.237
				#选择ping的节点，ping 节点选择的越好，HA集群就越强壮，可以选择固定的路由器作为ping节点，但是最好不要选择集群中的成员作为ping节点，ping节点仅仅用来测试网络连接
			ping_group group1 192.168.12.120 192.168.12.237
				#类似于ping  ping一组ip地址
			apiauth pingd  gid=haclient uid=hacluster
			respawn hacluster /usr/local/ha/lib/heartbeat/pingd -m 100 -d 5s
				#该选项是可选配置，列出与heartbeat一起启动和关闭的进程，该进程一般是和heartbeat集成的插件，这些进程遇到故障可以自动重新启动。最常用的进程是pingd，此进程用于检测和监控网卡状态，需要配合ping语句指定的ping node来检测网络的连通性。其中hacluster表示启动pingd进程的身份。
			
			#下面的配置是关键，也就是激活crm管理，开始使用v2 style格式
			# crm respawn
				#注意，还可以使用crm yes的写法，但这样写的话，如果后面的cib.xml配置有问题
				#会导致heartbeat直接重启该服务器，所以，测试时建议使用respawn的写法
			#下面是对传输的数据进行压缩，是可选项
			compression     bz2
			compression_threshold 2

			注意，v2 style不支持ipfail功能，须使用pingd代替

		资源文件(/etc/ha.d/haresources)
		node1   IPaddr::192.168.1.100/24/eth0   httpd

		认证文件(/etc/ha.d/authkeys)
		auth 1
		1 crc






	补充资料：组播IP地址

		组播IP地址用于标识一个IP组播组。IANA（internet assigned number authority）把D类地址空间分配给IP组播，其范围是从224.0.0.0到239.255.255.255。如下图所示（二进制表示），IP组播地址前四位均为1110八位组⑴ 八位组⑵ 八位组⑶ 八位组⑷1110
		XXXX XXXXXXXX XXXXXXXX XXXXXXXX组播组可以是永久的也可以是临时的。组播组地址中，有一部分由官方分配的，称为永久组播组。永久组播组保持不变的是它的ip地址，组中的成员构成可以发生变化。永久组播组中成员的数量都可以是任意的，甚至可以为零。那些没有保留下来供永久组播组使用的ip组播地址，可以被临时组播组利用。
		224.0.0.0～224.0.0.255为预留的组播地址（永久组地址），地址224.0.0.0保留不做分配，其它地址供路由协议使用。
		224.0.1.0～238.255.255.255为用户可用的组播地址（临时组地址），全网范围内有效。
		239.0.0.0～239.255.255.255为本地管理组播地址，仅在特定的本地范围内有效。常用的预留组播地址列表如下：
		224.0.0.0 基准地址（保留）
		224.0.0.1 所有主机的地址
		224.0.0.2 所有组播路由器的地址
		224.0.0.3 不分配
		224.0.0.4dvmrp（Distance Vector Multicast Routing Protocol，距离矢量组播路由协议）路由器
		224.0.0.5 ospf（Open Shortest Path First，开放最短路径优先）路由器
		224.0.0.6 ospf dr（Designated Router，指定路由器）
		224.0.0.7 st (Shared Tree，共享树）路由器
		224.0.0.8 st主机
		224.0.0.9 rip-2路由器
		224.0.0.10 Eigrp(Enhanced Interior Gateway Routing Protocol，增强网关内部路由线路协议）路由器　224.0.0.11 活动代理
		224.0.0.12 dhcp服务器/中继代理
		224.0.0.13 所有pim (Protocol Independent Multicast，协议无关组播）路由器
		224.0.0.14 rsvp （Resource Reservation Protocol，资源预留协议）封装
		224.0.0.15 所有cbt 路由器
		224.0.0.16 指定sbm（Subnetwork Bandwidth Management，子网带宽管理）
		224.0.0.17 所有sbms
		224.0.0.18 vrrp（Virtual Router Redundancy Protocol，虚拟路由器冗余协议）
		239.255.255.255 SSDP协议使用


回顾：HA基础原理、heartbeat v2
	HA基础原理：
		Messaging Layer：心跳信息、集群事务信息
			解决方案：heartbeat, corosync, cman(openais)
		CRM：承上启下，将那些本身无ha能力的资源旋转于Messaging Layer层之上使其信息层的功能；
			CRM类型：
				haresources(hb v1)：配置接口为直接编辑其配置文件haresources
				crm (hb v2)：守护进程crmd（crmsh-->crmd），监听在指定的端口；配置文件cib（cluster information base），xml格式；配置接口：GUI（hb_gui）；
				pacemaker (hb v3): 守护进程crmd （crmsh, pcs --> crmd）；配置文件cib.xml；配置接口：CLI(crm,pcs), GUI(hawk,LCMC)；
				rgmanager : 守护进程(/etc/cluster/cluster.xml)，配置接口Conga(Web GUI); 控制端luci, 被控制端ricci
			LRM：负责执行本地与资源action相关的事务；
		RA：Resource Agent classes
			heartbeat legacy: /etc/ha.d/haresource.d/
			LSB: /etc/init.d/
			OCF：
				provider
			STONITH：控制硬件（软件，肉件）实现资源隔离的脚本或程序；

		资源隔离：
			节点级别：STONITH
			资源级别：fencing

		集群会分裂：
			两种状态之一：法定票数(大于总票数的一半)
				with quorum
				without quorum

			without quorum之时，如何采取对资源管控的策略：
				stopped
				ignore
				freeze
				suicide

	HA集群的工作模型：
		A/P：两节点，active, passive；工作于主备模型；
		A/A：两节点，active/active，双主模型；
		N-M：N>M, N个节点，M个服务；活动节点数为N，备用节点数为N-M
		N-N：

	资源运行的倾向性（资源转移倾向性）：
		rgmanager:
			failover domain, node priority
		pacemaker：
			资源黏性：资源倾向于留在当前的分数（-oo, +oo）
			资源约束：
				位置约束：资源对某节点运行的倾向性（-oo, +oo）
					inf: 正无穷
					-inf: 负无穷
					n: 
					-n:
				排列约束：定义资源彼此间的倾向性（是否在一起）
					inf: 
					-inf: 
					n, -n
				顺序约束：属于同一服务的多个资源运行在同一节点时，其启动及关闭的次序约束；
					A --> B --> C
					C --> B --> A

			例如：定义高可用的web服务，有三个资源vip, httpd, nfs，思考，如何限制它们要运行于同一节点，并且如何按vip-->nfs-->httpd的次序启动？
				排序约束：inf
				顺序约束：mandatory

	资源类型：
		primitive, native: 主资源，其仅能运行某一节点
		group: 组资源，可用于实现限制多个资源运行于同一节点及对此些资源统一进行管理
		clone: 克隆资源，一个资源可以运行于多个节点；
			应该指定：最大克隆的份数，每个节点最多可以运行的克隆；
		master/slave: 主从资源，特殊的克隆资源；

	pacemaker的crm的角色：
		DC：Designated Coordinator


	案例：如何将director配置为高可用？
		需要哪些资源：vip, ipvs规则
			vip RA: IPaddr, IPaddr2
			ipvs rules RA：
				/etc/rc.d/init.d/ipvsadm
				自定义脚本放置于/etc/init.d/


			vip: 172.16.100.24
			ipvs rules RA: /etc/init.d/ipvsadm

	练习：基于heartbeat v2 crm实现配置基于nfs的mysql HA集群；


ansible:
	
	运维工具
		agent: 通常基于ssl实现，例如puppet, funct等
		agentless: 通常基于ssh实现，例如fabric, ansible等

	幂等性：
	期望状态：

	官方站点：www.ansible.com/home

	# ansible <host pattern> [-m MODULE] -a 'MODULE_ARGS'

	模块：
		command
		user
		copy
		cron
		file
		filesystem
		group
		hostname
		ping
		yum
		service
		shell
		script

	获取模块帮助：
		ansible-doc -l
		ansible-doc MODULE_NAME

	Playbooks:
		Tasks: 
			任务：由各模块所支持执行的特定操作；
				-m user -a 'name= password='
		Variables: 
			变量：
		Templates: 
			模板：
				文本文件模板：使用模板语言来定义
		Handlers: 
			处理器：
				事先定义好的可以在某些条件下被触发的操作；
		Roles: 
			角色：
				层次型组织playbook及其所依赖的各种资源的一种机制；
				角色可被单独调用；

	博客：基于role的方式定义安装配置LAMP平台

回顾：ansible: agentless(ssh, paramiko)
	OS config
	deployment
	task execution

	structure:
		Host Inventory
		Modules：command, copy, file, user, group, shell, script, cron, yum, service, hostname, fetch, filesystem
			ansible-doc -l
			ansible-doc MODULE_NAME
			ansible-doc -s MODULE_NAME
		playbooks
			tasks
			variables
			templates
			handlers
		roles
			common
				files
				vars
				templates
				handlers
				tasks
				meta
				default
			web
			db
			cache

		条件测试：when
			比较表达式：变量、facts
			此前某任务的执行状态
		迭代：
			{{ item.key1 }}, {{ item.key2 }}
			with_items:
				- 
				-
				-

HA：heartbeat
	
	RA的classes:
		Legacy heartbeat v1
		LSB
		OCF
			provider
		STONITH

	Resource的类型：
		primitive(native)
		group
		clone
		master/slave

	HA Service：
		vip
		httpd
		filesystem

		两种方案：
			group
			constraint
				location
				order
				colocation

				score:
					inf
					-inf

		
	资源隔离：
		节点级别：STONITH
		资源级别：fencing

		应用场景：集群分裂(split-brain)

		必要条件：没有隔离设备，禁止使用HA；

	HA的部署方案：
		heartbeat v1 + haresources
		heartbeat v2 + crm
		heartbeat v3 + cluster-glue + pacemaker
		corosync + cluster-glue + pacemaker
		cman + rgmanager
		keepalived + script

	配置HA的前提：
		时间同步、基于主机名互相通信、SSH互信

		corosync, pacemaker

		pacemaker的配置接口：
			crmsh
			pcs

	crmsh: 
		crm命令两种模式：
			交互式模式
			命令模式


		crm(live)# configure 
		crm(live)configure# show
		node node1.magedu.com \
			attributes standby="off"
		node node2.magedu.com \
			attributes standby="off"
		primitive webip ocf:heartbeat:IPaddr \
			params ip="172.16.100.13" \
			op monitor interval="30s" timeout="20s"
		primitive webserver lsb:httpd \
			op monitor interval="30s" timeout="20s"
		primitive webstore ocf:heartbeat:Filesystem \
			params device="172.16.100.6:/web/htdocs" directory="/var/www/html" fstype="nfs" \
			op monitor interval="60s" timeout="40s" \
			op start timeout="60s" interval="0" \
			op stop timeout="60s" interval="0"
		group webservice webip webstore webserver
		order webip_before_webstore_before_webserver inf: webip webstore webserver
		property $id="cib-bootstrap-options" \
			dc-version="1.1.10-14.el6-368c726" \
			cluster-infrastructure="classic openais (with plugin)" \
			expected-quorum-votes="2" \
			stonith-enabled="false" \
			no-quorum-policy="ignore" \
			last-lrm-refresh="1410517422"

回顾：
	ansible
	corosync + pacemaker + crmsh(pcs)

	cib.xml, crmsh

	1、Cluster options
	2、resource defaults 
	3、resource
		primitive
		group
		clone
		master/slave
	4、constraint
		location
		order
		colocation

	crmsh: 
		resource
		ra
		configure
		node
		status

	硬盘接口：
		IDE
		SCSI
			Target:
				LUN：Logical Unit Number

		SATA
		SAS
		USB

	DAS：Direct Attached Storage
	NAS：Network Attached Storage
	SAN：Storage Area Network
		iSCSI

	drbd: 
		两种工作模式：
			primary/secondary
				primary: r/w
				secondary：不允许挂载
					traditional filesystem
			primary/primary
				primary: r/w
					Cluster FileSystem

	drbd的配置有三段：
		全局段：
		通用配置段（common）：为各res提供共享配置
		资源专用配置段(通过定义资源进行): 为特定的res提供专用配置
	
		node node1.magedu.com \
			attributes standby="off"
		node node2.magedu.com \
			attributes standby="off"
		primitive mydrbd ocf:linbit:drbd \
			params drbd_resource="mydata" \
			op monitor role="Master" interval="10s" timeout="20s" \
			op monitor role="Slave" interval="20s" timeout="30s" \
			op start timeout="240s" interval="0" \
			op stop timeout="100s" interval="0"
		primitive myfs ocf:heartbeat:Filesystem \
			params device="/dev/drbd1" directory="/mydata" fstype="ext4" \
			op monitor interval="20s" timeout="60s" \
			op start timeout="60s" interval="0" \
			op stop timeout="60s" interval="0"
		primitive myip ocf:heartbeat:IPaddr \
			params ip="172.16.100.18" \
			op monitor interval="30s" timeout="20s"
		primitive myserver lsb:mysqld \
			op monitor interval="30s" timeout="20s"
		group myservice myip myfs myserver \
			meta target-role="Started"
		ms ms_mydrbd mydrbd \
			meta master-max="1" master-node-max="1" clone-max="2" clone-node-max="1" notify="True"
		colocation myfs_with_ms_mydrbd_master inf: myfs ms_mydrbd:Master
		colocation myserver_with_myfs inf: myserver myfs
		order myfs_after_ms_mydrbd_master inf: ms_mydrbd:promote myfs:start
		property $id="cib-bootstrap-options" \
			dc-version="1.1.10-14.el6-368c726" \
			cluster-infrastructure="classic openais (with plugin)" \
			expected-quorum-votes="2" \
			stonith-enabled="false" \
			no-quorum-policy="ignore"		

	博客：mysql + drbd + corosync

回顾：
    Nginx:
    	web服务器
    	http, mail反向代理 (proxy)

    	C10k：1W

    	8cores, 2cores, 6cores

Cluster: 
	iSCSI:
		p/s, p/p(cluster filesystem)

	DAS: 
		(ATA)IDE: 133MB/s
		SATA:
			II: 3Gbps
			III: 6Gbps

		SCSI: Small Computer System Interface
			Controller
			Host Adapter

			UltraSCSI-320: 320MB/s
			UltraSCSI-640: 640MB/s

		SAS: 6Gbps

		USB, eSATA, 1394

	NAS: Network Attached Storage
		文件服务器
			NFS, CIFS(SMB)

	SAN: Storage Aera Network
		FC: FC SAN
		Ethernet: IP SAN
			iSCSI

	SCSI, iSCSI

	scsi-target-utils: 有两种配置方式
		命令行配置方式: tgtadm
		基于配置文件的配置方式：
			编辑配置/etc/tgt/targets.conf，定义target和lun等
			启动tgtd服务
				由tgt-admin命令读取并设定

		认证机制：
			基于IP认证
			基于用户认证：CHAP

		监听于：tcp，3260端口

	iscsi-initiator-utils:
		主配置文件：/etc/iscsi/iscsid.conf
		配置工具：iscsiadm
		服务脚本：iscsi, iscsid


	iqn: iscsi qualified name

	tgtadm命令：
		模式化的命令：target, logicalunit, account
			target: 管理target
			logicalunit：
			accout: 管理用户帐号

		target模式的管理命令：
			new
			delete
			update
			show
			bind
			unbind

			iqn格式的名称：
				iqn.yyyy-mm.reverse-domain-name:string[.substring]

		logicalunit模式的管理命令：
			new
			delete

	<target iqn....>
		backing-store 
		backing-store 
		...
		initiator-address 
	</target>

回顾：scsi & iscsi
	scsi: 协议，分层
		应用层
		传输层
		物理层
	iscsi：tcp/ip网络
	iscsi server: target, lun
		CentOS: scsi-target-utils
	iscsi initiator:
		CentOS: iscsi-initiator-utils

	scsi-target-utils
		tgtadm: target, logicalunit, account
			--op {new|delete|update|show|bind|unbind}
			--op {new|delete}
			--op {new|delete}

		tgt-admin: /etc/tgt/targets.conf
			tgtd服务脚本

		3260

	iscsi-initiator-utils
		iscsiadm
			-m {discovery|node|session}
			-t st
			-p 

	NAS, SAN:
		Openfiler、FreeNAS、Nexentore

Keepalived:
	HA:
		heartbeat、corosync
		keepalived：lvs(director: HA, ipvs rules, health check)

	vrrp: virtual redundent routing protocol

		生成路由：
			静态路由
			动态路由：OSPF, RIP2

	适用场景：
		lvs: keepalived
		nginx, haproxy(reverse proxy): keepalived

	节点类型：
		MASTER
		BACKUP
		Initialized

	25/tcp: smtp
		127.0.0.1:25

		mail

	vrrp instance、dual master、notfiy_master、nginx

	定义检测脚本：
		使用单独的配置段定义检测机制
			vrrp_script CHK_NAME {
				script "/path/to/somefile.sh"  
				interval #
				weight -5
				fall 3
				rise 1
			}

		在实例调用定义的检测机制，其才能生效
			vrrp_instance NAME {
				track_script {
					CHK_NAME
				}
			}


	博客：keepalived高可用反向代理的nginx; 双主模型高可用ipvs；

	获取帮助：
		man keepalived.conf
		/usr/share/doc/keepalived-*/


回顾：keepalived
	vrrp, checker, ipvs wrapper, vrrp_script, track_script, 
	notify_master, notify_backup, notify_fault

	keepalived + nginx/ipvs/haproxy

	vrrp: 虚拟冗余路由协议

	ipvs: 生成规则，检查RS health check

Cluster
	LB:
		4: ipvs
		7: nginx
		7: haproxy, varnish, tomcat
	HA:
		heartbeat
		corosync
		keepalived	