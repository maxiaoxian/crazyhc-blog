---
layout: post
title: "系统管理-Linux运维"
description: "Linux运维之系统管理vim,cron,log,volum"
categories: [linux，redhat]
tags: [linux,redhat]
redirect_from:
  - /2017/10/02/
---

> Linux运维之系统管理。关于基本用法、磁盘管理、监控、日志等等。查缺补漏，只对一些使用技巧做个总结。

* Kramdown table of contents
{:toc .toc}

## vim基本用法

1、 进入编辑模式的命令。

|-----------------+------------| 
| 按键 | 功能 |
|-----------------|:-----------|
| i | 从光标所在位置前面开始插入文本 |
| I | 从光标所在行的行首开始插入文本 |
| a | 从光标所在的位置后面开始插入文本 |
| A | 从光标所在行的行尾开始插入文本 |
| o | 在光标所在的行下方新增一行插入文本 |
| O | 在光标所在行上方新增一行插入文本 |
| s | 删除光标所在字符串并开始插入文本 |
| S | 删除光标所在行并开始插入文本 |
|-----------------+------------|

2、 进入末行模式的命令。

|-----------------+------------| 
| 按键 | 功能 |
|-----------------|:-----------|
| : | 在后面接要执行的命令 |
| / | 在后面接要搜索的字符串，从光标位置开始向下搜索，n往后搜，N往前搜索 |
| ? | 同/，搜索方向相反 |
|-----------------+------------|

3、 光标移动命令。

|-----------------+------------| 
| 命令（n表示数字） | 功能 |
|-----------------|:-----------|
| (n)(h/j/k/l) | 光标向左/下/上/右移动n个字符 |
| Ctrl+f/b/d/u | 屏幕向下/上/移动一页（半页） |
| n<space 键> | 光标向后移动 n 字符 |
| n<enter 键> | 光标向下移动 n 行 |
| 0或者^/$ | 光标移动到行首/行尾 |
| gg/G/nG | 光标移动到文件第一行/最后一行/第n行 |
| x/X/nx | 向后/前删除一（n）个字符 |
| dd/ndd | 删除光标所在的行/向下删除 n 行 |
| yy/nyy | 复制光标所在一（向下 n）行 |
| p/P | 粘贴到光标位置下/上一行 |
|-----------------+------------|

6、 替换搜索命令。

|-----------------+------------| 
| 命令 | 功能 |
|-----------------|:-----------|
| n1,n2s/word1/word2/g | 将从n1行到n2行之间的word1替换为word2,g全局匹配 |
| %s/$/word2/g | 在整个文件的每行行尾插入 word2 |
| %/var/char-&/g| 在整个文件中匹配到 var 后替换为 char-var,&指代匹配的结果 |
| 1,$s/word1/word2/gc | 将从第一行到最后一行之间的word1替换为word2,c表示每次替换确认 |
|-----------------+------------|

5、 其他常用命令。

|-----------------+------------| 
| 命令 | 功能 |
|-----------------|:-----------|
| r/R | 替换光标所在字符/一直替换 |
| u/U | 撤销前一/所有操作 |
| Ctrl+r | 重做上一个操作 |
| J | 合并光标所在行与下一行 |
| set nu/set nonu | 设置行号/取消行号 |
| set autoindent | 设置自动对齐格式，(取消 set noautoindent) |
| nohlsearch | 取消搜索到的关键字的高亮显示 |
|-----------------+------------|

## 计划任务

1、 一次性任务

    [root@geust02 ~]# yum install  -y  at
	[root@geust02 ~]# systemctl   start  atd
	[root@geust02 ~]# systemctl   enable   atd
	#　示例：在 2015 年 3 月 23 日执行命令 wall “hello”。
	# 格式非常灵活，at 5:30pm ，at now +5minutes ，at 17:30 tomorrow
	[root@geust02 ~]# at 16:50 03232015
	at> wall "hello"
	at> <EOT>
	#　查询当前等待执行的任务。
	[root@geust02 ~]# atq
	3 2010-03-23 16:50 a root
	# 删除一个等待的任务。
	[root@geust02 ~]# atrm 3
	# 保存计划任务文件路径/var/spool/at/
	[root@geust02 ~]# ls /var/spool/at/

2、 定时任务

范围取值还有一特殊字符的用法:   
"\*" 表示任何时间都接受；  
"," 表示分隔的时段都适用；如 1，3，5 表示某一个域内 1，3，5 都适用。  
"-" 表示一段时间范围都适用；如 2-5 表示 2、3、4、5 都适用。  
"/" 表示每隔多少时间；如在分钟域的*/5 表示每隔 5 分钟。  
	
	# root 用户创建一个计划任务
	[root@geust02 ~]# crontab -e
	# 列出当前用户的周期计划任务列表
	[root@geust02 ~]# crontab -l
	# 删除当前用户所有的计划任务
	[root@geust02 ~]# crontab -r
	# 系统中保存的用户周期性计划任务的文件
	[root@geust02 ~]# ls /var/spool/cron/
	# 系统级需要定时运行的任务
	[root@geust02 ~]# vim /etc/crontab
	# Example of job definition:
	# .---------------- minute (0 - 59)
	# | .------------- hour (0 - 23)
	# | | .---------- day of month (1 - 31)
	# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
	# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
	# | | | | |
	# * * * * * user-name command to be executed

## 日志分析

	# 手动分割日志
	[root@server01 httpd]# logrotate -f  /etc/logrotate.d/httpd 
	# 查看 logratate.conf 配置文件信息
	[root@server01 ~]# grep -Ev '^#' /etc/logrotate.conf
	主要参数详解：
	weekly # 默认每周对日志文件进行一次轮替。
	rotate 4 # 保留 4 个日志文件。
	create # 创建新的日志文件。
	include /etc/logrotate.d #包括该目录下的文件。
	/var/log/wtmp { # 对 wtmp 日志文件的单独定义。
		 monthly # 每一个月进行一次轮替。
		 minsize 1M # 或者是日志大小超过 1M 时进行轮替。
		 create 0664 root utmp # 新建日志文件的权限和用户及所属组。
		 rotate 1 #只保留一个日志文件。
	}
	# 查看子配置文件目录下的 syslog 的单独定义的轮替规则
	[root@server01 logrotate.d]# cat /etc/logrotate.d/syslog
	/var/log/messages /var/log/secure /var/log/maillog /var/log/spooler /var/log/boot.log 
	/var/log/cron { #定义了多个日志文件的轮替。
		 sharedscripts # 与 endscript 搭配，可以定义轮替前或轮替后执行的命令。
		 postrotate # 以下是轮替很执行的命令。如要在轮替前执行，则使用 prerotate。
		 /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
		 /bin/kill -HUP `cat /var/run/rsyslogd.pid 2> /dev/null` 2> /dev/null || true
		 endscript
	}

## 特俗权限

1、 特殊权限

|-----------------+------------+------------| 
| 特殊权限 | 对文件的影响 | 对目录的影响 |
|-----------------|:-----------|:-----------|
| u+s | 以拥有文件的用户身份执行文件，而不是以运行文件的用户身份 | 无影响。 |
| g+s | 以拥有文件的组身份执行文件 | 在目录中最新创建的文件将其组所有者设置为与目录的组所有者相同 |
| o+t (sticky) | 无影响 | 对目录具有写入权限的用户仅可以删除其拥有的文件，而无法删除其他用户所拥有的文件 |
|-----------------+------------+------------|

2、 隐藏的扩展属性

	# 查看文件的扩展属性
	[root@server01 ~]# lsattr testfile.txt 
	# 设置文件的扩展属性。
	# 添加“i”的属性表示该文件不能被删除、不能修改、不能重命名、不能创建链接
	[root@server01 ~]# chattr +i testfile.txt 
	[root@server01 ~]# lsattr testfile.txt 

3、 访问控制列表 ACL

ACL可以实现除了针对文件或目录的拥有者、所属组，其他人进行读、写、执行的权限的划分外的人员进行权限细分。

	# 查看文件或目录的 ACL 权限。	
	# 文件 testfile 的用户和组都是 root，它的权限是 644，只有 root 用户是读写，其他人都是只读。而设置了 ACL，ACL 用户 zhangsan 则对该文件拥有读、写、执行的权限。
	[root@server01 test]# getfacl testfile 
	# file: testfile
	# owner: root
	# group: root
	user::rwuser:zhangsan:rwx
	group::r--
	mask::rwx
	other::r--
	# 添加了 ACL 的文件，使用 ls -l 命令列出时，权限位的最后带有“+”号。
	[root@server01 test]# ll testfile 
	-rw-rwxr--+ 1 root root 75 3 月 26 09:33 testfile
	# 针对用户 zhangsan 来设置权限为 rwx
	[root@server01 test]# setfacl -m u:zhangsan:rwx acltest.file
	# 针对组 sales 来设置权限为 rw
	[root@server01 test]# setfacl -m g:sales:rw acltest.file
	# 设置限制的权限为 r
	# 因为设置了限制权限 mask 为 r--，两个权限相与后，最终 zhangsan 对该文件的权限为 r--；同理文件所属组的权限为 r--；
	[root@server01 test]# setfacl -m m:r acltest.file
	# 同时设置多个 ACL 用户和组的权限
	[root@server01 test]# setfacl -m u:user03:rwx,u:user04:rwx,g:sales:rw testfile
	# 删除 ACL 权限
	[root@server01 test]# setfacl -x g:zhangsan acltest.file
	# 一次性删除所有的 ACL 权限
	[root@server01 test]# setfacl -b acltest.file
	# 命令 cp 和 mv 都支持备份时保留文件的 ACL，cp 命令需要加上-p 参数。
	# 备份目录及其子目录中文件的 ACL
	[root@server01 test]# getfacl -R testdir/ > testdir.acl
	# 为测试，删除原文件所有的 ACL
	[root@server01 test]# setfacl -R -b testdir/
	# 从 testdir.acl 文件中恢复被删除的 ACL 信息
	[root@server01 test]# setfacl --restore testdir.acl

## 逻辑卷

1、 调整swap分区的大小

	# 先使用fdisk分区，并标记为swap
	[root@server01 ~]# fdisk /dev/vdb
	   设备 Boot      Start         End      Blocks   Id  System
	/dev/vdb1            2048     4196351     2097152   83  Linux
	/dev/vdb2         4196352     6293503     1048576   83  Linux
	
	命令(输入 m 获取帮助)：t
	分区号 (1,2,4，默认 4)：2
	Hex 代码(输入 L 列出所有代码)：L
	Hex 代码(输入 L 列出所有代码)：82
	已将分区“Linux”的类型更改为“Linux swap / Solaris”
	
	命令(输入 m 获取帮助)：p
	
	   设备 Boot      Start         End      Blocks   Id  System
	/dev/vdb1            2048     4196351     2097152   83  Linux
	/dev/vdb2         4196352     6293503     1048576   82  Linux swap / Solaris
	
	命令(输入 m 获取帮助)：w
	
	[root@server01 ~]# partprobe
	
	# 格式化vdb2为swap文件系统
	[root@server01 ~]# mkswap    /dev/vdb2
	
	# 使用vdb2 swap
	[root@server01 ~]# swapon  /dev/vdb2
	
	# 如果需要开机自动激活，写入/etc/fstab
	[root@server01 ~]# blkid   /dev/vdb2
	/dev/vdb2: UUID="4d6446c4-2016-4d10-87d1-22d28e7f0678" TYPE="swap"	
	[root@server01 ~]# vim  /etc/fstab
	UUID=4d6446c4-2016-4d10-87d1-22d28e7f0678   swap    swap    defaults     0 0
	
	# 使用交换文件的方式创建swap
	[root@server01 ~]# dd   if=/dev/zero   of=/swapfile   bs=4k   count=250000
	记录了250000+0 的读入
	记录了250000+0 的写出
	1024000000字节(1.0 GB)已复制，11.2164 秒，91.3 MB/秒
	
	[root@server01 ~]# mkswap   /swapfile 
	正在设置交换空间版本 1，大小 = 999996 KiB
	无标签，UUID=d7cefa54-9bb6-42af-9698-164a6e1694e0
	[root@server01 ~]# swapon   /swapfile 
	swapon: /swapfile：不安全的权限 0644，建议使用 0600。
	[root@server01 ~]# chmod  0600  /swapfile 

2、 逻辑卷设置

同步文件系统，ext４的文件系统采用resize2fs命令，而xfs文件系统采用xfs_growfs命令同步  
格式话文件系统，ext4使用mkfs.ext4，而xfs文件系统采用mkfs.xfs  
partprobe:将磁盘分区表变化信息通知内核,请求操作系统重新加载分区表。

	[root@server01 ~]# fdisk  /dev/vdc
	命令(输入 m 获取帮助)：n
	Partition type:
	   p   primary (0 primary, 0 extended, 4 free)
	   e   extended
	Select (default p): 
	Using default response p
	分区号 (1-4，默认 1)：
	起始 扇区 (2048-10485759，默认为 2048)：
	将使用默认值 2048
	Last 扇区, +扇区 or +size{K,M,G} (2048-10485759，默认为 10485759)：
	将使用默认值 10485759
	分区 1 已设置为 Linux 类型，大小设为 5 GiB
	
	命令(输入 m 获取帮助)：t
	已选择分区 1
	Hex 代码(输入 L 列出所有代码)：8e
	已将分区“Linux”的类型更改为“Linux LVM”
	
	命令(输入 m 获取帮助)：w
	
	[root@server01 ~]# fdisk  -l |grep  vd
	磁盘 /dev/vda：21.5 GB, 21474836480 字节，41943040 个扇区
	/dev/vda1   *        2048     1026047      512000   83  Linux
	/dev/vda2         1026048    41943039    20458496   8e  Linux LVM
	磁盘 /dev/vdb：2147 MB, 2147483648 字节，4194304 个扇区
	/dev/vdb1            2048     4194303     2096128   8e  Linux LVM
	磁盘 /dev/vdc：5368 MB, 5368709120 字节，10485760 个扇区
	/dev/vdc1            2048    10485759     5241856   8e  Linux LVM
	[root@server01 ~]# partprobe
	[root@server01 ~]# cat /proc/partitions |grep  vd
	 252        0   20971520 vda
	 252        1     512000 vda1
	 252        2   20458496 vda2
	 252       16    2097152 vdb
	 252       17    2096128 vdb1
	 252       32    5242880 vdc
	 252       33    5241856 vdc1
	
	# 创建物理卷，卷组，逻辑卷
	
	[root@server01 ~]# pvcreate    /dev/vdb1   /dev/vdc1
	  Physical volume "/dev/vdb1" successfully created
	  Physical volume "/dev/vdc1" successfully created

	[root@server01 ~]# vgcreate     vg_test    /dev/vdb1  /dev/vdc1
	  Volume group "vg_test" successfully created
	
	[root@server01 ~]# lvcreate  -n   lv_test   -L   4G   vg_test
	  Logical volume "lv_test" created.

	[root@server01 ~]# mkfs.ext4     /dev/vg_test/lv_test 

	# 扩展逻辑卷
	[root@server01 test]# vgdisplay    vg_test
	[root@server01 test]# lvextend    -L +2G   /dev/vg_test/lv_test 
	[root@server01 test]# resize2fs    /dev/vg_test/lv_test

	# 加1G的硬盘
	[root@server01 test]# pvcreate   /dev/vdd1
	  Physical volume "/dev/vdd1" successfully created
	[root@server01 test]# vgextend   vg_test   /dev/vdd1
	  Volume group "vg_test" successfully extended
	[root@server01 test]# vgdisplay   vg_test
	[root@server01 test]# lvextend   -l   +100%FREE   /dev/vg_test/lv_test 
	  Size of logical volume vg_test/lv_test changed from 6.00 GiB (1536 extents) to 7.99 GiB (2045 extents).
	  Logical volume lv_test successfully resized.
	[root@server01 test]# resize2fs   /dev/vg_test/lv_test


	