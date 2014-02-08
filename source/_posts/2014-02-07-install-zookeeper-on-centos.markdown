---
layout: post
title: "在CentOS上安装ZooKeeper集群"
date: 2014-02-07 23:40
comments: true
categories: Hadoop
---

环境：CentOS 6.5, jdk 1.7, ZooKeeper 3.4.5

本文主要参考官网的[Getting Started](http://zookeeper.apache.org/doc/trunk/zookeeperStarted.html)

##1. 单机模式(Standalone mode)
单机模式在开发和调试阶段很有用。

###1.1 下载，解压

    $ wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.5/zookeeper-3.4.5.tar.gz
    $ tar zxf zookeeper-3.4.5.tar.gz -C ~/local/opt

###1.2 启动
默认就是单机模式，

    $ mv conf/zoo_sample.cfg conf/zoo.cfg
    $ ./bin/zdServer.sh start

###1.3 使用java 客户端连接ZooKeeper

    $ ./bin/zkCli.sh -server 127.0.0.1:2181

然后就可以使用各种命令了，跟文件操作命令很类似，输入`help`可以看到所有命令。

####1.4 关闭

    $ ./bin/zdServer.sh stop

##2. 分布式模式(Replicated mode)
在生产环境中，要配置成分布式模式，才能发挥威力。

<!-- more -->

ZooKeeper集群，也称为ZooKeeper ensemble，或者  quorum.

###2.1 准备3台机器
假设有三台机器，hostname和ip对应关系是：

	192.168.1.131 zk01
	192.168.1.132 zk02
	192.168.1.133 zk03

ZooKeeper不存在明显的master/slave关系，各个节点都是服务器，leader挂了，会立马从follower中选举一个出来作为leader.

由于没有主从关系，也不用配置SSH无密码登录了，各个zk服务器是自己启动的，互相之间通过TCP端口来交换数据。

###2.2 修改配置文件conf/zoo.cfg

	tickTime=2000
	initLimit=10
	syncLimit=5
	dataDir=/home/zookeeper/local/var/zookeeper
	clientPort=2181
	server.1=zk01:2888:3888
	server.2=zk02:2888:3888
	server.3=zk03:2888:3888

我一般把服务器程序，即需要启动daemon进程的程序，放在单独的用户里安装；且用户的数据，放在`local/var`下面，所以我的dataDir是`/home/zookeeper/local/var/zookeeper`。

###2.3 myid文件
要在每台机器的dataDir下，新建一个myid文件，里面存放一个数字，用来标识当前主机。

    zookeeper@zk01:$ echo "1" >> ~/local/var/zookeeper/myid
    zookeeper@zk02:$ echo "2" >> ~/local/var/zookeeper/myid
    zookeeper@zk03:$ echo "3" >> ~/local/var/zookeeper/myid

###2.4 启动每台机器

    zookeeper@zk01:$ ~/local/opt/zookeeper-3.4.5/bin/zkServer.sh start
    zookeeper@zk02:$ ~/local/opt/zookeeper-3.4.5/bin/zkServer.sh start
    zookeeper@zk03:$ ~/local/opt/zookeeper-3.4.5/bin/zkServer.sh start

因为3个节点的启动是有顺序的，所以在陆续启动三个节点的时候，前面先启动的节点连接未启动的节点的时候会报出一些错误。可以忽略。

###2.5 查看状态

    $ ~/local/opt/zookeeper-3.4.5/bin/zkServer.sh status

##3 使用java客户端连接ZooKeeper集群

找一台机器，解压zookeeper压缩包，不用配置，就可以使用java客户端连接ZooKeeper集群中的任意一台服务器了。

    $ ./bin/zkCli.sh -server zk01:2181
    $ ./bin/zkCli.sh -server zk01:2181
    $ ./bin/zkCli.sh -server zk01:2181

连接上以后，就可以执行各种命令，使用`help`查看帮助。


##参考资料

1. [Getting Started](http://zookeeper.apache.org/doc/trunk/zookeeperStarted.html)
1. [Zookeeper 3.4.5 集群安装笔记](http://blog.csdn.net/jmy99527/article/details/17582349)

