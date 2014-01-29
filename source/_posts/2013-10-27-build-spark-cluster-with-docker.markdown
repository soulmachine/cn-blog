---
layout: post
title: "使用docker打造spark集群"
date: 2013-10-27 20:30
comments: true
categories: Spark
---
**前提条件：**安装好了docker，见我的另一篇博客，[Docker安装](http://www.yanjiuyanjiu.com/blog/20131025)

有两种方式，

* [Spark官方repo](https://github.com/apache/incubator-spark)里，docker文件夹下的脚本。官方的这个脚本封装很薄，尽可能把必要的信息展示出来。
* [AMPLab开源的这个独立小项目](https://github.com/amplab/docker-scripts)，来打造一个spark集群。这个脚本封装很深，自带了一个DNS服务器，还有hadoop，非常自动化，缺点是很多信息看不到了。

# 1. 第1种方式

## git clone 源码
首先要把官方repo的代码下载下来

	git clone git@github.com:apache/incubator-spark.git

## （可选）修改apt源
在国内，将apt源修改国内源，例如163的源，速度会快很多。将`base/Dockerfile`里的
	
	RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list

替换为

	RUN echo "deb http://mirrors.163.com/ubuntu/ precise main restricted universe multiverse" > /etc/apt/sources.list
	RUN echo "deb http://mirrors.163.com/ubuntu/ precise-security main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb http://mirrors.163.com/ubuntu/ precise-updates main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb http://mirrors.163.com/ubuntu/ precise-proposed main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb http://mirrors.163.com/ubuntu/ precise-backports main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb-src http://mirrors.163.com/ubuntu/ precise main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb-src http://mirrors.163.com/ubuntu/ precise-security main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb-src http://mirrors.163.com/ubuntu/ precise-updates main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb-src http://mirrors.163.com/ubuntu/ precise-proposed main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb-src http://mirrors.163.com/ubuntu/ precise-backports main restricted universe multiverse" >> /etc/apt/sources.list
	
## build镜像
将`build`和`spark-test/build`里的`docker`命令前，添加`sudo`，然后执行`docker`下的`build`

	cd docker
	./build

## 启动master

	sudo docker run -v $SPARK_HOME:/opt/spark spark-test-master

## 启动worker
新开一个终端窗口（强烈推荐tmux），启动一个worker

	sudo docker run -v $SPARK_HOME:/opt/spark spark-test-worker <master_ip>

可以在master终端窗口看到worker注册上来了。

可以再开多个终端窗口，启动多个worker。
	

# 2. 第2种方式

## 升级wget
如果发现wget不识别`--no-proxy`选项，需要升级wget。

## 下载镜像
为了让脚本第一次执行的时候更快，还是手动下载所有的镜像吧，amplab在index.docker.io上有一个官方账号，把这个账号有关spark的repo都pull下来。

	sudo docker pull amplab/apache-hadoop-hdfs-precise
	sudo docker pull amplab/dnsmasq-precise
	sudo docker pull amplab/spark-worker
	sudo docker pull amplab/spark-master
	sudo docker pull amplab/spark-shell
	

## git clone 脚本

	git@github.com:amplab/docker-scripts.git

这个脚本可以一键启动集群，爽啊哈哈哈！

## 一键启动spark集群

	sudo ./deploy/deploy.sh -i amplab/spark:0.8.0 -w 3 

## 启动 Spark shell
启动一个交互式shell吧，IP为上一步输出的Master的IP

	sudo docker run -i -t -dns 172.17.0.90 amplab/spark-shell:0.8.0

## 运行一个简单的的例子

	scala> val textFile = sc.textFile("hdfs://master:9000/user/hdfs/test.txt")
	scala> textFile.count()
	scala> textFile.map({line => line}).collect()
	
## 关闭集群

	$ sudo ./deploy/kill_all.sh spark
	$ sudo ./deploy/kill_all.sh nameserver

## 更多详情请参考项目主页的文档

<https://github.com/amplab/docker-scripts>
