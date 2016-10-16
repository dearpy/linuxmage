HA Cluster（keepalived）：
	
	系统可用性：
		A = MTBF/(MTBF+MTTR)
			(0,1), 95%,
			几个9: 99%, 99.9%, 99.99%, 99.999%
				1%, 0.1%, 0.01%
				
				降低MTTR：冗余（redundant）
				
	高可用服务：
		高可用资源：
			web service:
				ip/httpd/filesystem
				
		network partition：
			隔离设备：
				node：STONITH 节点级别
				资源：fence
				
	HA Cluster实现方案：
		vrrp协议的实现
			keepalived
		ais：完备HA集群
			heartbeat
			corosync
		
	keepalived：
		
		vrrp协议 ：Virtual Routing Redundancy Protocol 完成地址转移
			虚拟路由器（有两个或多个物理设备组成）、
				VRID（有相同VRID的一组路由器构成的虚拟路由器）、
				master（虚拟路由器中承担报文转发任务的路由器）、
				backup（Master出现故障时，能够代替master工作的路由器）、
				VIP（虚拟路由器的IP 一个虚拟路由器可以有一个或多个IP地址）、
				VMAC（虚拟mac地址，一个虚拟路由器拥有一个虚拟mac地址格式为：00-00-5e-00-01-{VRID}）
				优先级(vrrp 根据优先级来确定虚拟路由器中每台路由器的地位)；
			
			抢占式、非抢占式
				 nopreempt 放在实例中instance

			工作模式：
				主/备
				主/主：配置多个virtual router；
			认证试工：
				无认证 
				简单字符串
				MD5
			
		keepalived：
			是vrrp协议的软件实现，原生设计的目的为了高可用ipvs服务；
			ka：
				vrrp协议完成地址流动；
				为vip地址所在的节点生成ipvs规则（定义在配置文件中）；
				为各rs做健康状态检测；
				基于脚本调用接口通过执行脚本完成脚本中定义的功能；
				
			组件：
				控制组件：配置文件分析器
				内在管理：
				IO复用器：
				核心组件：
					vrrp stack
					checkers
					ipvs wrapper
					watch dog
					
			HA Cluster的配置前提：
				(1) 各节点时间必须同步；
					ntp, chrony
					
				(2) 确保iptables及selinux不会成为阻碍；
				
				(3) 各节点之间可通过主机名互相通信（对KA并非必须）；
						给每一个主机起一个标识性的主机名
				(4) 各节点之间的root用户 可以基于密钥认证的ssh互相通信；
				(5)安装ipvsadm管理命令;确保启动keepalived后，可以查看ipvs是否配置成功
		keepalived：
			CentOS 6.4+,  CentOS 7.2
			
			程序环境：
				配置文件：/etc/keepalived/keepalived.conf
				程序文件：/usr/sbin/keepalived
				Unit File：keepalived.service 
						/etc/sysconfig/keepalive
				配置文件组成：

					GLOBAL CONFIGURATION
						Global definations
						static routes
					VRRPD CONFIGURATION
						VRRP synchronization group(s)
						VRRP instance(s)
					LVS CONFIGURATION
						Virtual server group(s)
						Virtual server(s)
						
			配置示例：
				! Configuration File for keepalived

				global_defs {
					notification_email {
						root@localhost
					}
					notification_email_from keepalived@localhost
					smtp_server 127.0.0.1
					smtp_connect_timeout 30
					router_id node1
					vrrp_mcast_group4 224.0.100.18
				}

				vrrp_instance VI_1 {
					state MASTER
					interface eno16777736
					virtual_router_id 15
					priority 100
					advert_int 1
					authentication {
						auth_type PASS
						auth_pass RrpIoZU7
					}
					virtual_ipaddress {
						172.16.100.99
					}
				}
				
			示例2（双主模型）：
				! Configuration File for keepalived

				global_defs {
					notification_email {
						root@localhost
					}
					notification_email_from keepalived@localhost
					smtp_server 127.0.0.1
					smtp_connect_timeout 30
					router_id node1
					vrrp_mcast_group4 224.0.100.18
				}

				vrrp_instance VI_1 {
					state MASTER
					interface eno16777736
					virtual_router_id 15
					priority 100
					advert_int 1
					authentication {
						auth_type PASS
						auth_pass RrpIoZU7
					}
					virtual_ipaddress {
						172.16.100.99
					}
				}

				vrrp_instance VI_2 {
					state BACKUP
					interface eno16777736
					virtual_router_id 16
					priority 98
					advert_int 1
					authentication {
						auth_type PASS
						auth_pass 7r0IoZU7
					}
					virtual_ipaddress {
						172.16.100.88
					}
				}	
				
			配置语法：
				vrrp_instance <STRING> {
					state MASTER|BACKUP
						当前节点在此虚拟路由器上的初始状态； 只能有一个MASTER主机，余下的都应该为BACKUP；
					interface IFACE_NAME
						vrrp实例绑定的接口； 
					virtual_router_id #
						虚拟路径器的VRID，范围是0-255；
					priority 100
						当前主机在此虚拟路径器中的优先级；范围是1-254；
					advert_int #
						vrrp通告时间的间隔；
					
					authentication {     # Authentication block
						# PASS||AH
						# PASS - Simple Passwd (suggested)
						# AH - IPSEC (not recommended))
						auth_type PASS
						# Password for accessing vrrpd.
						# should be the same for all machines.
						# Only the first eight (8) characters are used.
						auth_pass 1234
					}
					
					virtual_ipaddress {
						<IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE> label <LABEL>
						192.168.200.17/24 dev eth1
						192.168.200.18/24 dev eth2 label eth2:1
					}	
					
					track_interface {
						eth0
						eth1
						...
					}					
					配置要监控的网络接口，一旦接口出现故障，则转为FAULT状态； 
				注：定义在实例内部
					
					notify_master <STRING>|<QUOTED-STRING>
					notify_backup <STRING>|<QUOTED-STRING>
					notify_fault <STRING>|<QUOTED-STRING>
					notify <STRING>|<QUOTED-STRING>					
				
				通知脚本示例：
					#!/bin/bash
					#
					contact='root@localhost'

					notify() {
						mailsubject="$(hostname) to be $1, vip floating."
						mailbody="$(date +'%F %T'): vrrp transition, $(hostname) changed to be $1"
						echo "$mailbody" | mail -s "$mailsubject" $contact
					}

					case $1 in
					master)
						notify master
						;;
					backup)
						notify backup
						nginx
						;;
					fault)
						notify fault
						nginx（systemctl restart httpd）
						;;
					*)
						echo "Usage: $(basename $0) {master|backup|fault}"
						exit 1
						;;
					esac		
					
				脚本调用方法示例：放在实例中
					notify_master "/etc/keepalived/notify.sh master"
					notify_backup "/etc/keepalived/notify.sh backup"
					notify_fault "/etc/keepalived/notify.sh fault"				
			
			虚拟服务器：man 5 keepalived.conf
				 virtual_server IP PORT |
				virtual_server fwmark FWM {
					 lb_algo rr|wrr|lc|wlc|lblc|sh|dh
						定义调度方法
					delay_loop <INT>
						服务轮询的时间间隔；检测realserver健康状态的时间间隔
					lb_kind NAT|DR|TUN
						lvs的类型
					persistence_timeout <INT>
						持久连接时长
					 protocol TCP
						服务协议，仅支持TCP
					sorry_server <IPADDR> <PORT>
						所有RS均故障时用于向用户 say sorry的主机；
					 real_server <IPADDR> <PORT> {
						weight <INT>
							权重
						notify_up <STRING>|<QUOTED-STRING>
						notify_down <STRING>|<QUOTED-STRING>
							RS状态改变通知脚本
						 HTTP_GET|SSL_GET {
							
						 }
						 TCP_CHECK {
						 
						 }
					 }
				}
				
				HTTP_GET|SSL_GET {
					url {
						#eg path / , or path /mrtg2/
						path <STRING>
						# healthcheck needs status_code
						# or status_code and digest
						# Digest computed with genhash
						# eg digest 9b3a0c85a887a256d6939da88aabd8cd
						digest <STRING>
						# status code returned in the HTTP header
						# eg status_code 200
						status_code <INT>
					}
			genhash：命令 生成摘要码  -s 服务器 -p 端口 -u url genhash -s 172.16.174.201 -p 80 -u index
					 nb_get_retry <INT>
						重试的次数；
					delay_before_retry <INT>
						重试之前延迟时长；
					connect_ip <IP ADDRESS>
						向当前RS的哪个IP地址发起健康状态检测请求；
					connect_port <PORT>
						向当前RS的哪个PORT发起健康状态检测请求；
					bindto <IP ADDRESS>
						发出健康状态检测请求时使用的源地址；
					 bind_port <PORT>
						源端口
					connect_timeout <INTEGER>
						连接超时时长
				}
				
				 TCP_CHECK {
					 nb_get_retry <INT>
						重试的次数；
					delay_before_retry <INT>
						重试之前延迟时长；
					connect_ip <IP ADDRESS>
						向当前RS的哪个IP地址发起健康状态检测请求；
					connect_port <PORT>
						向当前RS的哪个PORT发起健康状态检测请求；
					bindto <IP ADDRESS>
						发出健康状态检测请求时使用的源地址；
					 bind_port <PORT>
						源端口
					connect_timeout <INTEGER>
						连接超时时长
				}
				
			示例：
		
				! Configuration File for keepalived

				global_defs {
					notification_email {
						root@localhost
					}
					notification_email_from keepalived@localhost
					smtp_server 127.0.0.1
					smtp_connect_timeout 30
					router_id node1
					vrrp_mcast_group4 224.0.100.18
				}

				vrrp_instance VI_1 {
					state MASTER
					interface eno16777736
					virtual_router_id 57
					priority 100
					advert_int 1
					authentication {
						auth_type PASS
						auth_pass 98181111
					}
					virtual_ipaddress {
						172.16.100.71/32 dev eno16777736 brd 172.16.100.71 label eno16777736:0
					}
				}


				virtual_server 172.16.100.71 80 {
					delay_loop 3
					lb_algo rr 
					lb_kind DR
					nat_mask 255.255.0.0
					protocol TCP

					sorry_server 127.0.0.1 80

					real_server 172.16.100.8 80 {
						weight 1
						HTTP_GET {
						url { 
							path /index.html
							#digest 640205b7b0fc66c1ea91c463fac6334d
							status_code 200
						}
						connect_timeout 3
						nb_get_retry 3
						delay_before_retry 3
						}
					}
					real_server 172.16.100.9 80 {
						weight 1
						HTTP_GET {
						url { 
							path /index.html
							#digest 640205b7b0fc66c1ea91c463fac6334d
							status_code 200
						}
						connect_timeout 3
						nb_get_retry 3
						delay_before_retry 3
						}
					}
				}
		
		keepalived调用外部辅助脚本，完成资源监控，并根据监控的结果状态来实现优先动态调整；
			vrrp_script：定义一个资源监控脚本；
				vrrp_script  <STRING> {
					script ""
					interval INT 
					weight -INT 
				}
					script "" 后面可以使命令 也可以是路径 命令成功了 状态返回值为0 失败了为其他 
					失败了 weight -INT 减 int 成功了 就会回到原先的 优先级
					interval INT 每隔多长时间检测一次
					killall 命令在 psmisc包中
			track_script：调用定义的资源监控脚本；
				track_script {
					SCRIPT_NAME
				}
				
			示例：
				! Configuration File for keepalived

				global_defs {
					notification_email {
						root@localhost
					}
					notification_email_from keepalived@localhost
					smtp_server 127.0.0.1
					smtp_connect_timeout 30
					router_id node1
					vrrp_mcast_group4 224.0.100.18
				}

				vrrp_script chk_down {
					script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
					interval 1
					weight -5
				}

				vrrp_script chk_httpd {
					script "killall -0 httpd && exit 0 || exit 1"
					interval 1
					weight -5
				}
		
				vrrp_instance VI_1 {
					state MASTER
					interface eno16777736
					virtual_router_id 57
					priority 100
					advert_int 1
					authentication {
						auth_type PASS
						auth_pass 98181111
					}
					virtual_ipaddress {
						172.16.100.71/32 dev eno16777736
					}

					track_script {
						chk_down
						chk_httpd
					}
					notify_master "/etc/keepalived/notify.sh master"
					notify_backup "/etc/keepalived/notify.sh backup"
					notify_fault "/etc/keepalived/notify.sh fault"
				}
				
		博客作业：以上所有内容；				
				
	实例：高可用 httpd
			总结：iptables 关闭 iptables -F selinux  setenforce 0 vim /etc/selinux/config selinux = disableS
			! Configuration File for keepalived
		
		global_defs {
		   notification_email {
		        root@localhost
		   }
		   notification_email_from keeplived@localhost
		   smtp_server 127.0.0.1
		   smtp_connect_timeout 30
		   router_id LVS_160
		  vrrp_mcast_group4 224.0.174.18
		}
		vrrp_script chk_nginx {
		        script "killall -0 httpd && exit 0 || exit 1"
		        interval 1
		        weight -5
		        fall 3
		        rise 1
		}
		vrrp_instance VI_1 {
		    state BACKUP
		    interface eno16777736
		    virtual_router_id 59
		    priority 99
		    advert_int 1
		    authentication {
		        auth_type PASS
		        auth_pass 1111
		    }
		    virtual_ipaddress {
		        172.16.174.176
		    }
		#nopreempt
		        track_script {
		        chk_nginx
		}
		notify_master "/etc/keepalived/notify.sh master"
		notify_backup "/etc/keepalived/notify.sh backup"
		notify_fault "/etc/keepalived/notify.sh fault"
		}
