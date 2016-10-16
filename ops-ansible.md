自整理
	ansible 可以通过调用外部的角色记录及通知类的操作
	ansible 包括
	host invertory   Playbooks  core modules  custom modules  Plugins   （logging ）
	接受管理主机清单	清单      操作							基于什么连接
		执行流程： 加载 playbook  基于 host invertory 连接被管控主机  用某些模块对主机进行某种操作  之后可以发出通知或记录日志（借助与外界工具）
	主要连接插件 ssh

	同时创建多个目录：
		mkdir -vp {nginx,haproxy,keepalive,varnish,memcache,mysql}/{files,templates,tasks,handlers,vars,meta,default}


运维：Operations

系统安装：
	bare metal：pxe, cobbler pxe（预执行环境）
	virtual machine：
Configuration：
	puppet (ruby)
	saltstack (python)
	chef
	cfengine
	...
Command and Control：
	fabric
	func

程序发布：
	人工智能（手动发布）
	脚本
	发布程序（运维程序）

	要求：
		1、不能影响用户体验；
		2、系统不能停机；
		3、不能导致系统故障或造成系统完全不可用；

	灰度模型：
		主机
		用户

		发布路径：
			/webapps/tuangou
			/webapps/tuangou-1.1
			/wepapps/tunagou-1.2

		在调度器上下线一批主机（标记为维护模式）--> 关闭服务 --> 部署新版本 --> 启动服务
				nginx server-down

		--> 在调度器上启用这一批主机；

ansible：
	Configuration
	Command and Control

	运维工个的分类：
		agent：puppet, func, ...
		agentless（ssh,）：ansible, fabric

	特性：
		模块化：调用特定的模块，完成特定任务；
		基于Python语言实现，由Paramiko, PyYAML和Jinja2三个关键模块；
		部署简单：agentless；
		支持自定义模块；
		支持playbook；

		幂等性；

	安装：
		epel， ansible
			配置文件：/etc/ansible/ansible.cfg
				主机清单：/etc/ansible/hosts

			主程序：
				ansible
				ansible-playbook
				ansible-doc

			ansible的简单使用格式：
				ansible  HOST-PATTERN  -m MOD_NAME  -a  MOD_ARGS
				ansible-doc -s group
	基于ssh连接：使用秘钥认证
		生成秘钥：
				ssh-keygen -t rsa [-P ''] [-f “/root/.ssh/id_rsa"]
		将公钥copy到要连接的服务器，并在.ssh/authorized_keys
				 ssh-copy-id 172.16.174.160


		ansible的常用模块： 注  “=”一般为必须给的参数
			获取模块列表：
				ansible-doc  -l
				ansible all -m ping
			command模块：在远程主机运行命令；
				模块省略为command  不支持管道
				ansible all -a "useradd testuser"

			shell模块：在远程主机在shell进程下运行命令，支持shell特性，如管道等；
				ansible all -m shell -a  'echo mageedu |passwd --stdin'
				一般用到管道 输入输出重定向时 使用shell模块

			copy模块： Copies files to remote locations.
				用法：
					(1) src=  dest=
					(2) content=  dest=
					owner, group, mode
					ansible all -m copy -a "src=  mode 600 dest="
							content='hello \n'
					ansible all -m copy -a "src=/etc/fstab dest=/tmp/fstab1"
					ansible all -m copy -a "content='hello' dest=/tmp/testfile"
			cron 模块：Manage cron.d and crontab entries.
				minute=
				day=
				month=
				weekday=
				hour=
				job=   '/sbin/ntpdate 172.16.174.100 > /dev/nuxll' name=Synctime
				*name=  必须指明
				cron_file
				state=
					present：创建
					absent：删除
					minute=*/5 job=   '/sbin/ntpdate 172.16.174.100 > /dev/nuxll' name=Synctime state=present
				实例：
						ansible all -m cron -a "minute=*/5 job='/sbin/ntpdate 172.16.174.100 &> /dev/null' name=Synctime"
						ansible all -m cron -a "state=Sync"

			fetch模块：Fetches a file from remote nodes
				pull 远程  src在远程注意的文件  dest

			file模块： Sets attributes of files
				用法：
					(1) 创建链接文件：*path=  src=  state=link   path 为 链接后的路径  src= 原文件的路径
					(2) 修改属性：path=  owner= mode= group=
					(3) 创建目录：path=  state=directory
					src=  path=/etc/file.link state=link
			hostname模块：Manage hostname
				name=
				变量 基于循环 多个主机命名

			pip模块：Manages Python library dependencies.
					远程主机自动安装 运行python所用到的工具

			yum模块：Manages packages with the `yum' package manager
				name=：程序包名称，可以带版本号；  默认最新版本
				state=
					present（最新）, latest
					absent 删除
					ansible all -m yum -a "name=httpd state=latest" state=absent
			     conf_file
					name=httpd state=latest
					安装好 把本地配置好的文件copy到其主机

			service模块：管理服务
				*name=
				state=
					started
					stopped
					restarted
				enabled=   开机自动启动
				runlevel=
			 实例：ansible all -m service -a "name=httpd state=started enabled=true"

			uri
			    url=
			user模块：管理用户账号 创建 删除 修改
				*name=
				system=yes|no
				uid=
				shell=
				group=
				groups=  附加组
				comment=	注释信息
				home=
				  remove (删除用户的时候同时删除家目录)   move_home  state=absent
			setup模块：获取facts
				present 创建
             示例：ansible all -m user -a "name=user111 state=present system=yes uid=306"


		YAML:yet another markup language
			YAML：语法格式
				列表 字典：有两个键值对组合而成的就叫做字典（多个kv 对组成）同类的元素：- 可以嵌套（key下 还可以有key）
					列表	 [1,2,3,4]
					字典	 {1：mon,2:tue}  {1：mon,2:[red,bule,yellow]}
				packages：
					- httpd
					- php
					- php-mysql
				version:0.1

		Playbook的核心元素： 通过读取yaml格式的文件 playbook就是由一个或多个play组成的列表
			Hosts：
			Tasks：任务
			Variables: （自带的，自定义）
			Templates：包含了模板语法的文本文件；
			Handlers：由特定条件触发的任务；

			Roles：把hosts剥离出来，playbook 就成了角色

			playbook的基础组件：最基础 hosts tasks user
				Hosts：运行指定任务的目标主机； 一个多个冒号分割
				remote_user: 在远程主机上执行任务的用户；
					定义全局 也可以单任务指定 task
					sudo_user：临时切换
				tasks：任务列表  任务执行 第一个在所有主机运行完，再运行第二个任务
					模块，模块参数；
					格式：
						(1) action: module arguments 较新版本
						(2) module: arguments 通用（建议使用）

						注意：shell和command模块后面直接跟命令，而非key=value类的参数列表； command：

					(1) 某任务的状态在运行后为changed时，可通过“notify”通知给相应的handlers；
					(2) 任务可以通过"tags“打标签，而后可在ansible-playbook命令上使用-t指定进行调用；可以同时多个标签 中间用空格或，号 试一下
						ansible-playbook -t instconf web3.yaml

			运行playbook的方式：
				(1) 测试
					ansible-playbook  --check
						只检测可能会发生的改变，但不真正执行操作；
					ansible-playbook  --list-hosts
						列出运行任务的主机；
					ansible 172.16.174.167 -m setup
				(2) 运行
					ansible-playbook first.yaml
	实例：
						- hosts: all
						  remote_user: root
						  tasks:
						  - name: createuser3
						    user: name=user3 system=yes uid=308
						  - name: createuser4
						    user: name=user4 system=yes uid=309
	实例：
						- hosts: websrv
						  remote_user: root
						  tasks:
						  - name: install httpd package
						    yum: name=httpd state=present
						  - name: install configure file
						    copy: src=files/httpd.conf dest=/etc/httpd/
						  - name: start httpd service
						    service: name=httpd state=started enabled=true
						  - name: execute ss command
						    shell: ss -tanl |grep :80

	实例
						- hosts: websrv
						  remote_user: root
						  tasks:
						  - name: install httpd package
						    yum: name=httpd state=present
						  - name: install configure file
						    copy: src=files/httpd.conf dest=/etc/httpd/conf/
						    notify: restart httpd
						  - name: start httpd service
						    service: name=httpd state=started enabled=true
						#  - name: execute ss command
						#    shell: ss -tanl |grep :80
						  handlers:
						  - name: restart httpd
						    service: name=httpd state=restarted


							可以将shell命令的结果重定向到文件当中。
			handlers：
				任务，在特定条件下触发；（一般为在配置文件修改时，触发某动作执行）
				接收到其它任务的通知时被触发；

			variables：
				(1) facts：可直接调用；setup模块提供的：
				(2) ansible-playbook命令的命令行中的自定义变量：{{pname}}调用变量
					-e VARS, --extra-vars=VARS

				(3) 通过roles传递变量；
				(4) Host Inventory （hosts中定义 实现不同主机拥有不同变量）
					(a) 向不同的主机传递不同的变量；
						IP/HOSTNAME  varaiable=value var2=value2   hname=www2
					实例： （1）[websrv]
								172.16.174.160 hname=www1
								172.16.174.177 hname=www2
								[websrv:vars]
								http_port=80

						  （2）ansible websrv -m hostname -a "name={{ hname }}"
					(b) 向组中的主机传递相同的变量；
						[groupname:vars]
						variable=value    http_port=8080
						在 ansible 中 hosts
					注意：invertory参数：
						用于定义ansible远程连接目标主机时使用的参数，而非传递给playbook的变量；
							ansible_ssh_host
							ansible_ssh_port
							ansible_ssh_user
							ansible_ssh_pass
							ansbile_sudo_pass  **
						...

回顾：运维工具，ansible
运维工具：
	OS Provision：pxe, cobbler,
	Configuration: puppet, saltstack, chef, cfengine, ...
	Command and Control：fabric, func
	Deployment：
	灰度升级 lvs 权重为weight=0 nginx upstreem server  down（注意是否为长连接，查看老的会话是否已经断开连接；keepalive时间超时）
ansible: agentless, ssh
		sudo user 注意
	ansible
	ansible-playbook

	模块：command, shell, cron, copy, file, ping, yum, service, user, setup, hostname, group, script

	playbook基本组件：
		hosts
		remote_user 以哪个用户的身份运行
		tasks
		handlers
		variables
		templates
		roles

ansible(2)

模块：
	group模块：添加或删除组
		*name=
		state=
		system=
		gid=

	script模块：
		-a "/PATH/TO/SCRIPT_FILE"
	将本地的某个脚本传递到远程主机时执行：ansible websrv -m script -a "/.."
	template模块：基于模板方式生成一个文件复制到远程主机
		*src=
		*dest=
		owner=
		group=
		mode=
		*必备参数

playbook的其它组件

	变量：
		ansible facts
		ansible-playbook  -e "var=value" (varname=value)
		host variable: host iventory
		group variable 主机组
			[groupname:vars]
			var=value
		roles
		   在vars 目录下：usrname： daemon
				使用时在 templates 调用即可
		调用：{{ variable }}
			insible-playbook -t instconf -e"username=adm" --check nginx.xml
		在playbook中定义变量的方法：
		vars:
		- var1: value1
		- var2: value2

	模板：templates
		文本文件，嵌套有脚本（使用模板编程语言编写）
			Jinja2（python 的模板编程语言）：
				字面量：
					字符串：使用单引号或双引号；
					数字：整数，浮点数；
					列表：[item1, item2, ...] 可变的
					元组：(item1, item2, ...) 不可变
					字典：{key1:value1, key2:value2, ...} key不一般为字符串 “keys”
					布尔型：true/false

				算术运算：
					+, -, *, /, //（除完只留商）, %, **（次方）

				比较操作：
					==, !=, >, >=, <, <=

				逻辑运算：
					and, or, not

			示例：
			- hosts: websrvs
			remote_user: root
			tasks:
				- name: install nginx
				yum: name=nginx state=present
				- name: install conf file
				template: src=files/nginx.conf.j2 dest=/etc/nginx/nginx.conf
				notify: restart nginx
				tags: instconf
				- name: start nginx service
				service: name=nginx state=started
				handlers:
				- name: restart nginx
				service: name=nginx state=restarted

				模板配置文件 ：nginx.conf.j2
				worker_processes {{ ansible_processor_vcpus }};
				listen {{ http_port }};

	条件测试：
		when语句：在task中使用，jinja2的语法格式(字符串加引号；数字不加引号)
			tasks:
			- name: install conf file to centos7
			  template: src=files/nginx.conf.c7.j2
			  when: ansible_distribution_major_version ==	 "7"
			- name: install conf file to centos6
			  template: src=files/nginx.conf.c6.j2
			  when: ansible_distribution_major_version == "6"

	循环：迭代，需要重复执行的任务；
		对迭代项的引用，固定变量名为”item“
		而后，要在task中使用with_items给定要迭代的元素列表；
			列表方法：
				字符串
				字典：key value

		- name: install some packages
		  yum: name={{ item }} state=present
		  with_items:
		  - nginx
		  - memcached
		  - php-fpm

		- name: add some groups
		  group: name={{ item }} state=present
		  with_items:
		  - group11
		  - group12
		  - group13
		- name: add some users
		  user: name={{ item.name }} group={{ item.group }} state=present
		  with_items:
		  - { name: 'user11', group: 'group11' }
		  - { name: 'user12', group: 'group12' }
		  - { name: 'user13', group: 'group13' }

角色(roles)：
	角色集合：
		roles/ 角色名就是目录名
			mysql/
			httpd/
			nginx/
			memcached/

	每个角色，以特定的层级目录结构进行组织： 角色是与主机分离的，谁用谁调用，只代表一种功能
		mysql/
			files/ ：存放由copy或script模块等调用的文件；
			templates/：template模块查找所需要模板文件的目录；
			tasks/：至少应该包含一个名为main.yml的文件；其它的文件需要在此文件中通过include进行包含；
			handlers/：至少应该包含一个名为main.yml的文件；其它的文件需要在此文件中通过include进行包含；
			vars/：至少应该包含一个名为main.yml的文件；其它的文件需要在此文件中通过include进行包含；
			meta/：至少应该包含一个名为main.yml的文件，定义当前角色的特殊设定及其依赖关系；其它的文件需要在此文件中通过include进行包含；
			default/：设定默认变量时使用此目录中的main.yml文件；

	在playbook调用角色方法1：
		- hosts: websrvs
		  remote_user: root
		  roles:
		  - mysql
		  - memcached
		  - nginx

	在playbook调用角色方法2：传递变量给角色(可以覆盖vars下同名的变量)
		- hosts:
		  remote_user:
		  roles:
		  - { role: nginx, username: nginx }
			键role用于指定角色名称；后续的k/v用于传递变量给角色；

		还可以基于条件测试实现角色调用；
		roles:
		- { role: mariadb, when: "ansible_distribution_major_version == '7' " }
		- { role：mysql，when："ansible_distribution_major_version == '6' "  }
变量生效次序：角色中-
	角色中定义task tasks目录下
		main.yml
		-name: install nginx package
			yum: name=nginx state=present
		-name: install conf file
			template:src=nginx.c7.j2 dest=/etc/nginx/nginx.conf
			notfiy: restart nginx
			tags:instconf
		-name:start nginx
			service: name=nginx state=started enabled=true
	角色中定义handler  handlers目录下
		main.yml
		- name：restart nginx
		 	service: name=nginx state=restart
	角色中定义var  vars 目录下
		变量由变量名和变量值组成 可以在yml中
			-hosts：websrv
			  romte_user:root
			  vars：
			  -username：testuser1
			  -groupname：testgroup1
			 task：
			  -name：create group
			   group：name={{ groupname }} state=present
			  -name: create user
			   user: name={{ username }} state=present
		hosts中定义变量；模板中使用 {{ http_port }}
			/etc/ansible/hosts
			[websrv01]
			172.16.1.100 http_port=80
			172.16.1.101 http_port=8080
			[websrv01:vars]
			http_port=80

		vars目录下
			main.yml
				http_port：80
				username: daemon

	调用nginx角色
	 nginx.yml
	 - host: web-srvs
	   romote_user: root
	   roles:
	   	-nginx
	 角色中传递变量
		- hosts:
		  remote_user:
		  roles:
		  - { role: nginx, username: nginx }  role 角色名 usename 变量名
	调用yml文件
		检测：ansible-playbook -t instconf --check nginx.yml
		执行:去掉check
			传递变量	ansible-playbook -e "hostname=magedu.com" nginx.yml
						ansible-playbook -t instconf -e "username=adm" --check nginx.yml
							-t 调用标签
							-e 变量替
							-check 测试运行
memcached
	roles memcached/tasks

	vim main.yml
		- name：install package
		  yum: name=memcached state=present
		- name: install conf file
		  template: src=memcached.conf.j2 dest=/etc/sysconfig/mecached
		  notify: restart memcached
		  tags: memconf
		- name: start memcached
		  service: name=memcached state=started enabled=true
	vim /handles/main.yml
		- name: restart memcached
		  service: name=memcached state=restarted
	vim lnm.yml
		- hosts:all
		  remote_user:root
		  roles:
		  	-{role: nginx, when:"ansible_distribution_major_version == '6' "}
		  	-{role: memcached,when:ansible_nodename='memcached' }
		  	 	判断条件：ansible 172.16.1.100 -m setup
	注：简写 ansible_memtotal_mb
		定义memcached 角色
			copy 文件到 templates/memcached.conf.j2 中 替换 CACHESIZE="{{ ansible_memtotal_mb//3 }}"
					// 整除 不会余数

实战项目：
	主/备模式高可用keepalived+nginx(proxy)
	两台主机：httpd+php
	一台主机：mysql-server或mariadb-server；

主机变量
	定义hosts
		[web-01]
		172.16.1.100 http_port=8080
		172.16.1.101 http_port=80
组变量
	定义hosts
		[web-01:vars]
		http_port = 80

		{{ ansible_processor_vcpus -1 }}
http://www.ansible.com.cn

