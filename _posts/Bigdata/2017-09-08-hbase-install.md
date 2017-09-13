---
layout: post
title: "Hbase的安装与使用"
description: "Hbase的安装与使用"
categories: [hadoop]
tags: [hadoop,hbase,bigdata]
redirect_from:
  - /2017/09/08/
---

> HBase – Hadoop Database，是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群。

* Kramdown table of contents
{:toc .toc}

## Hbase单机模式

单机模式比较简单，参考一下：[Hbase官方文档](http://abloz.com/hbase/book.html#quickstart)

## Hbase伪分布式
伪分布模式也比较简单，参考一下：[Hbase官方文档 ](http://abloz.com/hbase/book.html#standalone_dist)  

## Hbase完全分布式

生产环境中使用。重点讲下这个。  
先配置HADOOP,和ZOOKEEPER。参照：《[Hadoop安装方式与开发环境](/blog/2017/09/05/hadoop-install/)》和《[Zookeeper的安装与使用](/blog/2017/09/05/zookeeper-install/)》两篇文章。  

然后，当然是下载解压，配置环境变量，这些就不多说了。  
重点讲下配置,包括hbase-env.sh，hbase-site.xml，regionservers ：  

	# 配置环境变量
	[root@hadoop-01 conf]# cat hbase-env.sh
	export JAVA_HOME=/usr/lib/jvm/java
	export HBASE_MANAGES_ZK=false
	
	# hbase配置
	[root@hadoop-01 conf]# cat hbase-site.xml 
	<configuration>
	  <!-- hadoop nameNode -->
	  <property>
	     <name>hbase.rootdir</name>
	     <value>hdfs://hadoop-01/hbase</value>
	  </property>
	  <property>
	     <name>hbase.cluster.distributed</name>
	     <value>true</value>
	  </property>
	  <!-- zookeeper节点 -->
	  <property>
	    <name>hbase.zookeeper.quorum</name>
	    <value>hadoop-01,hadoop-02,hadoop-03</value>
	  </property>
	</configuration>

	# 配置regionservers
	[root@hadoop-01 conf]# cat regionservers 
	hadoop-02
	hadoop-03

	# 启动看看
	[root@hadoop-01 conf]# start-hbase.sh

最后，用jps命令查看下进程。master节点：HMaster，数据节点：HRegionServer

## Hbase的Shell  
使用比较简单，参考一下：[HBase官方文档](http://abloz.com/hbase/book.html#shell_exercises)

## JAVA客户端调用

