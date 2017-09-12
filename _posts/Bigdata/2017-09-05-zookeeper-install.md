---
layout: post
title: "Zookeeper的安装与使用"
description: "Zookeeper的安装与使用"
categories: [hadoop]
tags: [hadoop,zookeeper,bigdata]
redirect_from:
  - /2017/09/12/
---

> ZooKeeper是一个分布式的，开放源码的分布式应用程序协同服务。是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。

* Kramdown table of contents
{:toc .toc}


## Zookeeper的安装

1、	准备3个节点（建议节点数为：2n+1）  

	[root@hadoop_1 ~]# cat /etc/hosts
	10.123.253.87 hadoop_1
	10.123.253.88 hadoop_2
	10.123.253.89 hadoop_3

2、  先安装JDK及JRE

    yum install -y java java-1.8.0-openjdk-devel

3、	下载zookeeper安装包并安装  

下载地址： http://www.apache.org/dyn/closer.cgi/zookeeper/  

配置环境变量：  

	[root@hadoop_1 ~]# vim /etc/profile
	export JAVA_HOME=/usr/lib/jvm/java
	export ZOOKEEPER_HOME=/opt/zookeeper-3.4.10
	export PATH=$PATH:$ZOOKEEPER_HOME/bin:$JAVA_HOME/bin
	[root@hadoop_1 ~]# . /etc/profile  

4、	分布式配置  
独立模式直接运行即可。  
伪分布式，同分布式模式，在同一台机器上运行多个不同端口的程序。  
主要讲讲分布式模式的配置：

	[root@hadoop_1 conf]# cp zoo_sample.cfg zoo.cfg
	[root@hadoop_3 conf]# vim zoo.cfg
	initLimit=5 # follow到leader的初始连接超时，指定的是tickTime的倍数。
	syncLimit=2 # leader和follow之间消息响应的时间限制，超时后，follow被丢弃。
	dataDir=/tmp/zookeeper
	server.1=hadoop_1:2888:3888
	server.2=hadoop_2:2888:3888
	server.3=hadoop_3:2888:3888

设置一个自己的标识：
	
	[root@hadoop_1 ~]# mkdir -p /tmp/zookeeper
	[root@hadoop_1 ~]# vim /tmp/zookeeper/myid
	1 # 同zoo.cfg中server.后的值。内容只有一行，数字，介于1~255

5、 启动zookeeper

	# 默认配置启动
	[root@hadoop_1 ~]# zkServer.sh start
	# 指定配置启动
	[root@hadoop_1 ~]# zkServer.sh start zoo1.cfg
	[root@hadoop_1 ~]# zkServer.sh stop
	[root@hadoop_1 ~]# zkServer.sh status # 检查一下状态，看看自己是什么节点
	[root@hadoop_1 ~]# zkServer.sh restart

就这么简单。over。

## zookeeper的基本用法

1、	远程获得server的信息

|-----------------+------------+-----------------| 
| 编码 | 命令 | 说明 | 
|-----------------|:-----------|:---------------:|
| 1 | conf | 配置信息 |
| 2 | cons | 连接信息 |
| 3 | dump | 未处理会话节点 |
| 4 | envi | 环境信息 |
| 5 | reqs | 未处理请求 |
| 6 | ruok | are you ok? imok |
| 7 | stat | 统计信息 |
| 8 | wchs | 服务器watch的详细信息 |
| 9 | wchp | 列出指定路径下的服务器信息 |
|-----------------+------------+-----------------|
	
	# 安装nc
	[root@hadoop_1 ~]# yum install -y nmap-ncat-6.40-7
	# 获取连接信息
	[root@hadoop_1 ~]# echo cons | nc hadoop_2 2181    
 	/10.123.253.87:37272[0](queued=0,recved=1,sent=0)