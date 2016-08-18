---
layout: post
title: "安装 mahout"
date: 2013-07-04 15:47
comments: true
categories: mahout
published: false
---

**前提条件**：已经安装好Hadoop集群，并启动。参考我的博客，[在CentOS上安装Hadoop](http://www.yanjiuyanjiu.com/blog/20130612)，[在Ubuntu上安装Hadoop](http://www.yanjiuyanjiu.com/blog/20120103)。

## 编译源码进行安装

见官网文档， [Installation/Setup](https://cwiki.apache.org/confluence/display/MAHOUT/Mahout+Wiki#MahoutWiki-Installation%2FSetup)

不推荐这个方式，太繁琐。

## 下载预编译好的压缩包进行安装

这种方法最简单。

### 下载

在首页点击右侧的"Download Mahout"按钮，下载预编译好的二进制包。我下载的是mahout-distribution-0.7.tar.gz。

### 解压

解压到$HOME目录

	$ tar -zxf mahout-distribution-0.7.tar.gz
	$ cd mahout-distribution-0.7

### 配置

只需要在Master这台机器上配置即可。

设置 HADOOP_PREFIX 环境变量，把Hadoop的安装位置告诉Mahout。或者设置HADOOP_HOME环境变量，不过HADOOP_HOME已经是derecated了，不建议使用。

设置 HADOOP_CONF_DIR，把Hadoop配置文件的位置告诉Mahout。

[可选]为了方便，将mahout命令加入PATH。

将Mahout的jar加入CLASSPATH，在编译代码时需要。

	$ vim ~/.bash_profile
	export HADOOP_PREFIX=$HOME/hadoop-1.1.2
	export HADOOP_CONF_DIR=$HADOOP_PREFIX/conf
	export PATH=$PATH:$HOME/mahout-distribution-0.7/bin
	export CLASSPATH=$CLASSPATH:$HOME/mahout-distribution-0.7/mahout-core-0.7.jar
	$ source  ~/.bash_profile

以上配置都是通过阅读 mahout这个脚本才知道的。

	$ vim mahout-distribution-0.7/bin/mahout

打开这个脚本，可以看到所有配置，一目了然。


## 运行自带的 KMeans 例子

主要参考官网文档，[Clustering of synthetic control data](https://cwiki.apache.org/confluence/display/MAHOUT/Clustering+of+synthetic+control+data)。

### 下载数据

下载地址：[synthetic_control.data](http://archive.ics.uci.edu/ml/databases/synthetic_control/synthetic_control.data)  
[数据格式说明](http://archive.ics.uci.edu/ml/databases/synthetic_control/synthetic_control.data.html)

### 上传到HDFS

	$ hadoop fs -mkdir testdata
	$ hadoop fs -put ./synthetic_control.data testdata

Mahout自带的mahout-examples-0.7-job.jar 默认从 testdata目录读取数据，所以目录必须命名为testdata

### 运行自带的kmeans算法

	$ mahout org.apache.mahout.clustering.syntheticcontrol.kmeans.Job

或者

	$ hadoop jar ~/mahout-distribution-0.7/mahout-examples-0.7-job.jar org.apache.mahout.clustering.syntheticcontrol.kmeans.Job

或者

	$ mahout jar ~/mahout-distribution-0.7/mahout-examples-0.7-job.jar org.apache.mahout.clustering.syntheticcontrol.kmeans.Job

三者是等价的。感兴趣的话去阅读 `bin/mahout` 这个脚本，看看最后几行，包含 `exec` 的那几行代码。

### 查看结果

	$ hadoop fs -lsr output

如果看到以下结果，说明算法运行成功，Mahout的安装也就成功了。

	clusteredPoints  clusters-0  clusters-1  clusters-10  clusters-2  clusters-3  clusters-4 clusters-5  clusters-6  clusters-7  clusters-8  clusters-9  data

## 参考资料
1. [Installation/Setup](https://cwiki.apache.org/confluence/display/MAHOUT/Mahout+Wiki#MahoutWiki-Installation%2FSetup)
1. [Mahout安装与配置](http://www.cnblogs.com/linjiqin/archive/2013/03/15/2961649.html)