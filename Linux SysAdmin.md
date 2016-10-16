
Linux文件系统
磁盘管理
RAID
LVM
程序安装
网络功能
sed命令
进程
内核管理
系统启动流程
定制、并编译内核
裁减Linux
系统安装：kickstart, pxe
shell编程：case，function, array, signal
gawk


Linux文件系统：
	存储空间：数据区，元数据区

		stat命令：元数据
			文件名，大小，时间戳，权限，属主、属组，对应的数据存储在哪些磁盘块上；

			index node: 索引区域中每个文件元数据条件
				每个inode都有其编号：ls -i

				如果某inode指向的常见类型的文件（f, d），指定向磁盘的数据区中的某个或某些个磁盘块

				目录：数据区存储的是（直接附属于此目录）文件名，以及与其对应的inode编号，
					dentry: 目录项

			格式化：创建文件系统

			bitmap：位图索引
				inode位图

				block bitmap

		块：块组

		super block:

		链接文件：
			/var/log/messages
			/tmp/log/abc inode: /var/log/messages

			一个inode可以被引用多次，其有计数器：在引用次为降为0之前是不会被标记为未用的。

			两个路径的文件名：指向同一个inode，此时，一个文件就称为另一个文件的硬链接

		创建链接：
			ln [-sv] SRC DEST

			硬链接：
				不能对目录文件创建硬链接；
				硬链接不能跨分区
				创建硬链接会增加inode引用计数

			符号链接：
				可以对目录创建
				不受分区限制
				对文件创建符号链接不会增加引用计数


		文件被删除：inode被标记为空闲，此inode指向的磁盘块被标记为空闲；
			如果inode被引用了多次，且此次删除未使得其引用计数降低为的话，这意味着文件被删除仅删除了一个访问路径；

		文件复制：创建一个新文件，并原文件中数据在新文件指向的磁盘块中再写一次的过程；

		文件移动：
			在同一个分区移到：移动文件仅是改变了文件访问路径；
			跨分区移到：在新分区创建文件，把数据复制过去，删除原分区数据；

	Linux的文件系统的类型：
		ext(2,3,4), xfs, ffs, ufs, reiserfs, jfs, vfat(fat32), ntfs

		交换文件系统：swap

		网络文件系统：nfs, smbfs(cifs)

		分布式文件系统：ceph

		光盘文件系统：isso9660

		btrfs, 


磁盘管理：

	I/O设备：
		磁盘
		网卡

	文件：read, write, open, close

	设备文件：特殊文件
		只有inode，而没有数据

		关联至一个驱动程序，进而跟对应的硬件设备打交道；

		/dev: b, c

			设备号：
				主设备号：用于标记设备类型
				次设备号：用于标记同一类型中的不同设备

			mknod [OPTION]... NAME TYPE [MAJOR MINOR]

			设备文件的命名：由ICANN

	磁盘设备文件：
		/dev/hd:
			IDE: 并口, 133MB/s
		/dev/sd:
			USB: 串行
			SATA: 串行，6Gbps/8
			SCSI: 并行，(Small Computer System Interface)
			SAS：串行，

		/dev/sd[a-z]
			分区：数字
				/dev/sda1
				/dev/sda2

				分区编号：
					主+扩展：1-4
					逻辑：5

	硬盘分区：
		磁道：track
			扇区：sector

		柱面：cylinder：
			分区根据柱面划分


		MBR：Master Boot Record
			主引导记录
			512Bytes: 引导启动OS
				446bytes: 程序，bootloader
				64bytes: 分区表，每16bytes标记一个分区，一共4分区
				2bytes: 5A, MBR有效性标记

			1T: 3主+1扩展（切割1个或多个逻辑）

		/proc: 

	分区创建：
		查看：fdisk -l [DEVICE]...

		创建分区：fdisk [DEVICE]
			交互式界面，有许多子命令
				p: 显示磁盘分区表
				n: new，新建分区
				d: delete，删除分区
				t: 修改分区的系统ID
				l: 列表出所有已经的系统ID
				w: 保存并退出
				q: 不保存退出

			对于已经有分区处于使用状态的磁盘来讲，新建分区后需要让内核重读其分区表：
				CentOS 5:
					# partprobe [DEVICE]
				CentOS 6:
					partx -a [DEVICE]
					kpartx -af  [DEVICE]

			分区创建工具：sfdisk和parted

	练习：写一个脚本
		1、提示输入一个对其执行分区的新硬件设备文件；
		2、提醒用户接下来的操作销毁所有的数据，你继续吗？
			y或yes: 继续
			n或no：退出
			其它字符：提醒输错了，再来一次；
		3、对磁盘新建分区：
			主分区1：大小512M，
			主分区2：大小2G
		4、创建完成后显示创建的结果；

回顾：
	Linux文件系统：ext(2,3,4), xfs, reiserfs(suse),
		VFS

		硬链接和符号链接：

		设备文件：
			b
			c

			mknod [option] NAME TYPE [MAJOR MINOR]

				/dev/null: 软件模拟的设备

		/dev/sd[a-z]#
			1-4: 
			5+: 

			IDE：/dev/hd

		分区：柱面 cylinder

		fdisk, sfdisk, parted

创建文件系统：
	mkfs : make file system
		-t FSTYPE [DEVICE]

		mkfs -t FSTYPE = mkfs.FSTYPE
			mkfs -t ext4 = mkfs.ext4

	注意：CentOS
		Linux内核是模块化的，这些模块支持动态装载和卸载；
			文件系统可能会被直接打包进内核，也可以被编译成内核模块；

				# lsmod 

		如果期望将某分区格式化成某特定文件系统，通常需要一个与之对应的在用户空间可使用命令行工具来实现；
			# yum -y install xfsprogs

		文件系统的日志功能：Journal
			ext2: 无日志功能
			ext3, ext4, xfs: 日志文件系统

	mke2fs:
		-t {ext2|ext3|ext4}：指定文件系统 
		-b {1024|2048|4096}：指定块大小
		-L LABEL: 打标
		-j: 相当于 -t ext3
		-i #: 每多少字节给创建一个inode，此字节数不应该少小块大小
		-N #: 直接指定可用的inode数；
		-m #: 指定预留空间占整个分区空间的百分比；默认为5；

		-O：指定分区特性


	blkid: 查看指定块设备的信息

	e2label：查看或设定卷标
		e2label DEVICE [LABEL]

	查看超级块信息：
		# tune2fs -l DEVICE
		# dumpe2fs -h DEVICE

	修改分区属性：tune2fs
		块大小无法调整；

		-j: ext2 --> ext3
		-L LABEL：修改卷标
		-m #: 修改预留空间百分比；
		-O [^]FEATURE: 启用指定特性，特性前加^，表示关闭此种特性

		-o [^]mount-options: 开启或关闭指定的挂载选项

	因进程意外中止或系统崩溃等情况导入写入操作非正常中止时，可能会导致文件损坏；此时，应该修复文件系统：
		注意：离线修复

		fsck
			-t fstype
			-a: 自动修复错误
			-r: 交互式修复错误

		e2fsck: 专用于修改ext系列文件系统
			-y: 对问题自动回答为yes 
			-f: 强制进行检测

	补充：windows不能识别Linux的文件系统
		U盘的文件系统FAT32
			# mkfs -t vfat 

	交换分区：swap
		缓解物理内存资源不够用的情况；

		创建交换分区：
			mkswap [-L LABEL] DEVICE

	文件系统挂载：默认只有管理员才有权限
		将额外的分区与根文件系统上的某目录建立关联关系的过程；
			目录中的原有文件会被隐藏

		挂载点：另一个文件系统的访问入口

		挂载： mount DEVICE MOUNT_POINT

			固定挂载点：/mnt, /media

			mount [option]... [-t fstype] [-o option]  设备  挂载点
				挂载点:
					1、事先存在；
					2、使用空闲目录；

				常用的挂载选项：
					-t fstype：指定文件系统类型
					-r: readonly, 只读挂载
					-w: read and write， 读写挂载
					-L LABEL：以卷标方式指定设备， mount -L MYDATA 挂载点
					-U UUID: 以UUID的方式指定设备，mount UUID='uuid' 挂载点, mount -U uuid 挂载点
					-a: 自动挂载所有(/etc/fstab文件中定义的)的支持自动挂载设备
					-n: 挂载时，不更新/etc/mtab文件

				-o option: 
					async：异步I/O，数据写操作先于内存完成，而后再根据某种策略同步至持久设备中
					sync: 同步I/O，
					atime/noatime: 文件和目录被访问时是更新最近一次的访问时间戳
					auto/noauto：设备是否支持mount的-a选项自动挂载
					diratime/nodiratime: 目录被访问时是更新最近一次的访问时间戳
					dev/nodev: 是否支持在此设备上使用设备；
					exec/noexec: 是否允许执行此设备上的二进制程序文件
					suid/nosuid: 是否支持在此设备的文件上使用suid
					remount: 重新挂载，通常用于不卸载的情况下重新指定挂载选项
					ro: 只读
					rw: 读写
					user/nouser: 是否允许普通挂载此文件设备
					acl: 在此设备是支持使用facl，默认不支持；

				例如：以指定挂载后支持acl为例：
					方法1：
						mount -o acl DEVICE MOUNT_POINT

					方法2：
						tune2fs -o acl DEVICE
							为设备设定默认挂载选项

						mount DEVICE MOUNT_POINT


			查看所有已经挂载的设备：
				# mount
				# cat /proc/mounts
				# cat /etc/mtab

				卸载：挂载点没有被进程访问时方可以卸载 ；


		卸载：umount DEVICE
			  umount MOUNT_POINT

			 查看哪些进程正在访问挂载的设备：
			 	fuser -v 挂载点

			 中止正在此挂载点的进程：
			 	fuser -km 挂载点

		df: disk free
			-h: human-readable
			-i: 显示inode的使用信息而非默认的磁盘空间使用信息

		du: disk usage
			-s: 
			-h: 

		练习：创建一个20G的分区，块大小为2048，预留百分比为3，卷标为MYDATA，要求挂载后支持acl，使用UUID的方式挂载至/mydata目录；
			使用重新挂载的功能，让其不支持dev功能；

	交换分区：
		mkswap

		free: 查看内存及交换分区的使用信息

		启用某交换分区设备
			swapon [DEVICE]
				-a: all, 启用所有交换分区
				-p #: 指定此交换设备的优先级

		禁用某交换分区设备
			swapoff [DEVICE]
				-a: 禁用所有

	自动挂载的设备的配置文件：/etc/fstab
		6字段：
			挂载的设备：
				设备文件
				LABEL
				UUID
			挂载点：
			文件系统类型
			挂载选项：
				挂载选项可以有多个，彼此间使用逗号分隔；
			转储频率：
				0：从不转储
				1: 每天转储
				2: 每隔一天
			自检次序：
				0：不自检，额外创建的文件系统都无须自动自检
				1：首先自检，通常只有根文件系统需要首先自检
				2：次级自检，不同的设备可以使用同一个自检次序
				3
				...



	练习：写一个脚本，完成如下功能
		1、列出当前系统上的所有磁盘设备；
		2、让用户选择一个磁盘设备，并在选择后显示指定设备上的所有分区信息；
		3、询问用户是否擦除此磁盘上的所有现存分区后重新添加三个分区；
			y或yes: 继续
			n或no: 中止脚本
			其它字符则提醒用户重新输入合法的字符
		4、在用户选择是后执行相应的分区操作
			创建三个分区
				主分区1：512M，ext4
				主分区2: 512M，swap
				主分区3：2G，ext4
		5、将创建的分区按如上说明分别格式为相应的文件系统；
		6、将主分区1挂载至/mnt/boot目录，主分区3挂载至/mnt/sysroot目录；

		扩展：在上述第3个步骤开始之后，先查看此设备上是否有分区被挂载，如果有，则先卸载之；

		# fdisk -l | awk '/^Disk \/dev\/[sh]d[a-z]/{print $2}' | tr -d ':'
		/dev/sda
		/dev/sdb


回顾：
	/etc/fstab:
		挂载的设备：
			设备文件：/dev/sda3
			LABEL：LABEL=MYDATA
			UUID：UUID=
		挂载点：
		文件系统类型：swap, ext4, xfs
		挂载选项：defaults,
		转储频率
		自检次序

	tune2fs:
		-m

	df 
		-h
		-i

	命令总结：fdisk, mkfs, mke2fs, mkswap, mount, umount, swapon, swapoff, fsck, e2fsck, dumpe2fs, tune2fs, df, du, mknod, ln, free, e2label, blkid, fuser


Linux RAID
	
	IDE, SCSI

	RAID: Redundant Array Inexpensive Disks

	RAID: Independent

	RAID Level
		raid0: 读、写性能提升，无容错能力，空间n*disk
		raid1: 写性能略有下降，读性能提升，容错，空间：1*disk
		raid4: 读、写性能提升，有容错能力（最多坏一块磁盘），空间：(n-1)*disk
			1 & 0 = 1
		raid5: 读、写性能提升，有容错能力（最多坏一块磁盘），空间：(n-1)*disk

		raid10, raid01

		raid10: 读、写性能提升，有容错能力（每一组可坏一块盘），空间：n*disk/2

		raid6: 有两块校验盘，容许同时坏两块，至少需要4块盘，空间：(n-2)*disk

	md: multidisks
		mdadm

		ext4(kernel)
		mke2fs(user)

	dm: device mapper
		lvm 

	mdadm工具：
		md: 支持将任何块设备组织成RAID

			sda, 
			raid5: sda1, sda2, sda3
				3*200G: 400G

		mdadm: 模块式化命令
			-A: 装配模式，重新识别此前实现的RAID
			-C：创建模式，创建RAID
			-F：监控模式

			管理模式：-f, -r, -a

		-C: 创建模式中专用选项
			-n #: 用于创建RAID设备的磁盘个数；
			-l #: 级别
			-a yes: 自动为创建的RAID生成设备文件;
			-c Chunk_Size: 

			md设备的设备文件，一般在/dev目录下，以md开头，后跟一个数字来区别

			# cat /proc/mdstat
				/proc/mdstat: 当前系统上所有已启用的软件RAID设备及其相关信息

		例如：创建一个10G空间的RAID0
			# mdadm -C /dev/md0 -a yes -n 2 -l 0 /dev/sdb{1,2}

		例如：创建大小为10G空间的RAID5：
			3*5G，6*2G
				(n-1)*2G

			# mdadm -C /dev/md1 -a yes -n 3 -l 5 /dev/sda{3,5} /dev/sdb3

	RAID: 组织多块硬盘当一个设备来使用
		硬件RAID：
			硬件控制器：创建RAID通过BIOS进行
				在OS中看到的仅是一个单独的设备

		软件RAID：
			无需任何硬件的RAID设备，仅需多个块设备（磁盘分区即可）
				在OS中看到的是多个基本的磁盘设备、磁盘分区等，而后将这多个块设备可以组织一个单独的设备使用
				即为软RAID


			装配模式：在某OS上创建的软件RAID，被迁移到其它主机上，并启动OS之后，
				Linux auto detect

		mdadm: 命令行工具，结果md模块实现软件RAID
			模式化工具：-C, -F, -A, 
				-a, -r, -f

			mdadm -D /dev/md#
				显示指定的软RAID的详细信息

			mdadm /dev/md# -f /dev/some_device
				将/dev/md#中的/dev/some_device手动设置为损坏

			mdadm /dev/md# -r /dev/some_device
				将/dev/md#中的损坏状态的/dev/some_device移除

			mdadm /dev/md# -a /dev/new_device
				新增设备

		练习：创建一个10G的RAID1？

		练习：创建一个可用空间为10G的RAID5设备，要求其chunk大小为256K，文件系统为ext4，开机可自动挂载至/backup目录，支持acl功能；有一个空闲盘；
		练习：创建一个可用空间为10G的RAID1设备，chunk大小为1024k，有一个空闲盘，开机自动挂载至/users目录，不支持dev设备文件；

	停止软件RAID：
		mdadm -S /dev/md#

	重新启用RAID：
		mdadm -A /dev/md# /dev/DEVICE...

		mdadm的配置文件/etc/mdadm.conf

	RAID：理解RAID各级的特性


	watch [-n #] <COMMAND>:
		阶段性地执行指定的COMMAND
			-n #: 指定间隔时间


bash编程：
	循环：
		循环次数已知：遍历
			for
		循环次数未知：
			while
			until

	for格式：
		for 变量名 in 列表; do
		    循环体
		done

		循环体：依赖于调用变量来实现其变化；

		循环可以嵌套；

		退出条件：遍历列表完成

	while格式：


		while 测试条件; do
			循环体
		done

		测试条件为真，进入循环；测试条件为假，退出循环；
			测试条件一般通过变量来描述，需要在循环体不变量地改变变量的值，以确保某一时刻测试条件为假，进而结束循环；

	until格式：


		until 测试条件; do
			循环体
		done

		测试条件为假，进入循环；测试条件为真，退出循环；
			测试条件一般通过变量来描述，需要在循环体不变量地改变变量的值，以确保某一时刻测试条件为真，进而结束循环；

	for的第二种使用格式 ：
		for ((初始条件;测试条件;修改表达式)); do
			循环体
		done

		求100以内所有正整数之和：
			while的实现：

				#!/bin/bash
				declare -i counter=1
				declare -i sum=0

				while [ $counter -le 100 ]; do
					let sum+=$counter
					let counter++
				done

				echo $sum

			for的实现：
				#!/bin/bash
				#
				declare -i sum=0

				for ((counter=1;$counter <= 100; counter++)); do
					let sum+=$counter
				done

				echo $sum

			练习：求100以内所有偶数之和；

				declare -i evensum=0
				for ((counter=2; $counter <=100; counter+=2)); do
					let evensum+=$counter
				done

	while循环：遍历文本文件

		格式：

		while read 变量名; do
			循环体
		done < /path/to/somefile

		变量名，每循环一次，记忆了文件中一行文本

		
		练习：显示其ID号为偶数的用户的用户名、ID号和SHELL

			#!/bin/bash
			#

			while read line; do
			    userID=`echo $line | cut -d: -f3`
			    if [ $[$userID%2] -eq 0 ];then
			        echo $line | cut -d: -f1,3,7
			    fi
			done < /etc/passwd

		练习：显示ID号为偶数，且ID号同GID的用户的用户名、ID和SHELL；
		while read line; do
			userID=`echo $line | cut -d: -f3`
			groupID=`echo $line | cut -d: -f4`
			if [ $[$userID%2] -eq 0 -a $userID -eq $groupID ]; then
			        echo $line | cut -d: -f1,3,7
			    fi
			done < /etc/passwd				

		练习：显示当前系统上所有挂载的文件系统中空间使用百分比大于10的设备和空间使用百分比；使用while循环实现；

			提示：此题目可以使用管道

回顾：
	RAID, md, mdadm, for, while

	容错：1, 4, 5, 6, 10
	写性能：0, 4, 5, 6, 10
	raid5: D
		(n-1)*D
		6: (n-2)*D
		10: n*D/2

LVM: Logical Volumn Manager
	lvm, lvm2

	dm: device mapper

	将一个或多个底层块设备组织一个逻辑的工具

	lv, multipath

	Block Devices:
		Pysical Extent
			PE: 大小固定

		存储空间边界：
			物理边界：
			逻辑边界

		逻辑卷：
			扩展：物理 --> 逻辑
			缩减：逻辑 --> 物理
				缩减不能少于已经存储的所有数据空间的大小

		卷组：	

		快照：snapshot

			数据：100G

	LVM：
		块设备：分区，RAID
			pv --> vg --> lv

			pv:
				pvcreate, pvs, pvdisplay, pvremove, pvmove, pvscan
			vg:
				vgcreate, vgs, vgdisplay, vgremove, vgextend, vgreduce, vgscan
			lv:
				lvcreate, lvs, lvdisplay, lvremove, lvextend, lvreduce, lvscan

			例如：10G的vg,
				1:10G PV
				2: 3+7G PV

		创建逻辑卷：
			lvcreate
				-n lv_name
				-L #UNIT   {mMgGtT}
				VG_NAME

			lv的访问路径：
				1、/dev/VG_NAME/LV_NAME
					/dev/myvg/mylv

				2、/dev/mapper/VG_NAME-LV_NAME
					/dev/mapper/myvg-mylv

				此两者均为符号链接，指向的文件为/dev/dm-#


		如何扩展逻辑卷：
			1、先确定扩展的目标大小；并确保对应的卷组中有足够的空闲空间可用；
				2G, 目标为4G
					+2G
					4G
			2、扩展物理边界
				lvextend -L 4G /dev/myvg/mylv
			3、扩展逻辑边界
				resize2fs  /dev/myvg/mylv

	缩减很危险！！！！
		缩减要离线
		1、先确定缩减后的目标大小；并确保对应的目标逻辑卷大小中有足够的空间可容纳原有所有数据；
		2、先制裁文件系统，并要执行强制检测
			e2fsck -f 
		3、缩减逻辑边界
			resize2fs DEVICE
		4、缩减物理边界
			lvreduce

			慎独

	创建快照卷：
		lvcreate 
			-L 
			-n
			-s 
			-p r

		注意：快照卷是对某逻辑卷进行的，因此必须跟目标逻辑卷在同一个卷组中；无须指明卷组；

	练习：
	1、创建一个由至少两个物理卷组成的大小为10G的卷组；要求，PE大小8M；而后在卷组中创建大小为5G的逻辑卷mylv1，格式化为ext4文件系统，开机自动挂载至/users目录；
	2、新建用户mageedu，其家目录为/users/mageedu，而后su至mageedu用户，复制/etc/fstab文件至自己的家目录；
	3、扩展mylv1至7G，确保/users/mageedu的数据不受影响；而后su至mageedu用户，验正数据可正常访问；
	4、缩减mylv1至4G，确保/users/mageedu的数据不受影响；而后su至mageedu用户，验正数据可正常访问；
	5、对mylv1创建快照卷snap-mylv1，并通过其cp内部的数据至/backups/目录中，要求保留原有属主属组等信息；


逻辑卷

dd 
	if=/path/to/src_file
	of=/path/to/dest_file
	bs=256K
	count=#

	100M

	dd if=/dev/zero of=/dev/sdb bs=512 count=1

	/dev/null: 吞进所有数据，直接丢弃
	/dev/zero: 泡泡机，吐零

	dd if=/dev/sdb of=/backup/mbr.backup bs=1 count=512
	dd if=/backup/mbr.backup of=/dev/sdb bs=512 count=1

	swap空间吃紧，创建新的swap设备

Linux压缩和解压缩工具
	
	压缩：根据一定算法
		文本文件

	compress/uncompress, .Z

	gzip/bzip2/xz

		gzip/gunzip
				后缀名：.gz
			-#: 压缩比，默认为6，范围为0-9；
			-d : gunzip
			-c: 将压缩后的结果输出至标准输出，
				gzip -c /path/to/somefile > /path/to/somecfile.gz

			zcat somefile.gz: 不解压查看gzip压缩后的文件的内容

		bzip2/bunzip2
			.bz2
			-#: 压缩比，默认为6，范围为1-9；
			-d: bunzip2
			-k: 保留原文件

			bzcat somefile.bz2: 


		xz/unxz
			.xz
			-#: 压缩比，默认为6，范围为0-9；
			-d: unxz
			-k: 保留原文件

			xzcat somefile.xz

		zip/unzip
			zip ZIPFILE.zip src_file...
			unzip ZIPFILE.zip

	归档工具：tar
		能实现将多个文件打包成单个文件，即为归档文件

		tar [option]... -f tarfile.tar src_file...

		创建归档：
			tar
				-c: create
				-f FILE.tar: 指定归档后的文件
				-v: 显示执行过程

		展开归档：
			tar 
				-x: extract
				-v:
				-f FILE.tar

		查看归档后的文件中包含了哪些原文件：
			tar
				-t: 
				-f FILE.tar

		tar可直接通过选项调用压缩工具执行压缩或解压：
			-z: gzip
			-j: bzip2
			-J: xz

	cpio: 

命令总结：dd, gzip, gunzip, zcat, bzip2, bunzip2, bzcat, xz, unxz, xzcat, zip, unzip, tar

挂载光盘：
	mount /dev/cdrom /media/cdrom

	文件系统：iso9660

	eject

挂载U盘：

练习：写一个脚本
	1、显示如下信息：
		Plz choose a compress tool:
		用户可键入：gzip, bzip2或xz;
	2、提醒用户指定要归档压缩的文件
	3、将用户归档压缩至/tmp目录中，文件名为原名加上相应的后缀；



	


网络配置与管理：

	冲突域：
		网桥：
		多接口：交换机

		学习：

	广播域：

	8位2进制数据：0-255

	0-255.0-255

1.1 -->  2.2

	1111 1111. 0000 0000

1.0
2.0

IP: 4段
	1.1.1.1

32位：
	大：0 000 0000 - 0 111 1111：0-127
	中：10 00 0000 - 10 11 1111：128-191
	小：110 0 0000 - 110 1 1111：192-223
	d	1110 0000 - 1110 1111：224-239
	e	1111 0000 - 1111 1111：240-255

全0：网络地址
全1：广播地址

A：1-126
	126个网络
	每个网络中的主机：2^24-2
B：128-191
	2^14个网络
	每个网络中的主机：2^16-2
C：192-223
	2^21个网络
	每个网络中的主机：2^8-2


172.16.0.0
255.255.0.0

172.16.0.0/255.255.255.0
172.16.255.0/255.255.255.0

172.16.255.255

192.168.0.0/255.255.255.0 --> 192.168.255.0/255.255.255.0

192.168.0.0/255.255.0.0

子网，超网

IANA, ICNAA

A类：
1-126个网络，127用于回环，2^7-1
容纳的主机数：2^24-2
默认掩码：255.0.0.0
主机位全0：网络地址
主机位全1：广播地址

B类：
2^14个网络
容纳的主机数：2^16-2
默认掩码：255.255.0.0

C类：
2^21个网络
容纳的主机数：2^8-2
默认掩码：255.255.255.0


A类：
1个：10.0.0.0/255.0.0.0

B类：
16个：172.16.0.0/255.255.0.0-172.31.0.0/255.255.0.0

C类：
256个：192.168.0.0/255.255.255.0-192.168.255.0/255.255.255.0


路由表：
	静态设置
	动态生成

	cost: 成本
		经过的跳数越少就越小
		时长

		路由协议：RIP2, OSPF, EIGRP
		可路由协议：IP协议

物理介质：物理层，物理层协议
链路层：数据帧，链接层协议
网络层：数据包，IP协议
传输层：传输层  （TCP,UDP）
	TCP：0-65535
	UDP：0-65535
应用层：标记资源



端口：用于标记进程
	0-65535：


Socket: IP:port
	172.16.100.7:80
	172.16.100.8:80

协议栈：内核
	TCP/IP协议簇

主机名：FQDN
	Full Qulified Domain Name
www.magedu.com

名称解析：DNS

MTU：Maximum Translate Unit



Fragment: IP分片

有限状态机

FSM: Finite State Machine


网络基本知识：
	TCP/IP：
		物理层：
		链路层：MAC
		网络层：IP报文
		传输层：TCP/UDP
		应用层：

	链路层：从设备到设备主机通信，MAC地址，IP<-->MAC (ARP/RARP)
		MTU：
	网络层：从源主机到目标主机之间通信，IP地址，IP报文
	传输层：从源主机进程到目标主机特定进程之间通信，TCP/UDP
	应用层：

	ISO/OSI：七层
		1-4：通信
		5-7：资源
			会话层
			表示层
			应用层

	TCP：有连接协议，建立逻辑连接
		SYN, ACK, FIN, RST, PSH, URG

		三次握手：
			SYN=1, ACK=0, FIN=0
			SYN=1, ACK=1, FIN=0
			SYN=0, ACK=1, FIN=0

		四次断开：

		有限状态机：

主机：TCP/IP协议栈

回顾：IPV4：5类地址：
		A：10.0.0.0/8
		B: 172.16.0.0/16, 172.31.0.0/16
		C: 192.168.0.0/24, 192.168.255.0/24
		D
		E

	OSI：7 layers
		4 :
			TCP, UDP		

	tcp三次握手：
		1次：SYN＝1，ACK＝0，FIN＝0
		2次：SYN=1, ACK=1, FIN=0
		3次：SYN＝0，ACK＝1，FIN=0

	A --> B, B --> A
	
	IP首部，TCP首部

	VLAN: 

Linux网络属性配置：
	IP／NETMASK
	路由：
		主机路由
		网络路由
		默认网关
	DNS服务器：
		主DNS服务器
		备用DNS服务器
	主机名

	配置网络属性：
		静态配置
		动态配置：DHCP
			Dynamic Host Configuration Protocol
			

	配置IP：
		用户空间工具：ifconfig (net-tools), ip (iproute2)
		网络设备服务配置文件: /etc/sysconfig/network-scripts/
			主机名：/etc/resolv.conf
		GUI／TUI

	网络设备的配置方式：
		内核识别硬件设备：驱动
			
		设备名称：
			以太网：ethX
				eth0, eth1, eth2, ...
			PPP网络：pppX
			loopback: 本地回环，lo

	ifconfig:
		默认为显示所有处于激活状态的连接
		－a

		ifconfig IFNAME:仅显示指定接口的信息
		ifconfig IFNAME ADDRESS
			ip/mask
				长格式：ifconfig IFNAME IP netmask MASK
				短格式：ifconfig IFNAME IP/MASK

	route: 
		route：显示路由信息
			-n: 数字格式的地址

		route add
			-host：目标为主机
				-host HOST_IP gw NEXT_HOP [dev DEVICE]
			-net：目标是网络
				-net NET_ADDRESS gw NEXT_HOP [dev DEVICE]

				-net 0.0.0.0: 表示目标为任意地址

			route add default gw GW_ADD
		
		route del
			-host HOST_IP
			-net NET_ADDRESS

	DNS服务器地址：
		本地解析: /etc/hosts
		DNS服务器解析：指定DNS服务器地址

		dig -t A FQDN
			Full Qualified Domain Name
			www.magedu.com

		dig -x IP:
			反解IP至FQDN

	使用命令配置的信息直接送往内核（TCP／IP协议栈）并立即生效；

	IP／NETMASK：
		配置文件有两类(/etc/sysconfig/network-scripts):
			配置IP、掩码和网关：
				以太网：ifcfg-IFNAME
				PPP: ifcfg-pppX
			配置路由：route-IFNAME

		CentOS 5: /etc/rc.d/init.d/network
		CentOS 6: /etc/rc.d/init.d/network
			    /etc/rc.d/init.d/NetworkManager

	/etc/rc.d/init.d/，/etc/init.d/ *
		SysV风格的脚本：多数脚本都用于控制Linux的后台进程，接受参数{start|stop|restart|status}
		
		# /etc/init.d/network start
		# service network start	
		
		配置某服务是否开机自动运行：
			# chkconfig SRVNAME on｜off
		查看哪些服务开机自动运行：
			# chkconfig --list 


	ifcfg-IFNAME配置文件的格式：
		DEVICE＝IFNAME: 此配置文件所关联到的设备，设备名称要与本文件名ifcfg-后面保持一致； 
		BOOTPROTO＝{bootp|dhcp|static|none}
		HWADDR=00:11:22:33:44:55:66 :当前设备的MAC地址； 
		NM_CONTROLLED={yes|no}: 是否接受NetworkManager服务脚本来配置此设备； 
		ONBOOT={yes|no}: 是否在开机过程中，自动激活此接口
		TYPE＝{Ethernet|Bridge}: 网络接口类型
		UUID＝
		IPADDR＝
		NETMASK＝
		GATEWAY＝
		DNS1＝
		DNS2＝
		IPV6INIT＝{yes|no}
		USERCTL={yes|no}: 是否允许普通用控制此接口
		
		PEERDNS＝{yes|no}: 不接受DHCP服务器指派的DNS服务器地址

	route-IFNAME: 
		配置文件的格式1：每行一个路由条目
			DESTINATION via NETX_HOP

		配置文件格式2: 每三行一个路由条目
			ADDRESS＃＝DESTINATION
			NETMASK＃＝MASK
			GATEWAY＃＝GW

	如何配置主机名：
		hostname
		hostname HOSTNAME

		配置文件：/etc/sysconfig/network
			HOSTNAME=主机名


	如何在一个网络接口配置多个IP地址：

		通过网络接口的别名来实现：IFNAME:#
			ens33, ens33:0, ens33:1, ens33:2
			eth0，eth0:0, eth0:1

		命令配置：立即生效
			ifconfig IFALIAS IP

		配置文件配置：别名不支持使用DHCP进行配置
			ifcfg-IFALIAS
				DEVICE=IFALIAS
				BOOTPROTO={static|none}
				IPADDR=
				NETMASK=
				ONBOOT=
				USERCTL=


	TUI或者GUI：
		TUI: system-config-network-tui
		GUI：system-config-network-gui

		setup --> Network Configuration


		修改的结果会保存至相应的网络接口的配置文件ifcfg-IFNAME，因此，不会立即生效；

	网络管理相关的工具：
		ping: ICMP
			ping [option]... IP
				-c #: 报文的个数
				-W timeout: 等待响应报文的超时时长；

		traceroute: 
			traceroute HOST
				获取从当前主机到达目标主机所经由的所有网关；

		mtr HOST

		netstat: (ss)
			-t: tcp协议相关
			-u: udp协议相关
			-n: 显示数字格式的地址
			-l: listen，显示处于监听状态的连接
				-tunl
			-a: 所有状态的连接
				-tan
			-p: 显示会话中的进程程序名及进程号
			-r: routing，显示路由表
				-rn

			名称解析：
				FQDN <==> IP
				Service Name <==> PORT

	显示网络接口设备的属性信息：
		ethtool IFNAME
			-S: 显示设备接口的统计数据

	课外任务：nmap, ncat, tcpdump

	ip命令：
		ip link : 管理接口
			show [IFNAME]
			set IFNAME {up|down}
				multicast {on|off}

			# ifconfig IFNAME {up|down}
			# ifup IFNAME
			# ifdown IFNAME

		ip addr: 管理协议地址
			ip addr {show|flush} [dev DEVICE] 

			ip addr {add|del} ADDRESS dev DEVICE  [label IFALIAS] [broadcast BCAST_ADDRESS]

				# ifconfig IFNAME ADDRESS broadcast BCAST_ADDRESS

		ip route: 管理路由
			ip route list

			ip route flush 

			ip route add DESTINATION [via NEXT_HOP] [src SOURCE_ADDRESS] [dev DEVICE]
			ip route del DESTINATION

命令总结：ifconfig, ifup, ifdown, route, netstat, ping, traceroute, mtr, ethtool, setup, dig, ip, ss
	
	ss:
		-t: tcp
		-u: udp
		-p: process
		-l: listening
		-n: numeric
		-a: all
		-e: 扩展信息
		-m: 套接字相关的内存使用信息
		-o state {established,fin_wait_1, fin_wait_2, listening}
			'( dport =   or sport =  )'
			只显示指定状态的连接，还可以指定过滤条件

练习：写一个脚本
	扫描172.16.250.0/16内的所有主机；在线的，使用绿色显示；不在线，使用红色显示；

	最后分别显示：在线和不在线各有多少主机；

	for (())
		if ping -c 1 -W 1 172.16.250.$host &> /dev/null; then

练习：写一个脚本
	1、提示用户输入一个IP地址；如果用户输入的地址不正确，则提醒其重新输入直到正确为止；
	2、如果正确，则添加至eth0上的辅助地址；
	3、得出正确的掩码；

	if [[ $ip =~  ]]; then
		ip addr add










Bonding的模式一共有7种： 
#define BOND_MODE_ROUNDROBIN       0   （balance-rr模式）网卡的负载均衡模式  
#define BOND_MODE_ACTIVEBACKUP     1   （active-backup模式）网卡的容错模式  
#define BOND_MODE_XOR              2   （balance-xor模式）需要交换机支持  
#define BOND_MODE_BROADCAST        3    （broadcast模式） 
#define BOND_MODE_8023AD           4   （IEEE 802.3ad动态链路聚合模式）需要交换机支持  
#define BOND_MODE_TLB              5   自适应传输负载均衡模式
#define BOND_MODE_ALB              6   网卡虚拟化方式

bonding模块的所有工作模式可以分为两类：多主型工作模式和主备型工作模式，balance-rr 和broadcast属于多主型工作模式而active-backup属于主备型工作模式。（balance-xor、自适应传输负载均衡模式（balance-tlb）和自适应负载均衡模式（balance-alb）也属于多主型工作模式，IEEE 802.3ad动态链路聚合模式（802.3ad）属于主备型工作模式










回顾：
	Linux网络功能：TCP/IP
		IP包头，TCP包头

	IP/NETMASK
	路由
	DNS服务器地址
	主机名
		www.magedu.com

	ifconfig, route, netstat, ip, ss


Linux程序包管理:
	
	Linux: GPL, BSE, Apache,

		源程序包：C, C++, perl

			预处理--> 编译  --> 汇编 --> 链接

		库：功能模块
			名称、函数（参数）
				函数名，所能够接受的参数个数，类型

				输入输出、数学计算、字符串处理
					函数

				头文件：head

	include <stdio.h>

	库：程序（无执行入口，不能独立执行，只能被能独立运行的程序调用时执行）
		库：源代码 --> 二进制格式

	应用程序程序员：开发环境 （API）
		依赖于：头文件
				库文件（开发库，运行库）

		POSIX API

	终端用户：
		程序（编译完成）
			依赖于：
				静态编译
				动态编译：
					运行库
					dll, so (shared object)

				ABI: Application Binary Interface

	源代码：终端用户
		安装：编译
			X86_64(64bits): 
			i686(32bits):
			ppc

		二进制格式：依赖于硬件平台

	API：一种源代码
		arm: 编译出以后，严重依赖于arm

	程序员：testapp.tar.gz

		testapp.x86_64
		testapp.i686
		testapp.arm

		testapp.noarch

	有些程序不依赖于硬件运行，依赖于虚拟机，不再依赖于低层硬件平台，noarch

	程序包的组成格式：
		二进制程序
			/bin, /sbin, /usr/bin, /usr/sbin, /usr/local/bin, /usr/local/sbin, /usr/local/APP/{bin,sbin}
			
			注意：
				有些特殊的应用程序放置于libexec目录中
				有些第三方应用默认安装于/opt目录
		库文件（开发库、运行库）
			/lib64, /usr/lib64, /usr/local/lib64, /usr/local/APP/lib
		配置文件
			/etc, /usr/local/APP/etc或conf目录
		帮助文件
			/usr/share/man, /usr/local/share/man, /usr/local/APP/man

			帮助文件：man, info, 
				doc: README, INSTALL, ChangeLog

		ldd:
			ldd /path/to/binary_file


	程序包管理器：
		1、数据库
			程序名及版本
			依赖关系：X --> Y,Z 
			功能性说明
			安装生成的各文件路径及校验码
		2、程序的组成清单
			文件清单
			安装卸载时运行的脚本

		功能：将编译好的程序打包成一个文件或有限的几个文件，可用于实现安装、卸载、升级、查询等功能；

	应用程序的命名格式：
		源代码：name-version.tar.{gz,bz2,xz}
			version: major.minor.release

			name-major.minor.release.tar.gz
				bash-4.2.3.tar.gz



	包管理器：
		Debian: dpkg, .deb
		RedHat: rpm(redhat package manager), .rpm
			RPM is Package Manager.

	包管理器的功能：
		打包
		安装
		卸载
		升级
		校验
		数据库管理

	程序间的依赖关系：繁杂
		X --> Y,Z 
			Y--> M,N
			Z --> A,B

	前端工具
		dpkg --> apt-get
		rpm --> yum 

	程序包管理器：
		打包：包括编译，打包

	1、rpm包
	2、yum
	3、源代码编译安装



rpm包的使用：

	源代码：
		name-version.tar.{gz,bz2,xz}
			version：major.minor.release
	
	rpm包的命名格式
		name-version-relase.arch.rpm

			version: major.minor.release，同源代码
			release: rpm自身的发行号，与程序源码的发行号无关，仅用于标识对rpm包不同制作的修订；同时，release还包含此包适用的OS
				bash-4.2.3-3.centos5
			arch: 适用于的硬件平台，
				x86: i386, i486, i586, i686等；
				x86_64: x86_64
				powerpc: ppc
				noarch: 依赖于虚拟机
					例如：bash-4.2.3-3.centos5.x86_64.rpm

		一个程序有20个功能：常用功能有8个，特殊A:3个，特殊B：6个，二次开发相关功能：3
			分包机制：
				核心包，主包：命名与源程序一致
					bash-4.2.3-3.centos7.x86_64.rpm
				子包：
					bash-a-4.2.3-3.centos7.x86_64.rpm
					bash-b-4.2.3-3.centos7.x86_64.rpm
					bash-devel-4.2.3-3.centos7.x86_64.rpm

				OS Vendor: 系统发行商提供的包

			获取rpm包的途径：
				1、发行的光盘或站点服务器
					镜像：
						http://mirrors.163.com
						http://mirrors.sohu.com
				2、项目的官网
					源代码
					rpm包
				3、很多第三方机构或个人制作并公开发布许多rpm包
					http://rpmfind.net
					http://rpm.pbone.net

				可靠的途径：EPEL
					Fedora-EPEL

		rpm包的合法性验正：
			包制作者制作完成之后会附加数字签名于包上；
				来源合法性
				包的完整性

				包的制作者使用单向加密提取原始数据的特征码，而后使用自己的私钥加密这段特性码，附加原始数据后面。
				验正过程：
					前提：必须有可靠机制获取到包制作者的公钥；
					1、使用制作者的公钥解密加密的特征码，能解密则意味着来源合法；
					2、使用与制作者同样的意向加密算法提取原始数据的特征码，并与解密出来的特征作比对，相同，则意味着完整性没问题；

	rpm包管理器的常见使用场景：


		安装程序包：
			rpm [option] /path/to/package_file 
				-i: install
				-v:
				-vv:
				-vvv:
				-h: 

				组合选项：-ivh

				--test: 仅作测试，有真正执行安装

				如果依赖于其它包：
					1、解决依赖关系
					2、忽略依赖关系
						能安装上，但有可能无法运行；
						--nodeps

				重新安装：
					--replacepkgs

						如果原有配置文件作了修改，很有可能不执行替换，而是将应该安装生成的配置文件重命名为 .rpmnew

		卸载程序包：
			rpm [option] package_name
				-e: erase

				如果被其它包所依赖：
					1、将依赖于此包的所有包一并卸载
					2、忽略依赖关系
						能卸载，但依赖于此包程序包可能会运行不正常；
						--nodeps

				如果包的配置文件安装后曾被改动过，卸载时，此文件将不会卸载，而是被重命名并保留，例如
					warning: /etc/zprofile saved as /etc/zprofile.rpmsave

		升级程序包：
			新版本替换老版本
			rpm [option] /path/to/package_file
				1、升级或安装
					-Uvh

				2、纯升级
					-Fvh

				X --> Y-2.2.1
					Y-2.2.3
					升级后的版本冲突等；

				--force: 强制升级

			注意：不应该对内核执行升级操作，而是安装
				系统允许多内核并存

		查询操作：
			1、查询某包是否安装
				rpm -q package_name...

			2、查询所有已经安装的包
				rpm -qa 

				按条件过滤：rpm -qa | grep 'PATTERN'

			3、查询包的描述信息
				rpm -qi package_name

			4、查询某包安装生成了哪些文件
				rpm -ql package_name

				(1) 查询某包安装生成了哪些配置文件
				rpm -qc package_name

				(2) 查询某包安装生成了哪些帮助文件
				rpm -qd package_name

				(3) 查询程序包的相关脚本
				rpm -q --scripts package_name

					脚本有四类：
						preinstall：安装前脚本
						postinstall: 安装后脚本
						preuninstall: 卸载前脚本
						postuninstall: 年前后脚本

			5、查询某文件是由哪个包安装生成的
			rpm -qf /path/to/some_file

			6、对尚未安装的包执行查询
			rpm [option] /path/to/package_file
				-qpi
				-qpl
				-qpc
				-qpd


		校验：
			用于检查包安装生成的文件属性是否发生变化

			rpm -V package_name

		       S file Size differs
		       M Mode differs (includes permissions and file type)
		       5 digest (formerly MD5 sum) differs
		       D Device major/minor number mismatch
		       L readLink(2) path mismatch
		       U User ownership differs
		       G Group ownership differs
		       T mTime differs
		       P caPabilities differ

		      某属性无变化，显示为.

	rpm包来源合法性及完整性检验：

		前提：在当前系统上导入包的制作者的公钥
			导入：
				rpm --import /path/to/key_file

				# rpm -qa gpg-pubkey*
				显示所有已经导入的gpg格式的公钥

				# rpm -qi gpg-pubkey-NAME
				显示密钥的详细信息

			检查包：安装过程中会自动执行

			手动检查：
				rpm -K /path/to/package_file
				rpm --checksig /path/to/package_file

					不检查包完整性：
						rpm -K --nodigest
					不检查来源合法性：
						rpm -K --nosignature

	数据库重建：
		数据库目录：/var/lib/rpm

		重建:
			rpm --initdb：初始化
				如果事先没有库，会新建一个；如果有，则不新建；

			rpm --rebuilddb: 重建
				直接重建，覆盖原有的数据库

	总结：安装、卸载、查询、升级、校验、公钥导入、合法性验正、库重建

	第三篇博客作业：rpm包管理

Linux: dpkg, rpm
	debian
	centos

	制作包：
		分包机制：
			核心包(name)、子包(name-subfunction)

		spec: 编写此文件

	rpm
		安装:
			X --> Y,Z

		rpm --> yum
			 YUM --> Yellowdog Update Modifier

		C/S架构：Client --> Server
			X --> Y,Z

			yum repository: yum仓库
				数据：
					各个rpm包；
				元数据：
					数据文件
						包名、版本信息、各包所包含的文件列表、依赖关系、包分组信息

						centos5: xml, centos6,7: sqlite

						createrepo: 制作yum仓库元数据的工具

			yum客户端：以安装为例
				第一步：获取仓库元数据
					缓存于本地：/var/cache/yum

				第二步：安装程序包
					yum客户端程序在本地分析元数据文件，并结合本地系统环境(已安装的包)做出要安装的程序包的决策
						X --> Y,Z,M,N

				第三步：获取程序包
					根据决策联系Yum仓库，下载各程序包缓存于本地后，一并进行安装；

			yum仓库：
				base库：通常为系统发行版所提供的程序包
				updates库：
				extra库：
				epel库：


		使用yum机制：
			1、确保有yum repo可用；
				rpm包的文件服务器，repodata目录所在父目录就是一个可用仓库；

				ftp     ftp://server/path/to/repository
				http    http://server/path/to/repository
				nfs 	nfs://server/nfs_path
				file    file:///path/to/repository
					file:///media/cdrom

			2、yum客户端
				指供repo配置文件，指明仓库访问路径及各种属性信息

				主配置文件（中心配置文件）：/etc/yum.conf
				一个或几个相关仓库的配置信息可保存为一个文件，文件名都以.repo结尾：/etc/yum.repos.d/

				在.repo文件定义一个yum repo指向的格式：
				[REPOID]
				name=Some name for this repository
				baseurl=file:///media/cdrom
						ftp://172.16.0.1/pub/ftp/centos

						 Must  be  a  URL  to  the  directory where the yum repository's 'repodata' directory lives
				enabled={0|1}
				gpgcheck={0|1}
				gpgkey=URL

				mirrorlist=URL to a file
					mirrorlist  Specifies a URL to a file containing a list of baseurls

				cost={1..n}
					默认为1000，指定访问此仓库的开销


		教室的repository:
			CentOS 6.5 x86_64的发行版
				http://172.16.0.1/cobbler/ks_mirror/centos-6.5-x86_64/

			CentOS 6.5的发行版：
				http://172.16.0.1/centos/6/extras/{i386,x86_64}

			fedora-epel
				http://172.16.0.1/fedora-epel/6/{i386,x86_64}

		yum客户端命令的使用：
			1、列出所有可用repo
				yum repolist {enabled|disabled|all}

			2、列出rpm包
				yum list {all|installed|available} 

				yum list KEYWORD*

			3、包的描述信息
				yum info package_name

			4、列出所有的包组信息
				yum grouplist

			5、显示包组的信息：例如组中包含的程序包列表
				yum groupinfo "GROUP NAME"

				CentOS6 跟开发相关的包组：
				Development Tools
				Server Platform Development
				Desktop Platform Development

			6、清理缓存
				yum clean {all|packages|metadata|expire-cache|rpmdb|plugins}

			7、安装程序包
				yum install package_name

				重新安装：
				yum reinstall package_name

			8、升级
				yum check-update: 检查可用的升级包

				yum update package_name

					x-1.3.1
					x-1.3.2, x-1.3.3, x-2.0.1

					yum update x-1.3.2

				yum downgrade package_name

			9、卸载
				yum remove|erase package_name

			10、查询某文件是由哪个包安装生成的
				yum whatprovides|provides /path/to/somefile

			11、安装包组
				yum groupinstall "GROUP NAME"

			12、卸载包组
				yum groupremove "GROUP NAME"

			

		假设：从其它处获得一个rpm包，如果此包依赖于其它包（在仓库中），如何安装？
			如果仅是单次安装需要：
				yum install /path/to/packe_file


	练习：安装开发环境包组，确保如下命令能执行
		# gcc --version
		gcc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-4)
		Copyright (C) 2010 Free Software Foundation, Inc.
		This is free software; see the source for copying conditions.  There is NO
		warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.


		baseurl=http://mirrors.sohu.com/centos/$releasever/os/$basearch

		centos5, test.repo
		centos6, test.repo






	yum配置文件中可用的四个变量：
		$releasever: 程序的版本，对Yum而言指的是redhat-release版本；只替换为主版本号，如RedHat 6.5，则替换为6; 
		$arch: 系统架构
		$basearch: 系统基本架构，如i686，i586等的基本架构为i386；
		$uuid: 
		$YUM0-9: 在系统中定义的环境变量，可以在yum中使用；

		获取当前系统相应变量替换结果的办法：
			# python
			Python 2.6.6 (r266:84292, Nov 22 2013, 12:16:22) 
			[GCC 4.4.7 20120313 (Red Hat 4.4.7-4)] on linux2
			Type "help", "copyright", "credits" or "license" for more information.
			>>> import yum,pprint
			>>> yb = yum.YumBase()
			>>> pprint.pprint(yb.conf.yumvar,width=1)
			Loaded plugins: fastestmirror, refresh-packagekit
			{'arch': 'ia32e',
			 'basearch': 'x86_64',
			 'releasever': '6',
			 'uuid': 'ea249181-2f82-4c40-8c42-d51b3fce319a'}
			>>> quit()

	yum支持插件

	如何自建yum仓库：
		1、如何自建基于光盘镜像的yum repo

		2、自建提供yum仓库的中心服务器
			ftp, http, nfs

			以http为例，步骤：
				(1) 安装httpd程序，并启动服务
					# rpm -ivh httpd-
					或者
					# yum install httpd

					启动服务
					# service httpd start
					# chkconfig httpd on

				(2) httpd的文档根目录为/var/www/html：
					创建子目录，存放某相关的所有rpm包
				
				(3) 为仓库生成元数据文件，以使能够作为仓库使用
					# rpm -ivh createrepo-
					或者
					# yum install createrepo	

					# createrepo /path/to/rpm_repo/	

				(4) 配置yum客户端使用此仓库即可



	编译安装源代码包：
		依赖于开发环境

		程序源码：
			c/c++
			perl
			python

		制作好的rpm包：编译过的
			20，
				主包：6
				分包1：2
				分包2：4

		开源应用程序：
			自建站点：
				apache, ASF
				mysql
				drbd
			代码托管：
				SourceForge
				github.com
				code.google.com

		编译工具：gcc
			gnu c complier
			gcc-c++

		源代码的组织格式：
			多文件：
				文件中的代码有依赖关系

			项目管理工具：
				GNU make(gcc)

				项目：50文件
					项目的制作者：利用make工具，为make提供一个配置文件

					autoconf： 生成编译环境检查及编译功能配置脚本
						生成configure
					automake: Makefile.in --> makefile

			编译源程序的步骤：
				# tar xf testapp-version.tar.{xz|bz2|gz}
				# cd testapp-version
				# ./configure
					还需通过许多选项指定编译特性
				# make
				# make install

			./configure脚本的使用：
				1、获取帮助
					./configure --help

				2、较通用的一些选项
					安装路径相关：
						--prefix=/path/to/somewhere: 指定安装路径
						--sysconfdir=/path/to/somewhere: 指定配置文件安装路径
					指定启用/禁用的特性
						--enable-FEATURE: 例如--enable-fpm
						--disable-FEATURE: 例如--disable-socket
					指定所依赖的功能、程序或文件
						--with-FUNCTION[=/path/to/somewhere]
						--without-FUNCTION

			安装后的配置：

			程序运行：
				1、让二进制程序直接，而无须输入路径
				# vim /etc/profile.d/APPNAME.sh
				export PATH=$PATH:/path/to/somewhere

				2、导出手册页：
				编辑/etc/man.config配置文件，添加一项MANPATH，路径为新安装的程序的man目录；
				
				# man -M /path/to/somewhere KEYWORD

			程序开发：如果其它应用程序依赖此程序的开发环境，或针对此程序做二次开发
				1、导出库文件
				第一步：指定让系统搜索定制的路径
					编辑/etc/ld.so.conf.d/APPNAME.conf
						一行一个库文件路径

				第二步：触发系统重新搜索所有的库文件并生成缓存
					# ldconfig 
						-v

				2、导出头文件
					/usr/local/nginx/include

					系统找头文件的路径是：/usr/include

					导出方式：创建链接进行
						ln -sv /usr/local/nginx/include /usr/include/nginx


	总结：yum, 源代码安装


回顾：
	程序安装：rpm、yum(C/S)、源代码编译

		rpm: 分包机制
			核心包、支包

		管理：打包、安装、卸载、查询、升级、校验、数据库管理、验正

		yum: yum + yum repo
			yum: yum command
			yum repo: file server (rpm包+元数据（repodata）)

			配置文件：/etc/yum.conf + /etc/yum.repos.d/*.repo
			[repoid]
			name=
			baseurl=URL
					url
					url
			enabled={0|1}
			gpgcheck={0|1}
			gpgkey=url
			cost=

		yum install x
			-y:
			--nogpgcheck 

		./configure, make, make install
			gcc, gcc-c++

		包：绿色软件，解压直接使用

进程管理：

	程序文件：静态文件
		指令+数据
			指令:CPU ，加工数据
				输出机制：

	运行中程序：进程
		CPU：（ring0: kernel, ring3: user program）
			mkdir --> system call (由内核执行代码)


	用户态，用户模式：--> 内核态、内核模式
		软中断：soft interrupt, si

	Linux: 多用户、多任务

		内存：分段、分页

		进程：进程+内核
			内存：4G，
				1G：内核
				3G：进程

			线性地址空间
			物理地址空间

			内核：精心编造一个数据结构
				tast structure：任务结构

			进程调度：
				程序，算法
					算法复杂度：
						O(1): 

					mmu: memory management unit

			进程分类：
				CPU-Bound: 
				I/O-Bound: 

		内核：进程调度、内存管理
			CFS

		进程分类：
			批处理进程
			交互式进程
			实时进程

		进程优先级：0-139
			实时优先级：0-99，数字越大，优先级越高
			静态优先级：100-139，数字越小，优先级越高
				nice值：-20, 19
					进程启动，默认nice值为0
			动态优先级：由内核维护，动态调整

		进程状态：
			运行态：running
			睡眠态：sleeping
				可中断睡眠：interruptable
				不可中断睡眠：等待外部满足之前无法继续运行, uninterruptable
			停止态：不会被内核调度并运行，stopped
			僵死态：zombie

		进程创建机制：每一个进程都是由其父进程fork()自身而来；

		进程间通信：IPC (InterProcess Communication)
			同一主机：
				signal
				shm：共享内存
				semerphor: 旗语

			不同主机：
				rpc: Remote Procedure Calling
				socket: 

		CPU虚拟化：时间片, timeslice
			5ms

			保存现场、恢复现场

			5ms, 2ms, 5ms
			1ms, 2ms, 1ms, 2ms

			抢占式多任务

		线程：比进程更小的可以被单独调度的单位；
			thread: lwp


	Linux进程管理工具：ps, pstree, pidof, top, htop, pmap, vmstat, dstat, kill, job, glance

	pstree: 查看进程树

	ps: 显示进程状态的命令，快照式、一次性
		支持两种风格：SysV, BSD

		进程：
			与终端相关的进程: a
			与终端无关的进程：x

			常用组合：ps aux
				VSZ: Virtual memory SiZe
				RSS: 常驻内存集

			STAT状态：
				R: running
				S: 可中断睡眠
				D：不可中断睡眠
				T：stopped
				Z：zombie

				s: session leader
				+：前台进程
				l: 多线程进程
				N：低优先级进程
				<: 高优先级进程

			COMMAND：包含在方括号中的进程表示为内核线程

			常用组合：ps -ef
				-e: 显示所有进程
				-f: 显示完整格式的信息

			常用组合：ps -eFH
				-F: 显示额外信息
				-H：显示进程的层次结构

			自定义要显示的信息：-o
				ps axo pid,command,psr,pri,ni
					ni: nice值
					pri: 优优级
					psr: 运行的CPU

			pgrep 
				-U UID：仅显示由指定用户启动的进程
				-G GID：仅显示与指定组相关的进程
				-t term...: 仅显示与指定终端相关的进程
				-l: 同时显示进程号和程序名

			pidof:显示指定命令所启动的进程的ID
				pidof COMMAND

		top: 
			M: 内存百分比
			P: CPU百分比
			T: 累积占用的CPU时间

			l: 显示或不显示负载信息
				过去1分钟、5分钟、15分钟的平均负载
					等待运行的进程队列的长度
			t: 显示或不显示进程及CPU相关的信息
				1: 数字，分别显示各CPU的相关信息
					us: user space
					sy: system
					ni: 
					id: 
					wa: wait io
					hi: hardware interrupt
					si: soft interrupt
					st: 
			m: 显示或不显示物理内存和交换内存的相关信息

			q: 退出
			k: 终止指定进程
			s: 修改刷新时间间隔

			常用选项：
				-d #: 指定刷新时间间隔
				-b: 以批次的方式显示top的刷新
				-n #: 显示的批次

		htop: 
			u: 交互式选择显示指定用户的进程
			l: 显示光标所在进程所打开的文件列表
			s: 显示光标所在进程执行的系统调用
			a: 绑定进程到指定的CPU
			#：快速定位光标至PID为#的进程上

			F1：获取帮助

		vmstat: 
			procs:
				r: 运行队列的长度
				b: 被阻塞（等待IO完成）队列的长度

			memory: 
				swpd: 从物理内存交换至swap中的数据量
				free: 空间物理内存
				buffer:
				cache：

			swap:
				si: swap in, 数据进入swap中的数据速率，kb/s
				so: swap out，数据离开swap中的数据速率

			io:
				bi: block in, 从块设备读入的数据速率，kb/s
				bo: block out，保存至块设备的数据速率

			system:
				in: interrupt, 中断速率
				cs: context switch, 进程切换速率

			cpu:
				us, sy, id, wa, st

			-s: 显示内存统计数据

	总结：
		ps, pstree, pgrep, pidof, top, htop, vmstat

		
进程管理：
	top,
		us%, sy%, wa%, ni%

		VSZ, RSS

	进程调度：
		CPU: timeslice, process

		CPU：
			O(1), CFS

			优先级：
				实时优先级：1-99
				静态优先级：100-139
				动态优先级：

		CPU：
			进程切换：上下文切换
				保存现场，恢复现场

		scale up: 向上扩展
		scale out: 向外扩展
			集群

	htop: a

	静态优先级：100-139
				-20, 19

				用户空间的程序：0, 120

	直接启动一个进程，并指定其Nice值：
		nice -n # COMMAND

	调整已运行的进程的nice值：
		renice -n # -p PID


	进程间通信：IPC
		同一主机：signal, shm, semerphor
		不同主机：socket, rpc

		信号：signal，借助于kill命令

		基于kill命令向其它的进程发信号：


		显示常用信号：
			# kill -l
			# man 7 signal

		每个信号都可以使用三种方式之一在Kill进行调用：
			数字代称: 1, 2, 9, 15
			信号完整名称：SIGHUP, SIGINT, SIGKILL, SIGTERM
			信号简称：HUP, INT, KILL, TERM

			1) SIGHUP：让程序重读配置文件需无须重新启动；
			2) SIGINT：interrupt，打断正在运行中的程序；
			9) SIGKILL：
			15) SIGTERM：

			kill [-SIGNAL] PID
				-9, -SIGKILL

			killall [-SIGNAL] COMMAND

	Linux作业控制：
		job

		前台作业：通过终端启动，并在终止之前一直占据着终端
		后台作业：作业启动之后即运行于后台，释放前台

		交互式模式：手动启动的非守护进程类的程序，一般都运行于前台；

		如何将作业运行于后台：
			1、运行中的作业：
				Ctrl+z
					送往后台后，作业处于STOPPED状态
			2、尚未启动作业：
				COMMAND &

				此类由手动方式控制的作业，与终端相关作业会被终止；如果把作业送往后台，且与终端无关：
					# nohup COMMAND &

		作业：作业号
			# jobs

		作业控制命令：
			# fg [[%]JOBNUM]: 将指定的作业调回前台
			# bg [[%]JOBNUM]: 让送往后台的作业在后台继续运行
			# kill %JOBNUM: 终止指定的作业

	进程查看：htop
		uptime, vmstat, iostat, netstat, ifstat, nfsstat

		dstat
			-c: 显示CPU统计数据
			-d: 显示disk统计数据
			-D DISK: 只显示指定disk的统计数据
			-g: 显示page的统计数据
			-i: 显示中断的统计数据
			-m: 显示内存的统计数据
			-l: 显示系统负载的统计数据
			-n: 显示网络接口相关
			-N INTERFACE: 仅显示指定的网络接口的数据
			-s: 显示交换内存
			-p: 进程队列
			--ipc: 显示ipc消息队列、信号量和共享内存的使用状况
			-y: 系统状态数据

			默认相当于使用“dstat -cdngy”，也相当于“dstat -a”

			此外，使用"dstat -f", 以完整格式显示所有信息，

			再者，使用"dstat -v"，显示结果类似于vmstat命令

			网络连接状态统计：
				--tcp
				--udp
				--raw
				--unix

	/proc/#/maps:
		进程内存映射表

		pmap命令：显示指定进程的物理内存空间映射表
		pmap PID

	glances: 
		由epel源所提供

	课外作业：nmap, netcat, tcpdump, nethogs, iftop

	命令总结：nice, renice, jobs, bg, fg, nohup, kill, dstat, uptime, pmap, glances


bash编程之case语句

	写一个脚本，使用格式：
		script.sh {start|stop|restart|status}

		1) start: 创建/var/lock/subsys/script.sh
		2) stop: 删除此文件
		3) restart: 先删除文件，再创建文件
		4) status: 如文件存在，显示running，否则，显示stopped

			#!/bin/bash
			#
			srv=`basename $0`

			lockFile="/var/lock/subsys/$srv"

			[ $# -lt 1 ] && echo "Usage: $srv {start|stop|restart|status}" && exit 4

			[ $UID -ne 0 ] && echo "Only root." && exit 5

			if [ "$1" == 'start' ]; then
			   if [ -f $lockFile ]; then
			        echo "$srv is running."
			        exit 7
			   else
			        touch $lockFile
			        [ $? -eq 0 ] && echo "Starting $srv OK."
			   fi
			elif [ "$1" == 'stop' ]; then
			    if [ -f $lockFile ]; then
			        rm -f $lockFile
			        [ $? -eq 0 ] && echo "Stopping $srv OK."
			    else
			        echo "$srv is stopped yes."
			        exit 6
			    fi
			elif [ "$1" == 'restart' ]; then
			    if [ -f $lockFile ];then
			        rm -f $lockFile
			        [ $? -eq 0 ] && echo "Stopping $srv OK."
			    else
			        echo "$srv is stopped yet."
			    fi
			    touch $lockFile
			    [ $? -eq 0 ] && echo "Starting $srv OK."
			elif [ "$1" == 'status' ];then
			    if [ -f $lockFile ];then
			        echo "$srv is running."
			    else
			        echo "$srv is stopped."
			    fi
			else
			    echo "Usage: $srv {start|stop|restart|status}" && exit 9
			fi



	条件测试：
		[ expression ]
		[[ expression ]]
		test expression
		COMMAND: 不能使用命令引用，要引用的命令执行后状态返回值
		[ `COMMAND` -eq 0 ]

		expression:
			整数测试：-eq, -ne, -ge, -le, -gt, -lt
			字符串测试: ==, !=, >, <, =~, -z, -n
				注意：字串比较时，尽量使用引号；
			文件测试：-e, -f, -d, -L, -b, -c, -S, -p, -r, -w, -x, -s

		多分支的if语句：
			if 条件1; then
				分支1
			elif 条件2; then
				分支2
			...
			else
				分支n
			fi


	case语句：有多个测试条件时，case语句会使得语法结构更明晰

		case 变量引用 in
		PATTERN1)
			分支1
			;;
		PATTERN2)
			分支2
			;;
		...
		*)
			分支n
			;;
		esac

		PATTERN：类同于文件名通配机制，但支持使用|表示或者；
			a|b: a或者b
			*：匹配任意长度的任意字符
			?: 匹配任意单个字符
			[]: 指定范围内的任意单个字符

	例如：用户键入字符后判断其所属的类别；

		#!/bin/bash
		#

		read -p "Plz enter a char: " char

		case $char in
		[0-9])
		    echo "a digit"
		    ;;
		[a-z])
		    echo "a char"
		    ;;
		*)
		    echo "a special word"
		    ;;
		esac


	写一个脚本，使用格式：
		script.sh {start|stop|restart|status}

		1) start: 创建/var/lock/subsys/script.sh
		2) stop: 删除此文件
		3) restart: 先删除文件，再创建文件
		4) status: 如文件存在，显示running，否则，显示stopped

			#!/bin/bash
			#
			srv=`basename $0`

			lockFile="/var/lock/subsys/$srv"

			[ $# -lt 1 ] && echo "Usage: $srv {start|stop|restart|status}" && exit 4

			[ $UID -ne 0 ] && echo "Only root." && exit 5

			case $1 in
			start)
			   if [ -f $lockFile ]; then
			        echo "$srv is running."
				exit 7
			   else
				touch $lockFile
				[ $? -eq 0 ] && echo "Starting $srv OK."
			   fi
			   ;;
			stop)
			    if [ -f $lockFile ]; then
				rm -f $lockFile
				[ $? -eq 0 ] && echo "Stopping $srv OK."
			    else
				echo "$srv is stopped yes."
				exit 6
			    fi
			    ;;
			restart)
			    if [ -f $lockFile ];then
			     	rm -f $lockFile
				[ $? -eq 0 ] && echo "Stopping $srv OK."
			    else
				echo "$srv is stopped yet."
			    fi
			    touch $lockFile
			    [ $? -eq 0 ] && echo "Starting $srv OK."
			    ;;
			status)
			    if [ -f $lockFile ];then
				echo "$srv is running."
			    else
				echo "$srv is stopped."
			    fi
			    ;;
			*)
			    echo "Usage: $srv {start|stop|restart|status}" && exit 9
			esac

	练习：写一个脚本，对/etc/目录及内部的所有文件打包压缩
	1、显示一个菜单，让用选择使用的压缩工具：
		xz) xz compress tool
		gz) gzip compress tool
		bz2) bzip2 compress tool
	2、根据用户选择的工具，对/etc执行相应的操作并保存至/backups目录，文件形如/backups/etc-日期时间.tar.压缩后缀

		#!/bin/bash
		#
		[ -d /backups ] || mkdir /backups

		cat << EOF
			xz) xz compress tool
			gz) gzip compress tool
			bz2) bzip2 compress tool
		EOF

		read -p "Plz choose an option: " choice

		case $choice in
		xz)
		    tar -Jcf /backups/etc-`date +%F-%H-%M-%S`.tar.xz /etc/ *
		    ;;
		gz)
		    tar -zcf /backups/etc-`date +%F-%H-%M-%S`.tar.gz /etc/ *
		    ;;
		bz2)
		    tar -jcf /backups/etc-`date +%F-%H-%M-%S`.tar.bz2 /etc/ *
		    ;;
		*)
		    echo "wrong option."
		    exit 1
		    ;;
		esac



	练习：写一个脚本，使用形式如下所示
		showifinfo.sh [-i INTERFACE|-a] [-v]
		要求：
		1、-i或-a不可同时使用，-i用于指定特定网卡接口，-a用于指定所有接口；
			显示接口的ip地址
		2、使用-v，则表示显示详细信息
			显示接口的ip地址、子网掩码、广播地址；
		3、默认表示仅使用-a选项;

			#!/bin/bash
			#
			verbose=0
			allInterface=0
			ifflag=0
			interface=0

			while [ $# -ge 1 ]; do
			    case $1 in
			    -a)
				allInterface=1
			        shift 1
				;;
			     -i)
				ifflag=1
				interface="$2"
			 	shift 2
				;;
			     -v)
				verbose=1
				shift
				;;
			     *)
				echo "wrong option"
				exit 2
				;;
			     esac
			done

			if [ $allInterface -eq 1 ]; then
			    if [ $verbose -eq 1 ]; then
				ifconfig | grep "inet addr:"
			    else
				ifconfig | grep "inet addr:" | awk '{print $2}' 
			    fi
			fi

			if [ $ifflag -eq 1 ]; then
			    if [ $verbose -eq 1 ]; then
				ifconfig $interface | grep "inet addr:"
			    else
				ifconfig $interface | grep "inet addr:" | awk '{print $2}' 
			    fi
			fi		




Linux内核及系统启动流程

	kernel的功能：
		进程管理
		文件系统
		硬件驱动
		内存管理
		安全功能：SELinux
		网络子系统


		标准库：glibc

		调用：返回
			利用别的组件的功能，完成某特定事务
			返回值

	内核设计流派：
		单内核体系：
			Linux
				支持模块化
				模块还可以动态装载或卸载

			Linux内核：核心 + 外围模块
				核心：/boot/vmlinux-VERSION-release
				模块：/lib/modules/VERSION-release
					.ko: kernel object
				ramdisk: /boot/initramfs-VERSION-release.img
					在内核启动过程中装载根文件系统时有用；

		微内核体系：
			Windows
			Solaris

	Linux系统启动流程：X86（PC）

		POST：加电自检
			ROM+RAM
				ROM：CMOS （BIOS）
				RAM：
						EFI（）

			引导次序：按次序找引导设备，第一个有引导程序的设备即为启动PC server所用到的设备；
				1st boot: 
				2nd boot:
				3rd boot:

		MBR：Master Boot Record
			1st sector：512bytes
				446: bootloader
				64: partation table
				2: 5A

		bootloader: 选择要启动的内核（在当前磁盘的某或某些分区上）

			LILO: LInux LOader
				0-1023范围内的柱面构成的分区的内核文件
			GRUB: GRand Unified Bootloader
				CentOS 5,6 Grub 0.97
				CentOS 7, Grub2 1.96

		kernel: 
			自身初始化
				探测所有能识别的硬件
				装载驱动程序
				以只读方式装载根文件系统（rootfs）
				/sbin/init

			CentOS 5: SysV, init
			CentOS 6: upstart, init
			CentOS 7: systemd, init

			CentOS 5 SysV, init, /etc/inittab --> /etc/rc.d/rc.sysinit
			CentOS 6 Upstart, /etc/init/ *.conf （/etc/inittab）--> /etc/rc.d/rc.sysinit
				chkconfig
			CentOS 7 Systemd, 
				/usr/lib/systemd/system/

		CentOS 5: init
			kernel ==> /sbin/init (/etc/inittab) 
				设定系统默认运行级别
					0-6: 7个级别
						0：关机
						1: single user mode, s, S, single
						2: multi user mode, 不支持NFS功能
						3：完全多用户模式，文本接口
						4：未使用，预留级别
						5：完全多用户械，图形接口
						6: 重启

					切换级别：init #

				进一步初始化系统
					/etc/rc.d/rc.sysinit

				启动指定的默认级别的默认为启动的服务，停止指定的级别下默认为关闭的服务；
					/etc/rc.d/rc#.d
						S##: 启动的服务
						K##：停止的服务

						##：01-99，数字越小，越优先启动或关闭；

					脚本如果期望能够被chkconfig命令使用，要在脚本中添加如下行
						# chkconfig: - 85 15
							-：当此脚本由chkconfig控制时，默认哪些级别就是开启的，
							85：启动优先级
							15: 关闭优先级

						# chkconfig --add SRV_SCRIPT
						# chkconfig --del SRV_SCRIPT
						# chkconfig SRV_SCRIPT {on|off}
							默认为2345级别；
							--level ######

						/etc/rc.d/rc.local (/etc/rc.local): 是一个脚本，通常为系统启动完成的最后运行一个脚本；

					定义一些组合键的功能：通常是Ctrl+Alt+Delete

					初始化字符终端

					如果有需要，启动图形终端

			CentOS 5: /etc/inittab
				每一行定义一种action和对应的程序；
					id:runlevels:action:process
					例如
					si::sysinit:/etc/rc.d/rc.sysinit

					action: respwan, wait, initdefault
					l3:3:wait:/etc/rc.d/rc 3

			CentOS 6: /etc/inittab
				由/etc/init/ *.conf 一类的配置文件定义init的初始化动作
					由upstart调用，程序为/sbin/init

	/etc/rc.d/rc.sysinit: 系统初始化脚本
		设定主机名：读取/etc/sysconfig/network文件中的HOSTNAME参数，并以之设定主机名
		打印文本欢迎信息
		激活SELinux和udev
		挂载/etc/fstab文件中定义的其它文件系统
		激活swap
		检测根文件系统，并以读写方式重新挂载；
		设置系统时钟
		根据/etc/sysctl.conf设置内核参数
		激活LVM和RAID设备；
		加载额外设备的驱动程序；
		清理操作

	初始化流程：POST --> （BIOS）boot sequence --> MBR(bootloader) --> kernel + ramdisk(initrd, initramfs) --> mount rootfs (ro) --> /sbin/init (CentOS 5: /etc/inittab, CentOS6 /etc/init/*.conf) 
		设定默认运行级别 --> 使用/etc/rc.d/rc.sysinit初始化系统 --> 分别启动并关闭指定服务 --> Ctrl+Alt+Delete组合键 --> 启动字符终端 --> 启动图形终端


	GRUB: GRand Unified Bootloader
		grub程序由两段组成：
			stage1: MBR (0柱面 0磁道 1扇区)
			stage1_5: MBR随后的扇区
			stage2: 读取grub.conf配置文件，并实现引导功能的扩展

		grub的功能：
			1、提供菜单，并提供交互式接口
				e: 进入编辑模式
			2、选择要启动的内核或系统
				允许传递引导参数给内核
				选择界面可隐藏
			3、为编辑功能提供保护机制
				启用内核文件：
					选择运行指定的内核得先输入密码
				传递参数：
					使用e命令得先输入密码

		grub命令行接口：
			root：指定哪个分区为接下来要启动的系统或内核文件所在的分区
				root （DEVICE）
					所有硬盘都识别为hd
					不同的硬盘基于数字标识：如hd0, hd1等
					同一个硬盘上的不同分区，也使用数字标识，如hd0,0  hd1,5

					root (hd0,0)

				find (DEVICE)/path/to/file

				kernel: 指定要运行的内核文件
				initrd: 为要运行的内核指定其可用的ramdisk文件

				boot: 启动此前配置好的内核或系统

		grub.conf：
			参数：
				default=: 选择第几个title配置的内核或系统，各title从0开始编号
				timeout=#: 菜单显示的超时时长；
				splashimage=/path/to/some_image_file：指定菜单的背景图片；此图片只能为14bits色，xpm格式，gzip压缩；
				hiddenmenu: 隐藏菜单
				title TILTE STRING：显示于菜单中的标题
					root
					kernel
					initrd

		grub保护机制：
			1、生成密码：
				# grub-md5-crypt
			2、保护编辑功能，则需要title之外的添加
				password --md5 密码串
			3、保护使用某内核，则需要内核对应的title之下添加
				password --md5 密码串

		安装grub的方式：
			1、使用grub-install命令
				# grub-install [--root-directory=/path/to/somewhere] DEVICE

				--root-directory=/path/to/somewhere
					/path/to/somewhere：内核及initrd文件所在的分区的挂载点的父目录，且此挂载点必须叫boot
						例如：/dev/sdb1: /mnt/boot
						# grub-install --root-directory=/mnt /dev/sdb



回顾：启动流程
	POST --> 引导次序（BIOS）--> BootLoader(MBR) --> kernel + ramdisk (临时根) --> 根切换 (rootfs) --> /sbin/init (配置文件)

	配置文件：设置默认运行级别 --> 指定系统初始化脚本进行系统初始化 --> 启动服务（关闭服务）(/etc/rc.d/rc#.d, /etc/rc.d/init.d/) --> /etc/rc.d/rc.local -> 设置CtrlAltDel组合的功用 --> 启动终端（mingetty），并在终端附加登录程序(login) --> 如果级别为5, 则要启动 X server

	login: root
	password: mageedu

	nsswitch: 检查用户帐是否存在，并且如果存在，还能将其解析为UID；
		库（API），而非服务
		/usr/lib64/libnss*
		/lib64/libnss*
	pam: pluggable authentication module, 做用户认证
		库 (API)
		/lib64/security/ *


	ramdisk
		centos5: initrd
			ram disk: 块设备
		centos6: initramfs
			ram fs:

	配置文件：
		c5: sysv init, /etc/inittab
		c6: upstart, /etc/inittab + /etc/init/ *.conf
			chkconfig --add, --del, --list, --level
		c7: systemd, /usr/lib/systemd/

	X-Window: Client --> Server
		X protocol
		twm
		Desktop: GNome, KDE, Xfce

	GRUB: GRand Unified Bootloader
		stage1
		stage1_5
		stage2

		命令行接口：
		grub>

		配置文件：
		/boot/grub/grub.conf

		编辑模式：

	Linux内核：单内核，模块化
		内核的某些功能：
			编译进内核本体  [*]
			编译成内核模块  [M]
			不选择使用	    [ ]

		内核的组成部分：
			/boot/vmlinuz-VERSION
			/lib/modules/VERSION/
				*.ko: kernel object
				模块间有可能存在依赖关系；

		内核模块管理：
			lsmod: 显示内核已装载模块

			动态装卸载模块：
				卸载：modprobe -r MOD_NAME
				装载：modprobe MOD_NAME

				装载：insmod /path/to/module_file
				卸载：rmmod MOD_NAME

			查看某模块的详细信息：
				modinfo MOD_NAME

			检查并生成模块间依赖关系的命令：
				depmod


bash编程函数：
	
	模块化，代码重用

	代码块，名

	语法：

	function F_NAME {
		函数体
	}

	F_NAME() {
		函数体
	}

	可调用：使用函数名
		函数名出现的地方，会被自动替换为函数；

	脚本：

	函数的返回值：
		函数的执行结果返回值：代码的输出
			函数中的打印语句：echo, print
			函数中调用的系统命令执行后返回的结果
		执行状态返回值：
			函数体中最后一次执行的命令状态结果
			自定函数执行状态的返回值：return #

	函数可以接受参数：
		在函数体中调用函数参数的方式同脚本中调用脚本参数的方式：位置参数
			$1, $2, ...
			$#, $*, $@


	示例：服务脚本示例

		#!/bin/bash
		#
		# chkconfig: 2345 67 34
		#
		srvName=$(basename $0)

		lockFile=/var/lock/subsys/$srvName

		start() {
		    if [ -f $lockFile ];then
			echo "$srvName is already running."
			return 1
		    else
			touch $lockFile
			[ $? -eq 0 ] && echo "Starting $srvName OK."
			return 0
		     fi
		}

		stop() {
		    if [ -f $lockFile ];then
			rm -f $lockFile &> /dev/null
			[ $? -eq 0 ] && echo "Stop $srvName OK" && return 0 
		    else
			echo "$srvName is not started."
			return 1
		    fi
		}	

		status() {
		    if [ -f $lockFile ]; then
			echo "$srvName is running."
		    else
			echo "$srvName is stopped."
		    fi
		    return 0
		}

		usage() {
		     echo "Usage: $srvName {start|stop|restart|status}"
		     return 0
		}

		case $1 in
		start)
			start
			;;
		stop)
			stop ;;
		restart)
			stop
			start ;;
		status)
			status ;;
		*)
			usage
			exit 1 ;;
		esac





练习：写一个脚本，完成如下功能(使用函数)：
1、提示用户输入一个可执行命令；
2、获取这个命令所依赖的所有库文件(使用ldd命令)；
3、复制命令至/mnt/sysroot/对应的目录中
	解释：假设，如果复制的是cat命令，其可执行程序的路径是/bin/cat，那么就要将/bin/cat复制到/mnt/sysroot/bin/目录中，如果复制的是useradd命令，而useradd的可执行文件路径为/usr/sbin/useradd，那么就要将其复制到/mnt/sysroot/usr/sbin/目录中；
4、复制各库文件至/mnt/sysroot/对应的目录中；


#!/bin/bash
#
target=/mnt/sysroot/

[ -d $target ] || mkdir $target

preCommand() {
    if which $1 &> /dev/null; then
	commandPath=`which --skip-alias $1`
	return 0
    else
	echo "No such command."
	return 1
    fi
}

commandCopy() {
    commandDir=`dirname $1`
    [ -d ${target}${commandDir} ] || mkdir -p ${target}${commandDir}
    [ -f ${target}${commandPath} ] || cp $1 ${target}${commandDir}
}

libCopy() {
    for lib in `ldd $1 | egrep -o "/[^[:space:]]+"`; do
	libDir=`dirname $lib`
	[ -d ${target}${libDir} ] || mkdir -p ${target}${libDir}
	[ -f ${target}${lib} ] || cp $lib ${target}${libDir}
    done
} 

read -p "Plz enter a command: " command

until [ "$command" == 'quit' ]; do

  if preCommand $command &> /dev/null; then
    commandCopy $commandPath
    libCopy $commandPath
  fi

  read -p "Plz enter a command: " command
done




练习：
	写一个脚本，判定192.168.0.0网络内有哪些主机在线，在线的用绿色显示；不在线的用红色显示；（使用函数）
	写一个脚本，判定172.16.0.0网络内有哪些主机在线，在线的用绿色显示；不在线的用红色显示；（使用函数）











练习：写一个脚本，完成如下功能(使用函数)：
1、脚本使用格式：
mkscript.sh [-D|--description "script description"] [-A|--author "script author"] /path/to/somefile
2、如果文件事先不存在，则创建；且前几行内容如下所示：
#!/bin/bash
# Description: script description
# Author: script author
#
3、如果事先存在，但不空，且第一行不是“#!/bin/bash”，则提示错误并退出；如果第一行是“#!/bin/bash”，则使用vim打开脚本；把光标直接定位至最后一行
4、打开脚本后关闭时判断脚本是否有语法错误
	如果有，提示输入y继续编辑，输入n放弃并退出；
	如果没有，则给此文件以执行权限；



练习：提示用户输入一个用户名，判断用户是否登录了当前系统; 
	如果没有登录，则停止5秒钟之后，再次判断；直到用户登录系统，显示“用户来了”，而后退出；



练习：写一个脚本，
	1、提示用户输入一个磁盘设备的设备文件，如果设备文件不存在，就提示用户重新输入，直到用户输入正确为止；
	2、用户可以输入quit退出；

练习：扩展前一题
	当用户给出正确的块设备后：
	1、显示用户输入块设备，并提示用户，后续的操作会损坏设备上的所有文件，让用户选择是否继续
	2、如果用户输入y，则继续后面的操作；
	3、如果用户输入n，则显示用户选择了中止，并退出脚本；
	4、输入任何其它字符，则让用户重新选择；

练习：扩展上一题
	1、如果用户选择了y, 则抹除指定块设备上的所有分区；

练习：再扩展
	1、在上面的磁盘创建两个主分区：
		(1) 50M
		(2) 512M
	2、均格式化为ext4文件系统；
	3、分别挂载至/mnt/boot和/mnt/sysroot

练习：继续扩展
	1、在此设备上安装grub；
	2、在/mnt/sysroot目录下创建根文件系统所需要各目录；

练习：扩展
	1、移植多个应用程序，至少包含bash、ifconfig等；







回顾：
	系统启动流程，内核模块加载及卸载的方法；bash函数的使用；

	写一个脚本：让用户输入一个网络地址，探测此网络内的所有主机是否在线；
		192.168.1.1-254
		172.16.0.1-172.16.255.254
		10.0.0.1-10.255.255.254


内核参数的配置：
	
	/proc, /sys

	/proc: 内核映像
		许多参数（读写，只读）
			只读文件：输出统计信息
			读写文件：设定内核工作特性
				/proc/sys

				不允许使用文本编辑器打开进行编写，而只能使用重定向的方式或使用专用的工具

			vm, net, fs

			net.ipv4.ip_forward

		/proc: 虚拟成了文件系统
			net/ipv4/ip_forward

	几个常用参数：
		kernel.hostname
		vm.drop_caches
		net.ipv4.icmp_echo_ignore_all
		net.ipv4.ip_forward

	Linux: ip地址属于内核，而非网卡

	修改内核参数的办法：
		echo "" > /proc/sys/

		sysctl -w VARAIABLE=VALUE

			配置文件：/etc/sysctl.conf

		sysctl -a: 显示sysctl可控制的所有内核参数；
		sysctl -p: 重读配置文件并生效之；

	/sys: 

		Linux 2.4-: /dev所有设备都是事先预置；
		Linux 2.6+: /dev下所有设备文件能够按需创建
			kernel初始化时，根文件系统尚未挂载
			/sys: 硬件设备的相关信息
			用户空间的某应用程序就可根据/sys中信息来为每个设备按需创建设备文件

			udev: 用户空间的程序，用于创建所需要设备， udevadmin,
				hotplug

			udev创建设备文件是根据udev规则进行，规则文件在/etc/udev/rules.d/


	ramdisk: /boot/initramfs-VERSION.img

		创建工具：mkinitrd
				  dracut

		uname:
			-n, -r

		# dracut /boot/initramfs-$(uname -r).img  $(uname -r)
		# dracut /boot/initramfs-3.10.1.img   3.10.1

		cpio, gzip

		展开initramfs文件：
		# cp /boot/initramfs-RELEASE.img /tmp/initramfs.img.gz
		# gzip -d /tmp/initramfs.img.gz
		# mkdir /tmp/initramfs
		# cd /tmp/initramfs
		# cpio -id < /tmp/initramfs.img.gz


bash子进程：
	
	exec COMMAND: 能启动command为一个进程，此进程会取代当前shell进程；


screen工具：
	
	启动新屏幕：screen
	退出新屏幕：exit, 意味着关闭
	拆除新屏幕：Ctrl+a, d    意味着临时隐藏

	screen -ls: 查看所有被隐藏的屏幕ID
	screen -r SID: 连接至某隐藏的屏幕


lftp: ftp客户端工具
	lftp HOST
		-u USERNAME,PASSWORD
		-p PORT

		-e 'CMD'

	lftp: 子命令

		help： 获取帮助
		cd: 切换远程服务器上的文件系统目录
		lcd: 切换本地文件系统目录
		get FILE: 下载

		!COMMAND: 执行shell命令，而非ftp命令

		mget FILE1...: 下载多个文件，支持使用通配符

		mirror DIR： 镜像目录至本地 

		put FILE: 
		mput FILE1...:


	lftpget类似于wget: 下载指定URL



kernel:
	www.kernel.org

	版本号：
		RHEL6: 
			6.0: 2.6.32
			6.1
			.
			6.5：2.6.32

		RHEL7：


		3.10： f1, f2

	前提：查看本地硬件信息常用工具

		查看CPU信息：
			# cat /proc/cpuinfo

			# x86info

				# yum install x86info

			# lscpu

		查看PCI：
			# lspci
				-v

		查看USB：
			# lsusb

		查看块设备：
			# lsblk

	编译：交叉编译
		cross-compiling

	编译内核的步骤：(安装好开发环境，c6: Development Tools, Server Platform Development, ncurses-devel)
		第一步：配置内核
			make config
			make allyesconfig
			make allnoconfig

			make menuconfig
			make gconfig （依赖GNome桌面环境及GNome的图形开发环境，gtk2）
			make kconfig （依赖KDE桌面环境及KDE的图形开发环境，qt）

		第二步：编译
			make

		第三步：安装内核模块
			make modules_install

			安装位置：/lib/modules/VERSION/
				分析模块间依赖关系并成dep文件

		第四步：安装内核
			make install

				安装内核：/boot/vmlinuz-VERSION
				编辑grub.conf，添加一新的title，


	获取源代码，展开指定目录下：
		# tar xf linux-3.10.10.tar.xz -C /usr/src


	后续的编译，开始之前的清理操作
		# make clean
			清理编译的文件，但保留配置文件
		# make mrproper
			移除所有编译生成的文件、配置文件和备份文件
		# make distclean
			完全清理

	1、将编译生成的文件保存至别处：
		# mkdir /path/to/somewhere
		# cd /path/to/somewhere
		# ./configure --ksource=/usr/src/linux

	2、如何只编译内核的部分代码
		(1) 只编译某子目录中的相关代码
		# cd /usr/src/linux
		# make  path/to/dir/

		例如：
		# make SUBDIR=arch/
		# make drivers/net/

		(2) 只编译部分模块
		# make M=path/to/dir

		# make M=drivers/net/

		(3) 只编译一个模块
		# make path/to/dir/MOD_NAME.ko

		例如
		# make drivers/net/ethernet/intel/e1000/e1000.ko

		(4)将编译生成的文件保存至别处：
		# make O=/path/to/somewhere

	3、交叉编译

		# make ARCH=arch

		获取某ARCH的可用的默认配置
		# make ARCH=arch help

		例如：
		# make ARCH=arm acs5k_defconfig

	



回顾：
	
	内核管理：功能、系统启动流程、查看硬件信息、简单内核定制

	系统启动流程：
		POST --> BootLoader(MBR) --> Kernel (ramdisk) --> rootfs (/sbin/init)

		内核模块：/lib/modules/VERSION/

		可做为启动设备：
			光驱、便携式移动设备、硬盘、网卡（系统引导，PXE）

		BootLoader: GRUB
			GRUB 0.97, GRUB Legacy
			GRUB 1.96, GRUB 2

		kernel (ramdisk): 
			安装程序: anaconda 

			PXE: DHCP, tftp(kernel+ramdisk)

		/sbin/init
			C5:
				/sbin/init (SysV,)
				echo $$
			C6: 
				/sbin/init (upstart), D-BUS
					/etc/rc.d/init.d/ *
					service

				运行级别：0-6
					默认: 3, 5

					/etc/inittab

			C7:
				/sbin/init (systemd)
					systemctl 

Linux系统安装：
	
	安装过程：
		POST --> Bootloader (kernel+ramdisk) --> anaconda

		BootLoader界面：
			GUI界面
			text界面
				boot:

		anaconda接口：
			text接口
			GUI接口

		安装过程分为两个阶段：

			安装前的配置阶段：  （既可交互式进行，亦可直接读取配置文件自动完成）
				键盘类型
				安装过程中的语言
				支持使用语言
				时区
				选择要使用磁盘设备
				分区、格式化配置
				选择要安装的包
				管理员密码
			安装阶段：
				在目标磁盘创建分区、执行分区格式化
				将选定的程序包安装至目标磁盘
				安装bootloader
			第一次启动配置：
				iptables
				selinux
				core kdump

		kickstart文件：文本文件

		建议单独分区：
			/
			/home
			/usr
			/var
			swap
		不能单独分区：
			/proc, /sys, /etc, /bin, /sbin, /lib, /media, /mnt, /dev

		/boot: 
			单独分区与否，取决于rootfs所在设备的类型

	boot: 
		启动安装过程的引导参数

			CentOS 6:
				text: 文本安装界面
				repo=
					http://server/path/to/repo/
					ftp://username:password@server/path/to/repo
				网络配置：
					ip=
					netmask=
					gateway=
					dns=
					ifname=: 指定此地址配置到地的网络接口
				指定使用的kickstart文件及其位置
					ks=cdrom:/path/to/ksfile
					ks=http://server/path/to/ksfile
					ks=ftp://username:password@server/path/to/ksfile

					例如：http://172.16.0.1/centos6.x86_64.cfg
				如果额外加载驱动程序：
					dd

			CentOS 7:
				指定安装源：
					inst.repo=

				指定额外需要加载驱动：
					inst.dd=

				指定kickstart文件及其位置：
					inst.ks=

				指定使用TUI界面：
					inst.text
					inst.cmdline: 必须与kickstart文件一同使用

				网络功能选项：
					ip=method
						可用method: dhcp, dhcp6, auto6
					ip=interface:method
					ip=IP::GATEWAY:NETMASK:HOSTNAME:INTERFACE:none
					nameserver=


	kickstart文件：
		命令段
		软件包段
			%packages
				pack_name
				@group
				-pack_name: 不安装的包，但如果被依赖，也会被安装
		脚本段
			%pre
				安装前脚本
			%post
				安装后脚本

		图形配置接口：
			# yum install system-config-kickstart

		配置命令：
			# system-config-kickstart

		配置完成后的语法检查命令：
			# ksvalidator /path/to/ks_file


	基于isolinux目录创建引导盘：
	# mkisofs -R -J -T -v --no-emul-boot --boot-load-size 4 --boot-info-table -V "CentOS 6.5 x86_64 boot" -b isolinux/isolinux.bin -c isolinux/boot.cat -o /root/boot.iso myiso/


回顾：Linux, C6, C7
	isolinux, anaconda, kickstart
	system-config-kickstart, ksvalidator

	批量部署：
		物理机：PXE
			负载均衡
		虚拟机：映像文件模板
			分布式文件系统

PXE + kickstart文件

	DHCP，WEB，tftp

	DHCP：

		DHCP：Dynamic Host Configuration Protocol
			ip/netmask
			gateway
			name server

		bootp

		C/S架构：Server, Client
			dhclient/dhcpd

			Client: DHCP DISCOVER
			Server: DHCP OFFER
			Client: DHCP REQUEST
			Server: DHCP ACK

			广播：

			租约：
				2hours

				DHCP REQUEST
				DHCP ACK

				1hour --> 2hours

				1hour --> 1.5hours --> 1.75hours --> 1.875hours

			DHCP DISCOVER

		DHCP Relay: 中继

	安装：

	# yum install dhcp

	C6:
		服务脚本：/etc/rc.d/init.d/

		dhcpd, 67/udp
		dhclient, 68/udp

		/var/lib/dhcpd/dhcpd.leases

		service dhcpd configtest
			配置文件语法测试

	保留地址：专用于某特定客户端的地址，不应该使用地址池中的地址；
		优先于地址池中的地址；


CentOS 7: 服务控制

		# systemctl is-enabled DAEMON.service
		# systemctl enable DAEMON.service
		# systemctl disable DAEMON.service

		# systemctl {start|stop|restart|status} DAEMON.service


总结：
	dhcpd.conf
		option domain-name
		option domain-name-servers
		option routers

		subnet NETWORK netmask MASK {
			*range* START_IP END_IP;

			host HOSTID {
				hardware ethernet 00:11:22:33:44:55;
				fixed-address IP;
			}
		}



tftp: 
	C/S
		Server: 69/udp
			listening:
				accept():  

				socket: 
					ip:port
						172.16.100.7:69

		Client: 
			connetc()

			使用大于1023的一个其它进程未注册使用的随机端口

		session:
			ip:port  -> ip:port

守护进程：
	独立守护进程：standalone
		/etc/rc.d/init.d/ *	
	xinetd: 超级守护进程
		瞬时守护进程：
			/etc/xinetd.d/ *
				配置启动：
					(1) chkconfig SERVICE_NAME on
					(2) 编辑配置文件，确保没有被禁用
						disable = no

		修改后的生效需要重启超级守护进程：
		# service xinetd restart


	DHCP:
		next-server
		pxelinux.0

	如何配置PXE：
		1、配置dhcp服务
			subnet ... netmask ... {
				...
				next-server TFTP-SERVER-IP;
				filename "pxelinux.0";
			}

		2、配置tftp server
			# yum install tftp-server
			# chkconfig tftp on
			# service xinetd restart
			# ss -unl | grep :69

		3、提供PXE的工作环境
			# yum install syslinux

			# cp /usr/share/syslinux/pxelinux.0  /var/lib/tftpboot/

		4、提供引导内核等文件

			挂载系统光盘，假设位置为/media/cdrom/

			# cp /media/cdrom/images/pxeboot/{vmlinuz,initrd.img}  /var/lib/tftpboot/
			# cp /media/cdrom/isolinux/{splash.jpg,vesamenu.c32,boot.msg}  /var/lib/tftpboot

			# mkdir /var/lib/tftpboot/pxelinux.cfg/
			# cp /media/cdrom/isolinux/isolinux.cfg  /var/lib/tftpboot/pxelinux.cfg/default

		5、提供安装源
			基于http服务实现

			# yum -y install httpd

			# mkdir -pv /var/www/html/centos/6/x84_64 

			# mount --bind /media/cdrom /var/www/html/centos/6/x84_64

		6、提供ks.cfg文件


	作业：写成博客


回顾：系统安装过程
	anaconda: 安装前配置、系统安装
	引导参数：linux
		ip, netmask, gateway, dns
		ks=

	kickstart文件：anaconda-ks.cfg  制作 system-config-kickstart
		命令段
		%packages
		@group
		[-]pkg_name
		%end

		脚本段：
			%pre：
				安装前脚本的执行环境为非完整的shell环境；

			%post：

	dhcp服务：
		c/s:
			dhcpd: 67/udp
			dhclient: 68/udp
 service configtest
		守护进程：
			standalone
			xinetd
				transient

		CentOS 6:
			独立守护进程：
				chkconfig [--level ###] SERVICE on|off

			非独立守护进程
				1、chkconfig SERVICE on|off
				2、/etc/xinetd.d/SERVICE
					disable = {yes|no}

		CentOS 7:
			systemctl enable SERVICE.service

	PXE: 
		tftp: 69/udp

		/etc/services文件
		/etc/protocols文件

		/var/lib/tftpboot/
			vmlinuz
			initrd.img
				安装源：images/pxelinux/
			vesamenu.c32
			splash.{jpg|png}
			boot.msg
				安装源：isolinux/
			pxelinux.0
				syslinux包
			pxelinux.cfg/
				default
					isolinux/isolinux.cfg

Linux故障排除：CentOS 6
	
	紧急救援模式：rescue
		微小的Linux

	1、grub损坏
		# chroot /mnt/sysimage
		# grub
		grub> root (DEVICE,PART)
		grub> setup (DEVICE)

		# grub-install /dev/DEVICE

	2、bash损坏
		重装bash包

	3、文件系统损坏
		进入紧急救援模式
		/etc/fstab，禁止开机自动挂载

	4、驱动文件损坏
		grub: 
			e:
				kernel:
					emergency：
						不会执行/etc/rc.d/rc.sysinit

总结：修复系统两种方式（紧急救援模式，grub下向内核传递参数（single, emergency））


脚本：写一个脚本，能够ping探测指定网络内的所有主机是否在线？

		#!/bin/bash
		#
		quitScript() {
		    echo "Quit..."
		}    

		trap 'quitScript; exit 5' SIGINT

		cnetPing() {
		    for i in {1..254}; do
			if ping -c 1 -W 1 $1.$i &> /dev/null; then
			     echo "$1.$i is up."
		 	else
			     echo "$1.$i is down."
			fi
		     done
		}

		bnetPing() {
		    for j in {0..255}; do
				cnetPing $1.$j 
		    done
		}

		anetPing() {
		    for m in {0..255}; do
				bnetPing $1.$m
		    done
		}

		netType=`echo $1 | cut -d"." -f1`

		if [ $netType -ge 1 -a $netType -le 126 ]; then
		    anetPing $netType
		elif [ $netType -ge 128 -a $netType -le 191 ]; then
		    bnetPing $(echo $1 | cut -d'.' -f1,2)
		elif [ $netType -ge 192 -a $netType -le 223 ]; then
		    cnetPing $(echo $1 | cut -d'.' -f1-3)
		else
		    echo "Wrong"
		    exit 2
		fi

信号捕捉：
	trap 'COMMAND' SIGNALE
		SIGINT



sed: 

	回顾：
		BRE：
			字符匹配：., [], [^]
			次数匹配: *, \?, \+, \{m,n\}
			锚定: ^, $, \<, \>
			分组引用: \(\), \1, \2, ...

		ERE:
			次数匹配：*, ?, +, {m,n}
			锚定：
			分组引用：(), \1, \2, ...
			或者：|

		grep: 文本过滤工具
		sed: stream editor，行编辑工具
		awk：文本报告生成器

	sed [option]... '地址 编辑命令' FILE...
	
		地址：
			行范围：
				start_line[, end_line]
				/patten1/,/patten2/: 第一次被pattern1匹配到的行开始，到第一次被pattern2匹配到的行结束之间的所有行；
			特定行：
				/pattern/
			无地址：全文

		pattern space: 模式空间

		编辑命令：命令可在之前加!取反
			p：打印
			d: 删除
			i \text: 行上方，text即为插入的内容
			a \text: 行下方，
			r /path/from/some_file: 
			w /path/to/some_file: 把符合条件的行保存至指定的文件中
			=: 显示符合条件行的行号
			s///: s@@@
				g,i


		选项：
			-n: 静默模式，不显示模式空间中的内容
			-r: 支持使用扩展正则表达式
			-i: 修改原文件；
			-e: sed -e "" -e "" -e "", sed "{COM1;COM2;COM3}"

		1、替换/etc/inittab中的"id:3:initdefault"一行数字为5；
			sed -e 's/id:3:initdefault/id:5:initdefault/' 

		2、删除/etc/init.d/functions的空白行；
			/^[[:space:]]*$/d

		3、删除/boot/grub/grub.conf文件中行首的空白字符；
			s

		4、echo一个路径给sed，通过sed取出其目录名；例如echo "/etc/sysconfig/" | sed，返回/etc；
		# echo '/etc/sysconfig/' | sed 's@[^/]\{1,\}/\?$@@'

		高级命令：t, T, D, P, n, N, g, G, h, H


awk: 基本应用
	可以实现对文件中每一行内容的每个字段分别进行格式化，而后进行显示

	awk, --> nawk
	gnu awk --> gawk 

	基本语法：
		awk [option]... 'script' FILE...
		awk [option]... '/PATTERN/{action}' FILE...

		支持使用：变量（内置变量、自定义变量）、循环、条件、数组

		PATTERN：
			地址范围：
				start_line, end_line
				/pat1/,/pat2/
			特定行：
				/pattern/
				#
			表达式：比较
				>, >=, ==, <, <=, !=, ~
			BEGIN模式：在{action}开始之前执行一次
			END模式：在{action}结束之后执行一次；


		内置变量：
			NF：Number of Field
			FS: Field Seperator，输入分隔符；-F:
			OFS：输出时的字段分隔符

			awk分隔符常用的有四种：
				字段的：FS, OFS
				行的：

			引用变量的值，不需要以$开头，所有以$开头的变量，是用于引用字段的值；

		action:
			print "string"
			print $#
			print $1,$2,...


		练习：
		1、显示gid小于500的组；
		# gawk -F: '$3<500{print $1}' /etc/group

		2、显示默认shell为nologin的用户；
		# gawk -F: '$7~/nologin$/{print $1}' /etc/passwd

		3、显示eth0网卡配置文件的配置信息，只显示=号后的内容；
		# gawk -F= '{print $2}' /etc/sysconfig/network-scripts/ifcfg-eth0

		4、显示/etc/sysctl.conf文件定义的内核参数的参数名；
		# awk -F= '/^[^#]/{print $1}' /etc/sysctl.conf

		5、显示eth0网卡的ip地址；
		# ifconfig eth0 | awk -F: '/inet addr/{print $2}' | awk '{print $1}'
		# ifconfig eth0 | awk 'BEGIN{FS="[ :]+"}/inet addr/{print $4}'


		awk: 

回顾：sed, awk
	sed语法：sed [option]... '地址 命令' FILE...
		-r, -i, -e, -n, -f	

	awk语法：gawk [option]... '/pattern/{action}' FILE...
		-F

		模式：
			地址范围：
				起始行,结束行
			表达式：
				$n~/pattern/
			特定行：
			BEGIN
			END 

bash编程：
	
	信号捕捉：trap 'COMMAND;COMMAND' 

	循环控制：
		continue: 提前进入下一轮循环
			用于条件语句中，仅在某些个特殊场景提前进入；
		break [n]：跳出当前循环
			用于条件语句中，

		例如：查看用户登录

			#!/bin/bash
			#
			read -p "Plz enter a username: " userName

			while true; do
			    if who | grep "\<$userName\>" &> /dev/null; then
			        break
			    fi
			    echo "not here."
			    sleep 5
			done

			echo "$userName is logged on."


			#!/bin/bash
			#
			read -p "Plz enter a username: " userName

			until who | grep "\<$userName\>" &> /dev/null; do
			  sleep 5
			  echo "not here"
			done

			echo "here"


写一个脚本：生成10个随机数，排序之；
	
	数组：
		数组名+索引
			数组元素

		索引的表示方式：
			数字索引：a[index]
				a[0], a[1]

			bash 4.0的关联数组
				a[hello], a[hi]

			declare -a 
					-A

		支持稀疏格式：
		仅一维数组

	数组的赋值：
		一次对一个元素赋值：
			a[0]=$RANDOM
			...

		一次对全部元素赋值：
			a=(red blue yellow green)

		按索引进行赋值：
			a=([0]=green [3]=red [2]=blue [6]=yellow)

		命令替换：

		用户输入：
			read -a ARRAY


	数组的访问：

		用索引访问：
			ARRAY[index]

	数组的长度：
		${#ARRAY[*]}
		${#ARRAY[@]}

		练习：写一个脚本，生成10个随机数，保存至数组中；而后显示数组下标为偶数的元素；

		for i in {0..9}; do
		    num[$i]=$RANDOM
		done

	从数组中挑选某元素:
		${ARRAY[@]:offset:number}
			切片：
				offset: 偏移的元素个数
				number: 取出的元素的个数

		${ARRAY[@]:offset}：取出偏移量后的所有元素

		${ARRAY[@]}: 取出所有元素


	数组复制：
		要使用${ARRAY[@]}

		$@: 每个参数是一个独立的串
		$*: 所有参数是一个串


	向数组追加元素：

		示例：复制一个数组中下标为偶数的元素至一个新数组中
			#!/bin/bash
			declare -a mylogs
			logs=(/var/log/ *.log)

			echo ${logs[@]}

			for i in `seq 0 ${#logs[@]}`; do
			    if [ $[$i%2] -eq 0 ];then
			       index=${#mylogs[@]}
			       mylogs[$index]=${logs[$i]}
			    fi
			done

			echo ${mylogs[@]}


	从数组中删除元素：

		unset ARRAY[index]

	练习1：生成10个随机数，升序排序

		#!/bin/bash
		for((i=0;i<10;i++))
		do
			rnd[$i]=$RANDOM
		done

		echo -e "total=${#rnd[@]}\n${rnd[@]}\nBegin to sort"

		for((i=9;i>=1;i--))
		do
			for((j=0;j<i;j++))
			do
				if [ ${rnd[$j]} -gt ${rnd[$[$j+1]]} ] ;then
					swapValue=${rnd[$j]}
					rnd[$j]=${rnd[$[$j+1]]}
					rnd[$[$j+1]]=$swapValue		
				fi
			done
		done

		echo ${rnd[@]}	

	练习2：打印九九乘法表

		#!/bin/bash
		for((i=1;i<=9;i++))
		do 
			strLine=""
			for((j=1;i<=9;j++))
			do
				strLine=$strLine"$i*$j="$[$i*$j]"\t"
				[ $i -eq $j ] && echo -e $strLine && break
			done
		done



字符串操作：

	字符串切片：
		${string:offset:length}

	取尾部的指定个数的字符：
		${string: -length}

	取子串：基于模式
		${variable#*word}：在variable中存储字串上，自左而右，查找第一次出现word，删除字符开始至此word处的所有内容；
		${variable##*word}：在variable中存储字串上，自左而右，查找最后一次出现word，删除字符开始至此word处的所有内容； 
			file='/var/log/messages'
				${file#*/}: 返回的结果是var/log/messages
				${file##*/}: 返回messages

		${variable%word*}: 在variable中存储字串上，自右而左，查找第一次出现word，删除此word处至字串尾部的所有内容；
		${variable%%world*}：在variable中存储字串上，自右而左，查找最后一次出现word，删除此word处至字串尾部的所有内容；
			file='/var/log/messages'
				${file%*/}: 返回的结果是/var/log
				${file%%*/}: 返回结果为空

			phonenumber='010-110-8'
				${phonenumber%%-*}
				${phonenumber##*-}

			url="http://www.magedu.com:80"

	查找替换：
		${variable/pattern/substi}: 替换第一次出现
		${variable//pattern/substi}：替换所有的出现

		${variable/#pattern/substi}：替换行首被pattern匹配到的内容
		${variable/%pattern/substi}：    行尾

		pattern可以使用globbing中的元字符：
			*
			?

	查找删除：
		${variable/pattern}
		${variable//pattern}
		${variable/#pattern}
		${variable/%pattern}

	大小写转换：
		小-->大：${variable^^}
		大-->小：${variable,,}


变量赋值操作：

	    ${parameter:-word}
              Use  Default  Values.  If parameter is unset or null, the expansion of word is substituted.  Otherwise, the value of parameter is
              substituted.
        ${parameter:=word}
              Assign Default Values.  If parameter is unset or null, the expansion of word is assigned to parameter.  The value of parameter is
              then substituted.  Positional parameters and special parameters may not be assigned to in this way.
        ${parameter:?word}
              Display Error if Null or Unset.  If parameter is null or unset, the expansion of word (or a message to that effect if word is not
              present) is written to the standard error and the shell, if it is not interactive, exits.  Otherwise, the value of  parameter  is
              substituted.
        ${parameter:+word}
              Use Alternate Value.  If parameter is null or unset, nothing is substituted, otherwise the expansion of word is substituted.


        ${variable:-string}
        	variable为空或未设定，那么返回string，否则，返回variable变量的值；

        ${variable:=string}
        	variable为空或未设定，则返回string，且将string赋值给变量variable，否则，返回variable的值；


    为脚本使用配置文件，并确保某变量有可用值的方式
    	variable=${variable:-default vaule}

	练习：读取/etc/sysconfig/network文件，利用其HOSTNAME变量的值设置主机名；


	库：函数的组合，通常是一个文件

书：
		《Linux命令行和shell编程宝典》
		《abs-guide》


命令：
	mktemp
		# mktemp 
			--tmpdir=

			-d： 创建临时目录

	install:
		复制文件
		创建目录

		增强型的复制命令：
			-o user
			-g group
			-m mode

			-d : 创建目录

练习：
	1、使用install创建目录/tmp/test; 
	2、在/tmp/test目录创建多个以.txt和.doc结尾的文件；
	3、将.txt结尾的文件的文件名后缀改为.TXT；.doc的改为.DOC (使用bash内置的字符串处理机制)

博客：总结bash脚本编程的所有语法知识点，要求语言通顺，案例详实；
