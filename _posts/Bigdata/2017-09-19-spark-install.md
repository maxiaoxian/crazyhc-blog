---
layout: post
title: "Spark的安装与使用"
description: "Spark的安装与使用"
categories: [Spark]
tags: [Spark,bigdata]
redirect_from:
  - /2017/09/19/
---

> Spark是加州大学伯克利分校AMP实验室开发通用内存并行计算框架。具有运行速度快、易用性好、通用性强和随处运行等特点。官方提供的数据表明，如果数据由磁盘读取，速度是Hadoop MapReduce的10倍以上，如果数据从内存中读取，速度可以高达100多倍。 

* Kramdown table of contents
{:toc .toc}

## 安装SPARK

spark与hadoop的关系，类似于操作系统（Hadoop）与应用（Spark）的关系。Spark专注于大数据分析方面，而存储则交给Hadoop、Hbase等。建议将他们安装在一块，这样利于加快IO读写速度。所以，SPARK集群推荐建立在HADOOP集群之上。  

1、 到官网下载对应版本的spark。网址：http://spark.apache.org/downloads.html  

2、 放到hadoop集群的节点上解压。spark的节点数量可以少于hadoop的节点。根据需要而定。  

3、 [zookeeper安装参考](/blog/2017/09/05/zookeeper-install/)、[hadoop安装参考](/blog/2017/09/05/hadoop-install/)

4、 配置spark集群

	[root@hadoop-01 ~]# cat /etc/profile	
	export JAVA_HOME=/usr/lib/jvm/java
	export ZOOKEEPER_HOME=/opt/zookeeper-3.4.10
	export HADOOP_HOME=/opt/hadoop-2.6.5
	export HBASE_HOME=/opt/hbase-1.2.6
	export SPARK_HOME=/opt/spark
	export PATH=$PATH:$ZOOKEEPER_HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin:$SPARK_HOME/bin:$SPARK_HOME/sbin

	[root@hadoop-01 ~]# cp -a /opt/spark/conf/spark-env.sh.template /opt/spark/conf/spark-env.sh
	[root@hadoop-01 ~]# cat /opt/spark/conf/spark-env.sh
	# 设置hadoop配置目录
	export HADOOP_CONF_DIR=/opt/hadoop-2.6.5/etc/hadoop 
	# zookeeper相关配置
	export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=hadoop-01,hadoop-02,hadoop-03 -Dspark.deploy.zookeeper.dir=/spark"
	# Master节点的ip，或host
	SPARK_MASTER_HOST=hadoop-01
	# 生成环境，内存建议调大一点，但不能超过物理机上限（默认：1G）
	SPARK_DRIVER_MEMORY=256M 
	# 执行任务的CPU数量（默认：1）
	SPARK_EXECUTOR_CORES=1
	# 执行任务的内存数量（默认：1G）
	SPARK_EXECUTOR_MEMORY=256M
	
	# Work节点的配置
	[root@hadoop-01 ~]# cat /opt/spark/conf/slaves
	hadoop-02
	hadoop-03

	# 这样就完了。然后，把配置copy到其他机器上。启动
	[root@hadoop-01 ~]# /opt/spark/sbin/start-all.sh 

用JPS命令查看一下吧。主节点上为MASTER，工作节点上为WORK。  

访问一下http://10.123.253.87:8080/，内容如下：  

![](/assets/images/blog/spark-web.png)

## 示例运行

先运行个示例，看看怎么用。后面有时间再详细讲解用法。

	# 本地模式两线程运行
	[root@hadoop-01 ~]# run-example SparkPi 10 --master local[2]

	# Spark Standalone 集群模式运行
	[root@hadoop-01 ~]# spark-submit \
  	--class org.apache.spark.examples.SparkPi \
  	--master spark://hadoop-01:7077 \
  	/opt/spark/examples/jars/spark-examples_2.11-2.2.0.jar \
  	100

	# Spark on YARN 集群上 yarn-cluster 模式运行
	[root@hadoop-01 ~]# spark-submit \
    --class org.apache.spark.examples.SparkPi \
    --master yarn-cluster \  # can also be `yarn-client`
    /opt/spark/examples/jars/spark-examples_2.11-2.2.0.jar\
    10

最后，推荐一个一键部署 Hadoop + Spark 集群工具：https://github.com/marc-chen/hadoop-spark-installer