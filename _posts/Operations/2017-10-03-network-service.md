---
layout: post
title: "网络服务-Linux运维"
description: "Linux运维之网络服务"
categories: [linux，redhat]
tags: [linux,redhat]
redirect_from:
  - /2017/10/03/
---

> Linux运维之网络服务。关于dns,dhcp,email,ntp,nfs,samba,vsftpd等服务的查缺补漏。

* Kramdown table of contents
{:toc .toc}

## ntp时间同步

1、 客户端（跟时间服务器同步时间）

同步时间方法1：

	[root@localhost ~]# yum install ntpdate
	
	[root@localhost ~]# ntpdate 0.rhel.pool.ntp.org
	2013年 04月 16日 星期二 20:30:46 CST
	16 Apr 20:30:50 ntpdate[2478]: step time server 218.75.4.130 offset 1.333828 sec

时间同步完后，系统还有一个硬件时钟

	[root@mini ~]# hwclock  --show  
	2016年08月05日 星期五 15时38分12秒  -0.357040 seconds
	
	[root@mini ~]# hwclock   --systohc

同步时间方法2：

	[root@localhost ~]# vim /etc/ntp/step-tickers
	0.rhel.pool.ntp.org
	ntp.api.bz
	cn.ntp.org.cn
	
	[root@localhost ~]# vim /etc/sysconfig/ntpdate
	# Options for ntpdate
	OPTIONS="-U ntp -s -b"	
	# Set to 'yes' to sync hw clock after successful ntpdate
	SYNC_HWCLOCK=no    # 可以改为yes同步硬件时钟。
	
	[root@localhost ~]# /etc/init.d/ntpdate start
	Apr 16 20:32:40 localhost ntpdate[2513]: step time server 61.153.197.226 offset 0.007627 sec

建议要定期对服务器进行时间同步，通过计划任务来完成。

	[root@localhost ~]# vim /etc/crontab
	30  3   *  *  * root   /etc/init.d/ntpdate start
	# 或者：
	# 30  3   *  *  * root  /usr/sbin/ntpdate 0.rhel.pool.ntp.org

2、 服务端（提供其他客户同步时间）

	[root@localhost ~]# yum install ntp
	
	[root@localhost ntp]# vim /etc/ntp.conf
	restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap  （可选）
	
	[root@localhost ntp]# service ntpd start
	Starting ntpd:                                             [  OK  ]
	[root@localhost ntp]# chkconfig ntpd on

ntp的监听端口是udp的***123***

	[root@foundation0 ~]# ss  -ntupl |grep ntp
	udp        0      0 0.0.0.0:123         0.0.0.0:*      22764/ntpd

## nfs网络文件系统

1、NFS的安装	

	[root@geust02 ~]# rpm   -qa  |grep  rpcbind
	[root@geust02 ~]# rpm   -qa  |grep  nfs-utils
	[root@geust02 ~]# yum  install  -y  rpcbind   nfs-utils

2、 NFS配置共享某个目录

	[root@geust02 ~]# vim   /etc/exports
	/public     *(ro)
	# /public     *(rw,sync)	# NFS支持写入
	# [root@geust02 ~]# chmod   777  /public/
	
	[root@geust02 ~]# mkdir   /public
	[root@geust02 ~]# cp   /etc/man_db.conf   /public/

3、 NFS的启动

	[root@geust02 ~]# systemctl    start   rpcbind
	[root@geust02 ~]# systemctl    enable   rpcbind
	
	[root@geust02 ~]# systemctl   start  nfs-server
	[root@geust02 ~]# systemctl   enable  nfs-server
	
	[root@geust02 ~]# ss  -4ntulp  |grep   rpcbind
	udp    UNCONN     0      0         *:886                   *:*                   users:(("rpcbind",pid=1135,fd=7))
	udp    UNCONN     0      0         *:111                   *:*                   users:(("rpcbind",pid=1135,fd=6))
	tcp    LISTEN     0      128       *:111                   *:*                   users:(("rpcbind",pid=1135,fd=8))

	# 使配置生效
	[root@geust02 ~]# exportfs    -av
	exporting *:/public
	

4、 客户端测试

	[root@365linux ~]# showmount  -e     192.168.122.109
	Export list for 192.168.122.109:
	/public *
	
	[root@365linux ~]# mount   192.168.122.109:/public     /mnt	
	[root@365linux ~]# ls  /mnt/
	man_db.conf

	[root@365linux ~]# vim /etc/fstab
 	192.168.122.109:/public    /mnt                nfs       defaults     0 0

还可以使用autofs，当然推荐还是使用fstab挂载吧。

备注：

1. NFS默认保留普通用户的文件拥有者的身份，但是，Linux系统对于用户的识别是通过UID来完成的，有可能造成，在客户端和服务器，同一个UID对应的用户名不一样。比如在客户端1001对应的lisi用户，而在服务端对应是zhangsan用户。要注意这点。
2. 而管理员root默认会被映射为nfsnobody，可以通过配置，可以取消root用户的匿名映射。
3. 可以通过配置, 使任何普通用户的访问映射成某个匿名用户，

示例：  
/public          *(rw,sync,no_root_squash)  // 不把root映射为nfsnobody  
/public          *(rw,sync,all_squash,anonuid=1000,anongid=1000) // 所有人都映射为匿名用户（包括root），匿名用户设置为UID＝1000的那个用户

## samba文件系统

1、 安装

	[root@geust02 ~]# rpm  -qa  |grep  samba
	[root@geust02 ~]# yum install  samba

2、 配置

	[root@geust02 ~]# vim /etc/samba/smb.conf
	
	# 配置一个可匿名访问的共享
	
	security = user  
	passdb backend = tdbsam  
	map to guest = Bad User  
	
	# 然后再到配置文件的最后添加：
	
	[pub]
	path =  /pub
	public = yes
	
	[root@geust02 ~]# mkdir  /pub
	[root@geust02 ~]# touch   /pub/smb.txt

配置一个匿名的可读写的共享。    
1. 在smb的配置文件中打开可写的选项，  
2. 共享目录本身要对客户端用户有可写入的权限。  

	[pub]
	path =  /pub
	public = yes
	writable = yes
	
	[root@geust02 ~]# chmod  777  /pub
	[root@geust02 ~]# systemctl   restart  smb

基于用户验证的共享.

	[pub]
	path =  /pub
	public = no     #　需要用户验证的共享
	valid users =  zhangsan  +sales   # 可以访问的用户zhangsan和sales组里的成员 +可以换成＠
	write list  =  zhangsan   # 只有zhangsan用户可以写入。

可写的选项说明：  
writable = yes  所有可访问的用户都可以写。  
write list = zhangsan  能访问用户中只有zhangsan可以写入。  
两个选项二选一。

create mask   控制上传文件的权限

	Default: create mask = 0644
	Example: create mask = 0664

directory mask  控制上传目录的权限

	Default: directory mask = 0755
	Example: directory mask = 0775


添加用户：  

	[root@geust02 ~]# id  zhangsan
	uid=1001(zhangsan) gid=1001(zhangsan) 组=1001(zhangsan)
	
	[root@geust02 ~]# id  lisi
	uid=2001(lisi) gid=2002(sales) 组=2002(sales)
	
	[root@geust02 ~]# smbpasswd   -a  zhangsan
	[root@geust02 ~]# smbpasswd   -a  lisi
	[root@geust02 ~]# systemctl    restart  smb

3、 启动

	[root@geust02 ~]# systemctl   start  smb
	
	[root@geust02 ~]# ss  -4ntupl   |grep  smb
	tcp    LISTEN     0      50        *:139                   *:*                   users:(("smbd",pid=1933,fd=38))
	tcp    LISTEN     0      50        *:445                   *:*                   users:(("smbd",pid=1933,fd=37))

4、 测试

在windows下面访问：在文件浏览器里面，输入\\192.168.122.109  
在Linux下，  
1. 链接到服务器，输入smb://192.168.122.109  
2. 在Nautilus的地址栏(ctrl-l)里面输入smb://192.168.122.109  
3. 命令行下面访问samba，需要安装 samba-client， cifs-utils

查看：  

	[root@vhost01 ~]# smbclient -L 192.168.122.109
	Sharename       Type      Comment
		---------       ----      -------
		pub             Disk

挂载：

	[root@teacher01 ~]# mount -t cifs //192.168.122.109/pub /mnt
	# 或者：
	[root@teacher01 ~]# mount  //192.168.122.109/pub /mnt

如果是Linux ，命令行下面挂载用户验证的samba的方式

	[root@teacher01 ~]# mount -t cifs //192.168.122.74/doc /mnt -o user=zhangsan

## vsftpd文件系统

1、 安装和访问：

	[root@geust02 ~]# rpm   -qa  |grep   vsftpd
	[root@geust02 ~]# yum install  -y  vsftpd
	[root@geust02 ~]# systemctl   start  vsftpd
	[root@geust02 ~]# ss  -ntupl  |grep  vsftpd
	tcp    LISTEN     0      32   :::21       :::*     users:(("vsftpd",pid=2781,fd=3))

	[root@teacher01 ~]# ftp  192.168.122.109
	Connected to 192.168.122.109 (192.168.122.109).
	220 (vsFTPd 3.0.2)
	Name (192.168.122.109:root): ftp
	331 Please specify the password.
	Password:
	230 Login successful.
	Remote system type is UNIX.
	Using binary mode to transfer files.
	ftp> ls
	227 Entering Passive Mode (192,168,122,109,154,218).
	150 Here comes the directory listing.
	drwxr-xr-x    2 0        0               6 Aug 03  2015 pub
	226 Directory send OK.


默认情况下，匿名用户访问的共享目录是/var/ftp/

	[root@vhost01 ~]# ls /var/ftp/
	pub

2、 相关配置：

默认本地用户登录到自己的家目录，可以进行上传下载的操作都可以。 默认情况下，本地用户可以登录FTP后，可以切换到别的系统目录去。这样很不安全。
	
	[root@geust02 ~]# vim /etc/vsftpd/vsftpd.conf
	anonymous_enable=NO
	chroot_local_user=YES　限制所有用户
	allow_writeable_chroot=YES  # 让ftp支持根目录可写
	
	[root@teacher01 ~]# ftp  192.168.122.109
	Connected to 192.168.122.109 (192.168.122.109).
	220 (vsFTPd 3.0.2)
	Name (192.168.122.109:root): liubei
	331 Please specify the password.
	Password:
	230 Login successful.
	Remote system type is UNIX.
	Using binary mode to transfer files.
	ftp> pwd
	257 "/home/liubei"
	ftp> help 


3、 FTP托管模式

	[root@geust02 vsftpd]# rpm  -qa  |grep xinetd
	[root@geust02 vsftpd]# yum install -y  xinetd
	[root@geust02 ~]# systemctl   stop vsftpd
	[root@geust02 ~]# systemctl   disable  vsftpd
	
	[root@geust02 ~]# vim /etc/vsftpd/vsftpd.conf
	listen=NO
	listen_ipv6=NO
	
	[root@geust02 ~]# vim  /etc/xinetd.d/vsftpd
	service ftp
	              {
	                     disable             = no
	                     socket_type         = stream
	                     wait                = no
	                     nice                = 10
	                     user                = root
	                     server              = /usr/sbin/vsftpd
	                     server_args         = /etc/vsftpd/vsftpd.conf
	               }
	
	[root@geust02 ~]# systemctl restart  xinetd
	[root@geust02 ~]# ss  -ntupl |grep  xinetd 
	tcp    LISTEN     0      64       :::21     :::*    users:(("xinetd",pid=3232,fd=5))

## dns配置

默认配置文件，bind能够提供本地（本机127.0.0.1）dns的解析服务，并仅提供缓存功能。

	[root@geust02 ~]# yum  install   -y   bind   bind-chroot  bind-utils
	# 不生成的话，会报错无法找到key
	[root@geust02 ~]# rndc-confgen -a -r /dev/urandom  
	wrote key file "/etc/rndc.key"
	
	[root@geust02 ~]# systemctl   start  named
	# 如果要使用安全的chroot方式， 
	# systemctl   start   named-chroot
	
	[root@geust02 ~]# ss  -4ntupl  |grep   named
	udp    UNCONN     0      0      127.0.0.1:53                    *:*                   users:(("named",pid=1602,fd=513),("named",pid=1602,fd=512))
	tcp    LISTEN     0      10     127.0.0.1:53                    *:*                   users:(("named",pid=1602,fd=20))

提供其他主机查询服务

	[root@geust02 ~]# vim /etc/named.conf
	
	listen-on port 53 { any; };   也可以写一个或多个对外ip地址，用；号隔开。
	#listen-on-v6 port 53 { ::1; };   注释掉或者写 {  none; }; 禁止监听ipv6
	
	allow-query     { any; };   # 也可以写成一个或多个指定的网段； 	
	zone "." IN {
	        type hint;
	        file "named.ca";	# 在进行递归查询的时候，根据named.ca文件中IP地址找到根域服务器。
	};

配置一个有自己数据库权威的DNS服务器

	# 配置过程可以参考以下目录当中的示例文件：
	[root@geust02 ~]# ls -R   /usr/share/doc/bind-9.9.4/sample/
	[root@geust02 ~]# vim /etc/named.conf
	# 在最后添加：
	zone  "upl.net" IN  {
	        type  master;
	        file  "upl.net.db";
	};
	
	[root@geust02 ~]# cd /var/named
	[root@geust02 named]# cp    /usr/share/doc/bind-9.9.4/sample/var/named/my.internal.zone.db     ./upl.net.db
	
	[root@geust02 named]# vim  upl.net.db
	# PS :  MX表示邮件解析记录； ＊表示泛解析；＠表示无主机名的直接解析。  解析条目中的IN可以省略不写。
	@ in soa localhost. root 1 3H 15M 1W 1D
	  ns localhost.
	www      A    192.168.122.110
	www      A    192.168.122.119
	ftp      A    192.168.122.111
	bbs      A    192.168.122.112
	news     CNAME  www
	
	[root@geust02 named]# chmod   640   upl.net.db 
	[root@geust02 named]# chgrp   named  upl.net.db
	
	[root@geust02 named]# systemctl  reload   named		

配置dns反向解析

	zone "122.168.192.in-addr.arpa" IN {
	         type master;
	         file "upl.net.rev.db";
	        };
	
	[root@geust02 named]# cp  -a    upl.net.db   upl.net.rev.db
	
	[root@geust02 named]# vim  upl.net.rev.db 
	@ in soa localhost. root 1 3H 15M 1W 1D
	  ns localhost.
	110      PTR   www.upl.net.
	119      PTR   www.upl.net.
	111      PTR   ftp.upl.net.
	112      PTR   bbs.upl.net.
	110      PTR   news.upl.net.
	119      PTR   news.upl.net.
	
	[root@geust02 named]# systemctl   reload   named

客户端测试：

	[root@365linux ~]# nslookup    192.168.122.110
	Server:		192.168.122.109
	Address:	192.168.122.109#53
	
	110.122.168.192.in-addr.arpa	name = www.upl.net.
	110.122.168.192.in-addr.arpa	name = news.upl.net.
	
	[root@365linux ~]# nslookup    192.168.122.111
	Server:		192.168.122.109
	Address:	192.168.122.109#53
	
	111.122.168.192.in-addr.arpa	name = ftp.upl.net.
	

## dhcp配置

路由器一般自带，不需要特意处理。除非是要做自动安装系统处理。

	[root@vhost01 ~]# yum install dhcp
	[root@vhost01 ~]# vim /etc/dhcp/dhcpd.conf
	option domain-name "crazyhc.com";
	option domain-name-servers 192.168.122.1, 223.6.6.6;	 # 默认DNS
	
	default-lease-time 6000;
	max-lease-time 72000;
	
	log-facility local7;
	
	subnet 192.168.122.0  netmask 255.255.255.0 {
	  range  192.168.122.200   192.168.122.250;
	  option routers    192.168.122.1;
	}
	
	subnet 172.16.0.0  netmask 255.255.0.0 {
	  range  172.16.0.2   172.16.255.254;
	  option routers    172.16.0.1;
	  option domain-name-servers 223.5.5.5, 223.6.6.6;
	}
	
	# 用于定义静态IP， MAC绑定的IP
	host server01 {
	  hardware ethernet 52:54:00:92:3f:48;
	  fixed-address 172.16.100.100;
	}

	[root@vhost01 ~]# service dhcpd restart
	[root@vhost02 ~]# dhclient -v eth0

## email配置

	[root@geust02 ~]# yum install mailx 
	[root@geust02 ~]# vim /etc/mail.rc 
	# 修改配置文件，我这里是使用qq邮箱发送
	set from=shenjianyu@thinktrader.net smtp=smtp.exmail.qq.com
	set smtp-auth-user=majx@thinker.com.cn smtp-auth-password=邮箱密码
	set smtp-auth=login

	[root@geust02 ~]# echo "test mail " |mail -s "test" 2241871@qq.com