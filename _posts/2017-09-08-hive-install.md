---
layout: post
title: "Hive的安装与使用"
description: "Hive的安装与使用"
categories: [hadoop]
tags: [hadoop,hive,bigdata]
redirect_from:
  - /2017/09/05/
---

> Hive是Facebook开发的构建于Hadoop集群之上的数据仓库应用，它提供了类似于SQL语法的HQL语句作为数据访问接口，这使得普通分析人员的应用Hadoop的学习曲线变小。

* Kramdown table of contents
{:toc .toc}

## 安装HIVE  

1、先安装HADOOP，具体查看《[Hadoop安装方式与开发环境](/blog/2017/09/05/hadoop-install/)》一文  

2、准备一个MYSQL环境。我们用MYSQL作为HIVE的Metadata存储。

2、下载稳定版HIVE解压。地址：[https://mirrors.cnnic.cn/apache/hive/](https://mirrors.cnnic.cn/apache/hive/)  
另外，还需要下载mysql-connector-java-5.1.44-bin.jar放到$HIVE_HOME/lib目录下

3、配置HIVE。

	[root@hadoop_0 apache-hive-2.1.1-bin]# cat /etc/profile
	export JAVA_HOME=/usr/lib/jvm/java
	export HADOOP_INSTALL=/opt/hadoop-2.6.5
	export HIVE_HOME=/opt/apache-hive-2.1.1-bin
	export PATH=$PATH:$HADOOP_INSTALL/bin:$HADOOP_INSTALL/sbin:$HIVE_HOME/bin
	[root@hadoop_0 conf]# cd conf ; cp hive-env.sh.template hive-env.sh ; cat hive-env.sh
	HADOOP_HOME=/opt/hadoop-2.6.5
	export HIVE_CONF_DIR=/opt/apache-hive-2.1.1-bin/conf
	[root@hadoop_0 conf]# cp hive-default.xml.template hive-site.xml ; cat hive-site.xml
	<!-- 数据库相关配置 -->
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
      	<value>jdbc:mysql://10.123.253.87:3306/hive?useSSL=false&amp;createDatabaseIfNotExist=true</value>
      	<description>JDBC connect string for a JDBC metastore</description>
    </property>
    <property>
     	<name>javax.jdo.option.ConnectionDriverName</name>
      	<value>com.mysql.jdbc.Driver</value>
      	<description>Driver class name for a JDBC metastore</description>
    </property>
    <property>
      	<name>javax.jdo.option.ConnectionUserName</name>
      	<value>root</value>
      	<description>username to use against metastore database</description>
    </property>
    <property>
      	<name>javax.jdo.option.ConnectionPassword</name>
      	<value>toor</value>
      	<description>password to use against metastore database</description>
    </property>
	<!-- 此配置需要修改，否则会报错 -->
	<property>
	  	<name>hive.querylog.location</name>
	  	<value>/opt/apache-hive-2.1.1-bin/iotmp</value>
	  	<description>Location of Hive run time structured log file</description>
	</property>
	<property>
    	<name>hive.exec.local.scratchdir</name>
    	<value>/opt/apache-hive-2.1.1-bin/iotmp</value>
    	<description>Local scratch space for Hive jobs</description>
	</property>
	<property>
    	<name>hive.downloaded.resources.dir</name>
    	<value>/opt/apache-hive-2.1.1-bin/iotmp</value>
    	<description>Temporary local directory for added resources in the remote file system.</description>
	</property>
	
	[root@hadoop_0 conf]# schematool -dbType mysql -initSchema # 导入数据
	
## 使用HIVE  
启动hiveserver2，访问http://10.123.253.87:10002查看。

	[root@hadoop_0 ~]# hiveserver2 &
	[root@hadoop_0 ~]# beeline # 使用beeline连接
	beeline> ! connect jdbc:hive2://  # 嵌入式连接方式。生产环境需要配置认证策略。
	0: jdbc:hive2://> show tables;
	OK
	+-----------+--+
	| tab_name  |
	+-----------+--+
	| test      |
	+-----------+--+
	1 row selected (0.12 seconds)

## JDBC调用HIVE

## SPRING调用HIVE
	
	
	
	
