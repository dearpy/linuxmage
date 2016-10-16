
	Nginx：
		engine X
	web服务器：http、https协议、句
		反代服务器：http reverse proxy, smtp/pop3 reverse proxy
			smtp: simple mail transmission protocol；
			pop3: post office protocol
			imap4: internet mail access protocol

			MIME: Multipurpose Internet Mail Extensions
				major/minor:
					text/html, text/plain, ..	可以参考 mime下的类型 主要讲：类型/子类型 application/x-javascript;
		web资源：URL
			scheme://host:port/path/to/source

			Method：
				GET、HEAD
				POST
				PUT、DELETE (WebDAV)
				OPTIONS
				TRACE
				i
		http事务：
			request <--> response
				stateless：
					Set-Cookie, Set-Cookie2
					Cookie, Cookie2

					session

				request：
					<method> <request-url> <version>
					<HEADERS>

					<body>

				response:
					<version> <status> <reason phrase>
					<HEADERS>

					<body>


					status code：
						1xx
						2xx：成功类的响应码，200, OK
						3xx：重定向类的响应码，301, 302, 304
						4xx：客户端错误，403，404，
						5xx：服务器端错误：502, bad gateway

			认证：basic, digest

		httpd: MPM
			prefork：主进程负责生成子进程，每个子进程响应一个请求；
			worker：主进程负责生成子进程，子进程负责生成线程，每个线程响应一个请求；
			event：主进程负责生成子进程，每个子进程响应多个请求；

	I/O模型：
				阻塞型、非阻塞型、复用型、信号驱动型、异步；
		阻塞：是在wait数据和copy数据的都阻塞
		非阻塞：在wait data 时没有阻塞并check data copy的状态，copy段被阻塞
		复用性：在wait data 时被阻塞并提供select（）显示，copy 段被阻塞
		signal-driven ：在 wait datat 不阻塞 并提供epoll调用，copy data 被 blocked ；注：在内核copy数据是很快的
		as 异步i/o:两阶段都不堵塞，通过 singal/hander机制，并在copy data 后 发送通知 aio

		同步/异步：
			关注的是消息通知机制；被调用者通知调用者
				注：被调用者发出的消息ws

			消息通知：
				同步：等待对方返回消息；
				异步：被调用者通过状态、通知或回调通知调用者被调用的运行状态；

		阻塞/非阻塞：
			关注调用者在等待结果返回之前所处的状态；

			阻塞：blocking，调用结果返回之前，调用者会被挂起；
			非阻塞：nonblocking，调用结果返回之前，调用不会被挂起；


		一次IO请求，都会由两个阶段组成：
			第一步：等待数据，即数据从磁盘到内核内存；
			第二步：复制数据，即数据从内核内存到进程内存；


			SELECT()：1024
			POLL()：

			c10k问题：主要就是指高并发多带来的问题
				Nginx就是很好解决此问题的行之有效的方法。
			事件驱动模型实现：
				Linux: epoll, libevent程序包；
				FreeBSE: kqueue
				Solaris： /dev/poll
	Nginx：
		http://nginx.org/x

		NGINX is a free, open-source, high-performance HTTP server and reverse proxy, as well as an IMAP/POP3 proxy server. NGINX is known for its high performance, stability, rich feature set, simple configuration, and low resource consumption.

		NGINX is one of a handful of servers written to address the C10K problem. Unlike traditional servers, NGINX doesn’t rely on threads to handle requests. Instead it uses a much more scalable event-driven (asynchronous) architecture. This architecture uses small, but more importantly, predictable amounts of memory under load. Even if you don’t expect to handle thousands of simultaneous requests, you can still benefit from NGINX’s high-performance and small memory footprint. NGINX scales in all directions: from the smallest VPS all the way up to large clusters of servers.

		NGINX powers several high-visibility sites, such as Netflix, Hulu, Pinterest, CloudFlare, Airbnb, WordPress.com, GitHub, SoundCloud, Zynga, Eventbrite, Zappos, Media Temple, Heroku, RightScale, Engine Yard, MaxCDN and many others.

		C10k：http://www.kegel.com/c10k.html

		nginx：tegine, openresty；

		Nginx的特性：
			nginx [engine x] is an HTTP and reverse proxy server, a mail proxy server, and a generic TCP/UDP proxy server, originally written by Igor Sysoev.
				http服务器
				http/imap/pop协议的反向代理服务器
				通用的TCP/UDP反代服务器（负载均衡）

			详情请见：http://nginx.org/en/

	Nginx程序架构：
		master/worker
			一个master进程：
				负责加载配置文件、管理worker进程、平滑升级等 ；
			一个或多个worker进程：
				处理并响应用户请求；
			缓存进程：
				cache loader
				cache manager

		nginx的功用：
			静态的web资源服务器；
			结合FastCGI/uwsgi/SCGI等协议反代动态资源请求；（lnmt, lnmp）
			http/https协议的反向代理（负载均衡）
			smtp/imap/pop协议的反代；
			tcp/udp协议的反代；

	Nginx的模块类型：
		核心模块：core module
		标准模块：
			Standard HTTP modules
			Optional HTTP modules
			Mail modules
			Stream modules
		3rd party modules

	nginx安装配置：
		官方的预制包：
			http://nginx.org/packages/centos/7/x86_64/
		编译安装需要的包组
	#yum install pcre-devel zlib-level openssl-devel
		编译安装：
			# yum install pcre-devel zlib-devel openssl-devel
			# ./configure --prefix=/usr/local/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_dav_module  --with-http_stub_status_module --with-threads --with-file-aio
			# make -j # && make install

		配置：
			配置的组成部分：
				主配置文件：nginx.conf
					include conf.d/*.conf
				fastcgi、uwsgi、SCGI等相关的配置文件
				mime.types：支持的mime类型

			主配置文件的配置指令：
				directive value1 [value2 ...];

				注意：
					(1) 指令必须以分号结尾；
					(2) 支持使用配置变量 ：
						由模块引入：内建变量；
						由用户引用：自定义变量；
							set variable_name value;
							引用变量：$variable_name

			主配置文件结构：
				main block：主配置段；全局配置段；
				event {
					...
				} 事件驱动相关的配置；
				http {
					...
				} http/https等 相关的配置段；
				mail {
					...
				}

				http配置的结构：
					http {
						...
						...
						server {
							...
							server_name
							root
							alias
							location {
								...
							}
							...
						}
						server {
							...
						}
					}

			main block常见的配置指令：
				分类：
					正常运行必备的配置；
					优化性能相关的配置；
					用于调试及定位问题相关的配置；

				正常运行必备的配置:     http://nginx.org/en/docs/dirindex.html  可使用的directives得配置方法
					1、user USERNAME [GROUPNAME];
						指定用于运行worker进程的用户和组；
						user nginx nginx
					2、pid  /PATH/TO/PID_FILE;
						指定nginx进程的piduse件路径；
						需要手动创建目录：/usr/local/nginx/logs/并且进程ID为master进程ID
						pid        logs/nginx.pid;
					3、worker_rlimit_nofile number ; 1024 太少
						单个worker进程所能够打开 的最大文件数量；

				性能优化相关的配置：
					1、worker_processes number|auto；2
						worker进程的数量；通常应该为cpu核心数量；

					2、worker_cpu_affinity cpumask ...;
						worker_cpu_affinity auto[cpumask];

						CPUMASK:
						worker_cpu_affinity 0001 0010 0100 1000;

					3、worker_priority nice;
						[-20,19]

				调试、定位问题：
					1、daemon on|off;
						是否以守护进程方式运行nginx进程；

					2、mastet_process on|off;
						是否以master/worker模型启动nignx进程；

					3、error_log file [level];
						错误日志文件位置及其级别；

					4、thread_pool name threads=number [max_queue=number];
						线程池中的线程数量，及后援队列的长度；


	回顾：IO模型、nginx

	IO模型：
		阻塞式
		非阻塞式
		IO复用（select, poll）
		信号驱动的IO（epoll, kqueue, /dev/poll）
		AIO

		阶段：等待数据准备完成，复制数据（从内核缓冲区到进程的地址空间）

	nginx：master/worker
		master
		worker（）
		cache loader
		cache manager

		模块类别：核心模块、标准模块（http标准模块，http可选模块，mail模块）、3rd模块

	Nginx(2)

	nginx.conf：
		main block
			Syntax:	thread_pool name threads=number [max_queue=number];
			thread_pool default threads=32 max_queue=65536;
		events {
			...
		}

			1、worker_connections number;
				每个worker进程所能够并发打开的最大连接数；

				worker_processes * worker_connections

			2、use method;
				指明并发连接请求处理时使用的方法；

				use  epoll;
				  epoll接口在磁盘IO的特殊化
			3、accept_mutex on | off;
				启用时，表示用于让多个worker轮流地、序列化地响应新请求；

		http {
			...
		}

		定义套接字相关功能

			1、server { ... }
				配置一个虚拟主机；

				server {
					listen PORT;
					server_name  HOSTNAME;
					root /PATH/TO/DOCUMENTROOT;
					...
				}

				注意：
					(1) 基于port的虚拟主机：
						listen指令要使用不同的端口；
					(2) 基于Hostname的虚拟主机；
						server_name指令指向不同的主机名；
					(3) 基于ip的虚拟主机：
						listen IP:PORT;

			2、listen address[:port] [default_server] [ssl] [backlog=number] [rcvbuf=size] [sndbuf=size];
			      listen port [default_server] [ssl];
			      listen unix:path [default_server] [ssl] ;

				default_server：默认虚拟主机；
				ssl：限制只能通过ssl连接提供服务；
				backlog：后缓队列的长度；
				rcvbuf：接收缓冲大小；
				sndbuf：发送缓冲区大小；

			3、server_name name ...;
				指明当前server的主机名；后可跟一个或空白字符分隔的多个主机；
				支持使用*任意长度的任意字符；
				支持~起始的正则表达式模式字符串；

				应用策略：
					(1) 首先精确匹配；
					(2) 左则*通配符匹配；
					(3) 右侧*通配符匹配；
					(4) 正则表达式模式匹配；

					server_name  www.magedu.com;

					server_name *.magedu.com;

					server_name  www.magedu.*;

					server_name ~^.*\.magedu\..*$;

					mail.magedu.com, www.magedu.com

			4、tcp_nodelay  on|off;
				对keepalived模式下的连接是否启用TCP_NODELAY选项；
				tcp_nodelay on 禁用nagle算法，nagle算法的好处时，当不需要及时发送数据（ftp）时可以开启nagle算法 积攒到一点字节的数据或到一定时间发送包，这样可以提高效率，节省资源

			5、sendfile on | off;
				是否启用sendfile功能；当启动此选项时，内核将请求的数据直接封装返回请求，而不需要发送到请求的进程中的内存


	定义路径相关配置
			6、root path;
				设置web资源路径映射；用于指明用户请求的url所对应的本地文件系统上的文档所在目录路径；
				可用上下文：http, server, location, if

			7、location [ = | ~ | ~* | ^~ ] uri { ... }
			      location @name { ... }

			      根据用户请求的URI来匹配定义的location，匹配到时，此请求将被相应的location块中的指令所处理；

				server {
					...
					location /images {
						aio on;
					}
					location {
						...
					}
				}

				=：URI精确匹配；
				~：做正则表达式模式匹配，区分字符大小写；
				~*：做正则表达式模式匹配，不区分字符大小写；
				^~：对URI的左半部分做匹配检查，不区分字符大小写；

				匹配优先级：=、^~、~、～*、不带符号；
					相同级别匹配最长的location
				http {
					thread_pool one threads=128 max_queue=65535;
					thread_pool two threads=32;
					server {
						location /one {
							aio threads=one;
						}
						location /two {
							aio threads=two;
						}
					}
				…
				}

			8、alias path;
				定义路径别名，文档映射的一种机制；仅能用于location上下文；

				alias  /bbs/  /web/forum/

				http://www.magedu.com/bbs/a.jpg

					location  /bbs/  {
						alias  /web/forum/;
					}

					/web/forum/a.jpg

					location  /bbs/  {
						root  /web/forum/;
					}

					/web/forum/bbs/a.jpg


				注意：
					root指令：给定的路径对应于location中的/uri/左侧的/;
					alias指令：给定的路径对应于location中的/uri/右侧的/；

			9、index file ...;
				可用位置：http, server, location

				默认主面；

			10、error_page code ... [=[response]] uri;
				根据用户请求的资源的http响应的状态码实现错误页重定向；

				http://www.magedu.com/hello.html --> 因为资源不存在而被改为对
					http://www.magedu.com/404.html

			11、



	定义客户端请求的相关配置

			12、keepalive_timeout timeout [header_timeout];
				设定保持连接的超时时长，0表示禁止长连接 ；默认为75s;

			13、keepalive_requests number;
				在一次长连接上所允许请求的资源的最大数量，默认为100；

			14、keepalive_disable none | browser ...;
				对哪种浏览器禁用长连接；

			15、send_timeout time;
				向客户端发送响应报文的超时时长； 特别地，是指两次写操作之间的间隔时长；

			16、client_body_buffer_size size;
				用于接收客户端请求报文的body部分的缓冲区大小；默认为16k；超时此大小时，其将被暂存到磁盘上；

			17、client_body_temp_path path [level1 [level2 [level3]]];
				设定用于存储客户端请求报文的body部分的临时存储路径及子目录结构和数量；

				/var/tmp/body  2 1 2
					2位16进制（2个4位二进制）创建一级子目录	1在每个一级子目录下再创建1位个16进制数
				00-ff


	对客户的请求进行限制的相关配置：
			18、limit_rate rate;
				限制响应给客户端的传输速率，单位是bytes/second，0表示无限制；
				server {
				    if ($slow) {
				        set $limit_rate 4k;
				    }
				    ...
				}

			19、limit_except method ... { ... };
				限制对指定的请求方法之外的其它方法的使用客户端；

				limit_except GET POST {
					allow  172.18.0.0/16;
					deny all;
				}

				表示除了GET和POST之外的其它方法仅允许172.18.0.0/16中的主机使用；

	文件操作优化的配置：
			20、aio on | off | threads[=pool];
				是否启用aio功能；

				很多人会将AIO理解成磁盘IO的异步方案，会将AIO狭隘化为类epoll接口在磁盘IO的特殊化，其实AIO应该是横架于整个内核的接口，它把所有的IO包括(本地设备，网络，管道等)以统一的异步接口提供给用户程序，每个子系统都针对接口实现自己的异步方案，而同步IO(Synchronous IO)只是在内核内部的”AIO+Blocking”.
				location /video/ {
				    aio            on;
				    output_buffers 1 64k;
				}

			21、directio size | off;
					增加安全，可能影响效率
			22、open_file_cache off;
				open_file_cache max=N [inactive=time];
					nginx可以缓存以下三种信息：
						(1) 文件的描述符、文件大小和最近一次的修改时间；
						(2) 打开的目录的结构；
						(3) 没有找到的或者没有权限访问的文件的相关信息；

					max=N：可缓存的缓存项上限；达到上限后会使用LRU算法实现缓存管理；

					inactive=time：缓存项的超时时长，在此处指定的时长内未被命中的缓存项即为非活动项；
				当缓存数超过max使用LRU算法进行删除缓存
				LRU是Least Recently Used 最近最少使用算法清除那些最近最少使用的文件
					open_file_cache  max=1000 inactive=20s;
					open_file_cache_valid    30s;
					open_file_cache_min_uses 2;
					open_file_cache_errors   on;
			23、open_file_cache_errors on | off;
				是否缓存查找时发生错误的文件一类的信息；

			24、open_file_cache_min_uses number;
				在open_file_cache指令的inactive参数指定的时长内，至少命中此处指定的次数方可不被归类到非活动项；

			25、open_file_cache_valid time;
				缓存项有效性的检查频率；默认是60s；

	ngx_http_access_module模块：
			实现基于ip的访问控制功能；

			26、allow address | CIDR | unix: | all;
			27、deny address | CIDR | unix: | all;

			可用上下文：http, server, location, limit_except

	ngx_http_auth_basic_module模块：
			28、auth_basic string | off;
				使用basic机制进行用户认证；

			29、auth_basic_user_file file;
				认证用的账号密码文件；

				文件格式：
					name:password:commet

				密码格式：
					htpasswd命令；

					location /admin/ {
					auth_basic "Admin Area";
					auth_basic_user_file /etc/nginx/.ngxpasswd;
					}

		ngx_http_stub_status_module模块：
			用于输出nginx的基本状态信息；
			location /status {
	            stub_status;
	   		 }
				注：status后面不可以加/

			Active connections: 1
			server accepts handled requests
			155 155 298
			Reading: 0 Writing: 1 Waiting: 0

			Active connections:  处于活动状态的客户端连接的数量；
			已经处理完成的
			accepts：已经接受的客户端请求的总数；
			handled：已经处理完成的客户端请求的总数；
			requests：客户端发来的总的请求数；有的可能没有响应
			正在请求的
			Reading：处于读取客户端请求报文首部的连接数；location /status {
	            stub_status;
	    }

			Writing：处于向客户端发送响应报文过程中的连接数；
			Waiting：处于等待客户端发出请求的空闲连接数；

		ngx_http_referer_module模块：
			The ngx_http_referer_module module is used to block access to a site for requests with invalid values in the “Referer” header field.

			30、valid_referers none | blocked | server_names | string ...;
				定义合法的referer数据；

				none：请求报文首部没有referer首部；
				blocked：请求报文的referer首部没有值；
				server_names：其值是主机名；
				arbitrary string：直接字符串，可以使用*作为通配符；
				regular expression：被指定的正则表达式模式匹配到的字符串；要使用~起始；

				valid_referers none blocked  server_names *.magedu.com magedu.* ~\.magedu\.;

				if ($invalid_referer) {
					return 403;
				}


			线程池：可以处理以前有master处理的进程
			wrk 压测工具
			https://github.com/wg/wrk
			loadrunner 专业一点 续航能力



	#ssl 基于IP地址

ngx_http_log_module
	访问日志
log_format name string ...; nginx 内建模块 $varname 引用变量

x_forward

rewrite :
ngx_http_rewrite_module    pcre

Nginx(3)

	ngx_http_ssl_module
	ssl		Secure Sockets Layer 安全套接层
			Transport Layer Security  传输层安全

工作在tcp 和app层之间的半层 openssl: libssl.so libecrypto.so,oopenssl
			是否启用当前虚拟主机的ssl功能；

		ssl_certificate file;
			当前虚拟主机使用PEM格式的证书文件；

		ssl_certificate_key file;
			当前虚拟主机上与其证书匹配的私钥文件；

		ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2];


		ssl_session_cache off | none | [builtin[:size]] [shared:name:size];
			builtin:
				a cache built in OpenSSL; used by one worker process only.
				builtin 不建议使用，容易产生内存碎片
			shared：
				a cache shared between all worker processes.
				1M缓存官方提供信息为4000个会话
				缓存 共享 shared 建议使用：shared:SSL:1m；
		ssl_session_timeout time;
			客户端连接可以复用ssl sessin cache中缓存的ssl参数的有效时长；缓存有效时长默认5分钟 ；
		ssl_ciphers  HIGH:!aNULL:!MD5;注：！取反 +身份
		ssl_prefer_server_ciphers  on; 有服务端选用加密算法

			Syntax:	ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2];
			Default:
			ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
			Context:	http, server
			Syntax:	ssl_ciphers ciphers;
				Default:
				ssl_ciphers HIGH:!aNULL:!MD5;
				Context:	http, server
				Specifies the enabled ciphers. The ciphers are specified in the format understood by the OpenSSL library, for example:

				ssl_ciphers ALL:!aNULL:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
				The full list can be viewed using the “openssl ciphers” command.
	ngx_http_log_module

			The ngx_http_log_module module writes request logs in the specified format.

		log_format name string ...;
			string可使用nginx核心及各模块内嵌的变量的值；
				 1.$remote_addr 与$http_x_forwarded_for 用以记录客户端的ip地址；
				 2.$remote_user ：用来记录客户端用户名称；
				 3.$time_local ： 用来记录访问时间与时区；
				 4.$request ： 用来记录请求的url与http协议；
				 5.$status ： 用来记录请求状态；成功是200，
				 6.$body_bytes_s ent ：记录发送给客户端文件主体内容大小；
				 7.$http_referer ：用来记录从那个页面链接访问过来的；
				 8.$http_user_agent ：记录客户端浏览器的相关信息；
				 9.$http_x_forwarded_for：请求报文中x_forwarded_for的值 由哪个主机反代服务器的值 真正的客户端地址
			作业：为nginx定义使用combined格式的日志；
			  httpd格式：LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined

			access_log off;
			access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
				路径     格式    缓存 缓存空间 在缓冲区留多少空间缓存日志（空间到服务器崩溃时丢失的信息量大） 刷新时间（每隔多长时间把buffer中的日志 写到硬盘path中）[if=condition]]什么条件做什么操作
		nginx内置变量
								内置变量存放在  ngx_http_core_module 模块中，变量的命名方式和apache 服务器变量是一致的。总而言之，这些变量代表着客户端请求头的内容，例如$http_user_agent, $http_cookie, 等等。下面是nginx支持的所有内置变量：

								$arg_name

								请求中的的参数名，即“?”后面的arg_name=arg_value形式的arg_name

								$args

								请求中的参数值

								$binary_remote_addr

								客户端地址的二进制形式, 固定长度为4个字节

								$body_bytes_sent

								传输给客户端的字节数，响应头不计算在内；这个变量和Apache的mod_log_config模块中的“%B”参数保持兼容

								$bytes_sent

								传输给客户端的字节数 (1.3.8, 1.2.5)

								$connection

								TCP连接的序列号 (1.3.8, 1.2.5)

								$connection_requests

								TCP连接当前的请求数量 (1.3.8, 1.2.5)

								$content_length

								“Content-Length” 请求头字段

								$content_type

								“Content-Type” 请求头字段

								$cookie_name

								cookie名称

								$document_root

								当前请求的文档根目录或别名

								$document_uri

								同 $uri

								$host

								优先级如下：HTTP请求行的主机名>”HOST”请求头字段>符合请求的服务器名

								$hostname

								主机名

								$http_name

								匹配任意请求头字段； 变量名中的后半部分“name”可以替换成任意请求头字段，如在配置文件中需要获取http请求头：“Accept-Language”，那么将“－”替换为下划线，大写字母替换为小写，形如：$http_accept_language即可。

								$https

								如果开启了SSL安全模式，值为“on”，否则为空字符串。

								$is_args

								如果请求中有参数，值为“?”，否则为空字符串。

								$limit_rate

								用于设置响应的速度限制，详见 limit_rate。

								$msec

								当前的Unix时间戳 (1.3.9, 1.2.6)

								$nginx_version

								nginx版本

								$pid

								工作进程的PID

								$pipe

								如果请求来自管道通信，值为“p”，否则为“.” (1.3.12, 1.2.7)

								$proxy_protocol_addr

								获取代理访问服务器的客户端地址，如果是直接访问，该值为空字符串。(1.5.12)

								$query_string

								同 $args

								$realpath_root

								当前请求的文档根目录或别名的真实路径，会将所有符号连接转换为真实路径。

								$remote_addr

								客户端地址

								$remote_port

								客户端端口

								$remote_user

								用于HTTP基础认证服务的用户名

								$request

								代表客户端的请求地址

								$request_body

								客户端的请求主体

								此变量可在location中使用，将请求主体通过proxy_pass, fastcgi_pass, uwsgi_pass, 和 scgi_pass传递给下一级的代理服务器。

								$request_body_file

								将客户端请求主体保存在临时文件中。文件处理结束后，此文件需删除。如果需要之一开启此功能，需要设置client_body_in_file_only。如果将次文件传递给后端的代理服务器，需要禁用request body，即设置proxy_pass_request_body off，fastcgi_pass_request_body off, uwsgi_pass_request_body off, or scgi_pass_request_body off 。

								$request_completion

								如果请求成功，值为”OK”，如果请求未完成或者请求不是一个范围请求的最后一部分，则为空。

								$request_filename

								当前连接请求的文件路径，由root或alias指令与URI请求生成。

								$request_length

								请求的长度 (包括请求的地址, http请求头和请求主体) (1.3.12, 1.2.7)

								$request_method

								HTTP请求方法，通常为“GET”或“POST”

								$request_time

								处理客户端请求使用的时间 (1.3.9, 1.2.6); 从读取客户端的第一个字节开始计时。

								$request_uri

								这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI，不包含主机名，例如：”/cnphp/test.php?arg=freemouse”。

								$scheme

								请求使用的Web协议, “http” 或 “https”

								$sent_http_name

								可以设置任意http响应头字段； 变量名中的后半部分“name”可以替换成任意响应头字段，如需要设置响应头Content-length，那么将“－”替换为下划线，大写字母替换为小写，形如：$sent_http_content_length 4096即可。

								$server_addr

								服务器端地址，需要注意的是：为了避免访问linux系统内核，应将ip地址提前设置在配置文件中。

								$server_name
								服务器名，www.cnphp.info

								$server_port
								服务器端口

								$server_protocol
								服务器的HTTP版本, 通常为 “HTTP/1.0” 或 “HTTP/1.1”

								$status

								HTTP响应代码 (1.3.2, 1.2.2)

								$tcpinfo_rtt, $tcpinfo_rttvar, $tcpinfo_snd_cwnd, $tcpinfo_rcv_space
								客户端TCP连接的具体信息

								$time_iso8601
								服务器时间的ISO 8610格式 (1.3.12, 1.2.7)

								$time_local
								服务器时间（LOG Format 格式） (1.3.12, 1.2.7)

								$uri

								请求中的当前URI(不带请求参数，参数位于$args)，可以不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改，$uri不包含主机名，如”/foo/bar.html”。




		open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time];
		open_log_file_cache off;
			日志文件的文件描述符缓存机制；

			max：最大可缓存的文件描述符数量；
			inactive：非活动时长；
			min_users：最少使用次数；
			valid：验正缓存条件有效性的时间周期；

	ngx_http_rewrite_module

		The ngx_http_rewrite_module module is used to change request URI using PCRE regular expressions, return redirects, and conditionally select configurations.

		rewrite regex replacement [flag];
			将用户请求的URI基于regex所描述的模式进行检查，匹配到时将其替换为replacement指定的URI；
			在同一级配置块中存在的多个rewrite规则，会自上而下逐个检查；隐含有循环机制；可以使用flag来控制循环方式；
			如果replacement是以http://或https://开头，则替换结果会直接以重定向返回给客户端：
				301：永久重定向；
			        rewrite ^/images/(.*\..*)$ /picture/$1 break;

			[flag]
				last：重定完成后停止对当前URI在当前location中后续的其它重写操作后复启新一轮替换检查；重启循环；
				break：重写完成后，即刻对当前location中的rewrite规则的检查，转而执行后续的其它的配置；结束循环；
				redirect：重写完成后以临时重定向方式直接返回重写后生成的新URI给客户端，由客户端重新请求； 不能以http://或https://开头；
				permanent：重写完成后以永久重定向的方式直接返回重写后生成的新URI给客户端，由客户重新请求； 以http://或https://开头；

		rewrite_log on | off;
			适配于notice级别并记录重写操作于错误日志中；

		return

			Stops processing and returns the specified code to a client.
		注：实例 当查看的页面不存在时，转到主页
				if (!-e $request_filename) {
				    rewrite ^/.* /index.php last;
				}

		if (condition) { ... }
			引入一个新的上下文；条件满足时，执行配置块中的配置指令；

			condition：
				比较操作符：
					==
					!=
					~：模式匹配，区分字符大小写；
					~*：模式匹配，不区分字符大小写；
					!~：
					!~*：
				文件及目录存在性判断：
					-e, !-e：存在与否；
					-f, !-f：存在且为一个普通文件与否；
					-d, !-d：
					-x, !-x：


		set $variable value;
			用户自定义变量

	ngx_http_gzip_module

			gzip on | off;

			gzip_buffers number size;
				用于实现压缩功能的缓存区数量及大小；x86 32 4k other 16 8k

			gzip_comp_level level;1

			gzip_disable regex ...;msie6
				对客户端浏览器类型字符串匹配至此处的regex所描述的模式的请求禁用压缩功能；

			gzip_min_length length;20

			gzip_proxied off | expired | no-cache | no-store | private | no_last_modified | no_etag | auth | any ...;
				对代理的请求所获取的响应报文是否启动压缩功能，以及如何启用；

			gzip_types mime-type ...; text/plain text/css text/xml application/json application/xml application/x-javascript;
				压缩过滤器，仅对此处设定的类型的内容启用压缩功能；

	conn-limit
	ngx_http_fastcgi_module

		lnmp：
			nginx+php：fastcgi
				nginx: fastcgi client
				fpm-php: fastcgi server

		The ngx_http_fastcgi_module module allows passing requests to a FastCGI server.

		常用指令：
			fastcgi_pass address;
				address为php-fpm服务器监听的地址；

			fastcgi_index name;
				fastcgi默认的主页资源；

			fastcgi_param  parameter value [if_not_empty];
				Sets a parameter that should be passed to the FastCGI server.
				fastcgi_param  SCRIPT_FILENAME  /data/vhost/amp2/phpMyAdmin$fastcgi_script_name;
								传递给fastcgi	补上路径（/scripts） +uri中的	 fastcgi_script_name URL中最后一段脚本名称不包含路径
			注：fastcgi_param  SCRIPT_FILENAME  /data/vhost/amp2/phpMyAdmin$fastcgi_script_name;


			fastcgi_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
				注意：http的上下文
				启用缓存后的工作流程：先检查nginx端有没有请求的缓存如果没有再向fastcgi（php-fpm）请求
				一般缓存命中率 30% 已经不错 缓存命中率有两种 字节命中率和页面命中率 衡量标准：节约的时间和命中的次数
					绝大部分缓存键值存储 键key（内存中） 请求的URL 把请的结果当值(磁盘中)
					key 为URL的hash码 对应URL内容（文件系统）的hash码 如果是文件名系统则为文件名的hash码
						存储在磁盘中hash码当文件名，hash为16进制数字  128/4 32 位数字 最左侧（有可能最右侧）的第两个数字切出来（最右侧不切）当一级目录名 一次后面的切出来当二级目录名 及三级   256 256 256
				path：缓存目录路径；
				levels=：缓存目录层级数量，以及每一级的目录数量；
					levels=1:2:1
							1位16进制：2位16进制：1位16进制
				keys_zone=name:size
					内存中的缓存空间的名称及大小；fpmcache：32
				[max_size=size]：磁盘上用于缓存数据的缓存空间上限；
				[inactive=time]：缓存时长；

			fastcgi_cache zone | off;
				调用指定的缓存空间来缓存数据；zone 缓存空间的名字fpmcache

			fastcgi_cache_key string;
				定义用作缓存项的key的字符串；fastcgi_cache_key localhost：9000$request_uri

			fastcgi_cache_methods GET | HEAD | POST ...;
				对哪些请求方法来检查缓存；

			fastcgi_cache_min_uses number;请求几次缓存内容

			fastcgi_cache_valid [code ...] time;
				对不同的响应码的资源可缓存时长；
					fastcgi_cache_valid 200 302 10m;
					fastcgi_cache_valid 404      1m;

		注意：调用缓存时，至少应该设定三个参数
			fastcgi_cache
			fastcgi_cache_key
			fastcgi_cache_valid

	博客作业：
		lnmp实现多个虚拟主机，部署wordpress和phpmyadmin，并为后一个主机提供http

Nginx(4)

	LB Cluster：
		传输层：lvs、nginx、haproxy
		应用层：nginx(http, https, smtp, pop, imap), haproxy(http), httpd(http/https), ats, perlbal, pound, ...

	nginx load balancer：
		tcp/udp

	nginx proxy：
		reverse proxy：

	应用程序发布：
		灰度模型：
			(1) 如果存在用户会话；
				从服务器上拆除会话；
			(2) 新版本应用程序存在bug；
				回滚；

	ngx_http_proxy_module
			`location / {
		    proxy_pass       http://localhost:8000;
		    proxy_set_header Host      $host;
		    proxy_set_header X-Real-IP $remote_addr;把远程主机的地址（客户端）设置成代理主机请求的首部（此项默认）
			}`
		(1) proxy_pass URL;
			location, if in location, limit_except

			注意：proxy_pass后面的路径不带uri时，其会将location的uri传递给后端主机；

				location /uri/ {
					proxy_pass http://HOST;
				}

			proxy_pass后面的路径是一个uri时，其会将location的uri替换为proxy_pass的uri；
				location /uri/ {
					proxy_pass http://HOST/new_uri/;
				}

			如果location定义其uri时使用正则表达式的模式，则proxy_pass之后必须不能使用uri；
				location ~|~* PATTERN {
					proxy_pass http://HOST;
				}

		(2) proxy_set_header field value;代理服务器可以修改或附加一个新值传递给被代理的服务器
			设定发往后端主机的请求报文的请求首部的值；

			示例：http server location
				proxy_set_header X-Real-IP $remote_addr;
				proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for ()
				proxy_set_header Host       $proxy_host;proxy_host代理服务器的IP地址
				proxy_set_header Connection close;	无论请求为长连接还是短连接都设置成短连接
					在代理后端的主机更改日志格式 LogFormat 添加 %{X-Real-IP}i 的值
		(3) proxy_cache_path

			proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];

		(4) proxy_cache zone | off;
			调用的缓存的名称，或禁用缓存；

		(5) proxy_cache_key string;
			缓存条目的键；

		(6) proxy_cache_valid [code ...] time;
			对各类响应码的缓存时长；

		使用示例：
			定义在http{}中：
				proxy_cache_path /var/cache/nginx/proxy_cache levels=1:2:1 keys_zone=pcache:10m max_size=1g;

			定义在server{}及其内部的组件中：
				proxy_cache pcache;
				proxy_cache_key $request_uri;
				proxy_cache_valid 200 302 10m;
				proxy_cache_valid 301 1h;
				proxy_cache_valid any 1m;

		(7) proxy_cache_use_stale error | timeout | invalid_header | updating | http_500 | http_502 | http_503 | http_504 | http_403 | http_404 | off ...;
				在哪种情况下 给用户响应过期缓存
		(8) 	proxy_connect_timeout 连接超时（反代主机向后端应用服务器连接的超时时长）
			proxy_read_timeout  用户请求响应超时时长（默认60秒；可以改长一些，减少用出现服务器连接超时）
			proxy_send_timeout  发送超时

		(9)proxy_buffer_size size
				proxy_buffering on | off
				proxy_buffers number size
		10）Syntax:	proxy_redirect default;
				proxy_redirect off;
				proxy_redirect redirect replacement;
				Default:
				proxy_redirect default;
				Context:	http, server, location
			proxy_redirect http://localhost:8000/ http://$host:$server_port/;
	ngx_http_headers_module
		The ngx_http_headers_module module allows adding the “Expires” and “Cache-Control” header fields, and arbitrary fields, to a response header.

		(1) 	add_header name value [always];
			向响应报文中添加自定义首部；
			http, server, location, if in location

			可用上下文：http, server, location, if in location

			add_header X-Via $server_addr;
			add_header X-Accel $server_name;

		(2) expires [modified] time;
			expires epoch | max | off;

			用于定义Expire或Cache-Control首部的值，或添加其它自定义首部；

回顾：
	nginx：
		web server
		http/https reverse proxy
		tcp/udp upstream server

	ngx_http_proxy_module
		proxy_pass
		proxy_set_header

		proxy_cache_path

		proxy_cache
		proxy_cache_key
		proxy_cache_valid

		proxy_connect_timeout, proxy_read_timeout, proxy_send_timeout

	ngx_http_headers_module
		add_header

Nginx(5)

	ngx_http_upstream_module
		The ngx_http_upstream_module module is used to define groups of servers that can be referenced by the proxy_pass, fajzfstcgi_pass, uwsgi_pass, scgi_pass, and memcached_pass directives.


				upstream backend {
				    server backend1.example.com       weight=5;
				    server backend2.example.com:8080;
				    server unix:/tmp/backend3;

				    server backup1.example.com:8080   backup;
				    server backup2.example.com:8080   backup;
				}

				server {
				    location / {
				        proxy_pass http://backend;
				    }
				}
		(1) upstream name { ... }
			定义后端服务器组；引入一个新的上下文；只能用于http{}上下文中；

		(2) server address [parameters];
			定义服务器地址和相关的参数；
				地址格式：
					IP[:PORT]
					HOSTNAME[:PORT]  FQDN
					unix:/PATH/TO/SOME_SOCK_FILE

				参数：
					weight=number
						权重，默认为1；
					max_fails=number
						失败尝试的最大次数；
					fail_timeout=time  健康检测
						设置服务器为不可用状态的超时时长；
					backup
						把服务器标记为“备用”状态；
					down
						手动标记其为不可用；
					server backend1.magedu.com weight=5 down backup; 下线服务器使用；
		(3) least_conn;upstream上下文
			最少连接调度算法； 当server拥有不同的权重时为wlc；

		(4) least_time header | last_byte;
			最短平均响应时长和最少连接；
			header：response_header;
			last_byte: full_response;

			仅Nginx Plus有效；

		(5) ip_hash;
			源地址hash算法；能够将来自同一个源IP地址的请求始终发往同一个upstream server；
			取模发调度 根据IP地址hash出来的值 对 upstream中的服务器总权重取模 余几就放在那台主机 过后每次访问时 就会同样访问此主机 一旦某个主机down后 整个就会遭到破坏
		(6) hash key [consistent]; 减少 IP hash 带来的破坏； 构建hash换 平均的将（0-2^32）
			基于指定的key的hash表实现请求调度，此处的key可以文本、变量或二者的组合；
			注：增删服务器时影响的服务器是有限的 优于ip_hash；缓存服务器 使用一致hash算法；
			consistent：参数，指定使用一致性hash算法；

			示例：
				hash $request_uri consistent  uri
				hash $remote_addr	原地址
				hash $cookie_name	cookie

		(7) keepalive connections;
			可使用长连接的连接数量；
			keepalive 32;
		(8) health_check [parameters]; 一般可以基于：网络 IP 传输层：端口 应用层 资源的请求
			定义对后端主机的健康状态检测机制；只能用于location上下文；

			可用参数：
				interval=time：检测频率，默认为每隔5秒钟；
				fails=number：判断服务器状态转为失败需要检测的次数；
				passes=number：判断服务器状态转为成功需要检测的次数；
				uri=uri：判断其健康与否时使用的uri；
				match=name：基于指定的match来衡量检测结果的成败；
				port=number：使用独立的端口进行检测；

			仅Nginx Plus有效；

		(9) match name { ... }
			Defines the named test set used to verify responses to health check requests.
			定义衡量某检测结果是否为成功的衡量机制；

			专用指令：
				status：期望的响应码；
					status CODE
					status ! CODE
					...
				header：基于响应报文的首部进行判断
					header HEADER=VALUE
					header HEADER ~ VALUE
					...
				body：基于响应报文的内容进行判断
					body ~ "PATTERN"
					body !~ "PATTERN"
				location / {
						    proxy_pass http://backend;
						    health_check;
						}
			仅Nginx Plus有效；
	Module ngx_http_memcached_module
				server {
				    location / {
				        set            $memcached_key "$uri?$args";
				        memcached_pass host:11211;
				        error_page     404 502 504 = @fallback;
				    }

				    location @fallback {
				        proxy_pass     http://backend;
				    }
				}
	博客作业：以上所有内容；
	课外实践：实践tengine；

	ngx_stream_core_module

		(1) listen address:port [ssl] [udp] [backlog=number] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
			监听的端口；
				默认为tcp协议；
				udp: 监听udp协议的端口；


		示例：此模块不支持backup
			stream {
				upstream sshsrvs {
					server 192.168.10.130:22;
					server 192.168.10.131:22;
					hash $remote_addr consistent;
				}

				server {
					listen 172.16.100.6:22202;
					proxy_pass sshsrvs;
				}
			}


				Syntax:	load_module file;
				Default:	—
				Context:	main
				This directive appeared in version 1.9.11.
				Loads a dynamic module.

				Example:

				load_module modules/ngx_mail_module.so;
				memcached：
					注意：添加的模块  属主应该改成 nginx 切记：文件操作时一定要有权限
	缓存服务器：
		缓存：cache，无持久存储功能；
		bypass缓存
		k/v cache

	LiveJournal旗下的Danga Interactive研发；

		特性：
			k/v cache, 可序列化数据；存储项：key, value, flag, expire time; 单数据项1m;
			功能的实现一半依赖于memcache server，一半依赖memcache client；
			分布式缓存：互不通信的分布式集群；
				分布式系统请求路由方法：取模法、一致哈希算法；
			O(1)的执行效率；
			清理过期数据：
				缓存耗尽：LRU，最近最少使用；
				缓存项过期：惰性清理机制；

	安装配置：
		由CentOS base仓库直接提供；
			监听的端口：
				11211/tcp, 11211/udp；

			主程序：/usr/bin/memcached
			环境配置文件：/etc/sysconfig/memcached

		协议格式：
			文本格式
			二进制格式

		命令：
			统计类：stats, stats items, stats slabs, stats sizes
			存储类：set, add, replace, append, prepend
			获取数据类：get, delete, incr/decr
			清空：flush_all
			add mykey 0（flag） 300（ttl） 20（长度） 输入键值
				decr(减) mykey 1   incr（增加） mykey 1 flush_all 清空
		memcached程序的常用选项：
			-m <num>：缓存空间大小，单位为MB，默认为64；
			-c <num>：并发连接数，默认为1024；
			-u USERNAME：程序的运行者；
			-p PORT: 监听的TCP端口；
			-U PORT: 监听的UDP端口；
			-l <ip_addr>：监听的IP地址；
			-M： 缓存空间耗尽时，向请求存储缓存项返回错误信息，而非使用默认的LRU算法清理缓存；
			-f <factor> ：chunk size growth factor (default: 1.25) lsb 减少内存碎片
			-t <num>线程数，默认为4 小于等于cpu核心数
		memcached -u mecached -f 1.5 -vv
	memcached认证机制：
		默认没有认证机制;可借助SASL进行认证；
			SASL:简单认证安全层 Simple Authentication Secure Layer;
	API:
		libmemcached（C/C++）
		PHP：
			extensions
				memcached
		php 会话保存在memcached中 就成为session server
			php.ini
					session.save_handler = memcached
					session.save_path = " "
		memcached -d -m 900 -u root -l 192.168.100.186 -p 11211 -c 256 -P /tmp/memcached.pid //启动memcached 启动参数说明：
	注：php-fpm 更改配置文件在 /etc/php-fpm/www.conf
			;php_value[session.save_handler] = files
			php_value[session.save_handler] = memcache
			;php_value[session.save_path] = /var/lib/php/session
			php_value[session.save_path] =  "tcp://172.16.174.160:11211?persistent=1&weight=1&timeout=1&retry_interval=15"
		php 在/etc/httpd/conf.d/php.conf
			#php_value session.save_handler "files"
			#php_value session.save_path    "/var/lib/php/session"
			php_value session.save_handler  "memcache"
			#php_value[session.save_path] = /var/lib/php/session
			php_value session.save_path   "tcp://172.16.174.160:11211？persistent=1&weight=1&timeout=1&retry_interval=15"

#连接测试memcached
			前提：
			1、配置各php支持使用memcache；
			2、安装配置好memcached服务器，这里假设其地址为172.16.200.11，端口为11211；


			一、配置php将会话保存至memcached中

			编辑php.ini文件，确保如下两个参数的值分别如下所示：
			session.save_handler = memcache
			session.save_path = "tcp://172.16.200.11:11211?persistent=1&weight=1&timeout=1&retry_interval=15"

			二、测试

			新建php页面setsess.php，为客户端设置启用session：
			<?php
			session_start();
			if (!isset($_SESSION['www.MageEdu.com'])) {
			  $_SESSION['www.MageEdu.com'] = time();
			}
			print $_SESSION['www.MageEdu.com'];
			print "<br><br>";
			print "Session ID: " . session_id();
			?>

			新建php页面showsess.php，获取当前用户的会话ID：
			<?php
			session_start();
			$memcache_obj = new Memcache;
			$memcache_obj->connect('172.16.200.11', 11211);
			$mysess=session_id();
			var_dump($memcache_obj->get($mysess));
			$memcache_obj->close();
			?>

			<?php
			// Generating cookies must take place before any HTML.
			// Check for existing "SessionId" cookie
			$session = $HTTP_COOKIE_VARS["SessionId"];
			if ( $session == "" ) {
			// Generate time-based unique id.
			// Use user's IP address to make more unique.
			$session = uniqid ( getenv ( "REMOTE_ADDR" ) );
			// Send session id - expires when browser exits
			SetCookie ( "SessionId", $session );
			}
			?>
			<HTML>
			<HEAD><TITLE>Session Test</TITLE></HEAD>
			<BODY> <br> 16 Current session id: <?php echo $session ?>
			</BODY></HTML>
配置实例：nginx.conf stream 模块 需要 --with-stream
worker_processes auto;

error_log /var/log/nginx/error.log info;

events {
    worker_connections  1024;
}

stream {
    upstream backend {
        hash $remote_addr consistent;

        server backend1.example.com:12345 weight=5;
        server 127.0.0.1:12345            max_fails=3 fail_timeout=30s;
        server unix:/tmp/backend3;
    }

    upstream dns {
       server 192.168.0.1:53535;
       server dns.example.com:53;
    }

    server {
        listen 12345;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }

    server {
        listen 127.0.0.1:53 udp;
        proxy_responses 1;
        proxy_timeout 20s;
        proxy_pass dns;
    }

    server {
        listen [::1]:12345;
        proxy_pass unix:/tmp/stream.socket;
    }
}


nginx.conf 配置文件

worker_rlimit_nofile 1024;

worker_processes  1;
worker_cpu_affinity 0001;
worker_priority -5;
error_log  logs/error.log;
error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
        use epoll;
        accept_mutex off;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
        log_format access '$remote_addr - $remote_user [$time_local] "$request" $request_time'
                        '$status $body_bytes_sent "$http_referer"'
                        '"$http_user_agent" "$http_x_forwardde_for" ';
    access_log logs/access.log access buffer=10m flush=1m;

    #tcp_nopush     on;
        tcp_nodelay on;
    #keepalive_timeout  0;
    keepalive_timeout  65;
        keepalive_requests 200;
#disable keepalive liulanqi
keepalive_disable none;
send_timeout 200;
client_body_buffer_size 16k;
client_body_temp_path /var/tmp/body 2 1;

limit_rate 0;
    gzip  on;
gzip_types text/plain ;
gzip_buffers 32 4k;
gzip_comp_level 2;
gzip_disable msie6;
gzip_min_length 32;
gzip_proxied expired no_last_modified;

fastcgi_cache_path /var/nginx/cache levels=2:1:1 keys_zone=fpmcache:1m inactive=30 max_size=10m ;
    server {
        listen       80;
        server_name  nginx.magedu.com;
        root /data/vhost/webnginx;
        #charset koi8-r;

        access_log logs/nginx.access.log.gzip access buffer=10m gzip=1 flush=20s;
        #open_log_file_cache on;
        open_log_file_cache max=4000 inactive=60 min_uses=2 valid=100;
        rewrite ^/images/(.*\..*)$ /picture/$1 break;
        #rewrite ^/(.*)$  http://nginx.magedu.com/$1 permanent;
        location / {
            index  index.html index.htm;

#               limit_except GET POST {
#                       allow 172.16.174.160;
#                       deny all;
#               }
        }
        location /images {
                alias /data/web/;
                aio     on;
                output_buffers 1 64k;
  }
        location /mystatus{
                stub_status;
                allow 172.16.174.160;
                deny all;
        }
        error_page  404   =1119           /error/404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
    #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    server {
        listen       443 ssl;
        server_name  nginx.magedu.com;

        ssl_certificate      https.crt;
        ssl_certificate_key  https.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
fastcgi_cache fpmcache;
fastcgi_cache_key 172.16.174.177:9000$request_uri;
fastcgi_cache_methods GET  HEAD  POST;
fastcgi_cache_min_uses 2;
fastcgi_cache_valid 200 302 10m;
fastcgi_cache_valid 404 1m;
#       rewrite ^http://nginx.magedu.com(/.*)$  https://nginx.magedu.com/$1 ;
        location / {
            root   /data/vhost/webnginx/phpMyAdmin;
            index  index.php index.html index.htm;
        }
        location /images {
                aio on;
                alias /data/vhost/webnginx/picture;
        }
        location ~ \.php$ {
            root           /data/vhost/amp2/phpMyAdmin;
            fastcgi_pass   172.16.174.177:9000;
            fastcgi_index  index.php;
           fastcgi_param  SCRIPT_FILENAME  /data/vhost/amp2/phpMyAdmin$fastcgi_script_name;
            include        fastcgi_params;

    }

}
}



strean 在1.11.2后支持的变量

$binary_remote_addr
client address in a binary form, value’s length is always 4 bytes for IPv4 addresses or 16 bytes for IPv6 addresses
$bytes_sent
number of bytes sent to a client
$connection
connection serial number
$hostname
host name
$msec
current time in seconds with the milliseconds resolution
$nginx_version
nginx version
$pid
PID of the worker process
$remote_addr
client address
$remote_port
client port
$server_addr
an address of the server which accepted a connection
Computing a value of this variable usually requires one system call. To avoid a system call, the listen directives must specify addresses and use the bind parameter.

$server_port
port of the server which accepted a connection
$time_iso8601
local time in the ISO 8601 standard format
$time_local
local time in the Common Log Format