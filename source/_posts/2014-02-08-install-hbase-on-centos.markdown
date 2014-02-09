---
layout: post
title: "在CentOS上安装HBase 0.96"
date: 2014-02-08 16:42
comments: true
categories: Hadoop
---

环境：CentOS 6.5, jdk 1.7, HBase 0.96.1.1


##（可选）创建新用户，并配置好SSH无密码登录
一般我倾向于把需要启动daemon进程，对外提供服务的程序，即服务器类的程序，安装在单独的用户下面。这样可以做到隔离，运维方面，安全性也提高了。

创建一个新的group,

    $ sudo groupadd hbase

创建一个新的用户，并加入group,

    $ sudo useradd -g hbase hbase

给新用户设置密码，

    $ sudo passwd hbase

在每台机器上创建hbase新用户，并配置好SSH无密码，参考我的另一篇博客，[SSH无密码登录的配置](http://www.yanjiuyanjiu.com/blog/20120102/)


##1. 单机模式(Standalone mode)

###1.1 下载，解压

    $ wget wget http://mirror.esocc.com/apache/hbase/hbase-0.96.1.1/hbase-0.96.1.1-hadoop2-bin.tar.gz
    $ tar zxf hbase-0.96.1.1-hadoop2-bin.tar.gz -C ~/local/opt

###1.2 hbase-env.sh
在这个文件中要指明JDK 安装在了哪里

    $ echo $JAVA_HOME
    /usr/lib/jvm/java
    $ vim conf/hbase-env.sh

取消`JAVA_HOME`那一行的注释，设置正确的JDK位置

    export JAVA_HOME=/usr/lib/jvm/java

###1.3 修改 conf/hbase-site.xml
内容如下

    <configuration>
      <property>
        <name>hbase.rootdir</name>
        <value>/home/hbase/local/var/hbase</value>
      </property>
      <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/home/hbase/local/var/zookeeper</value>
      </property>
    </configuration>

`hbase.rootdir`目录是用来存放HBase的相关信息的，默认值是`/tmp/hbase-${user.name}/hbase`； `hbase.zookeeper.property.dataDir`目录是用来存放zookeeper（HBase内置了zookeeper）的相关信息的，默认值是`/tmp/hbase-${user.name}/zookeeper`。

###1.4 启动

    $ ./bin/start-hbase.sh
    starting Master, logging to logs/hbase-user-master-example.org.out

<!-- more -->

###1.5 试用一下HBase shell
$ ./bin/hbase shell
    2014-02-09 23:56:28,637 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
    HBase Shell; enter 'help<RETURN>' for list of supported commands.
    Type "exit<RETURN>" to leave the HBase Shell
    Version 0.96.1.1-hadoop2, rUnknown, Tue Dec 17 12:22:12 PST 2013
    
    hbase(main):001:0>

创建一张名字为`test`的表，只有一个列，名为`cf`。为了验证创建是否成功，用`list`命令查看所有的table，并用`put`命令插入一些值。

    hbase(main):003:0> create 'test', 'cf'
    0 row(s) in 1.2200 seconds
    hbase(main):003:0> list 'test'
    ..
    1 row(s) in 0.0550 seconds
    hbase(main):004:0> put 'test', 'row1', 'cf:a', 'value1'
    0 row(s) in 0.0560 seconds
    hbase(main):005:0> put 'test', 'row2', 'cf:b', 'value2'
    0 row(s) in 0.0370 seconds
    hbase(main):006:0> put 'test', 'row3', 'cf:c', 'value3'
    0 row(s) in 0.0450 seconds

用`scan`命令扫描table，验证一下刚才的插入是否成功。

    hbase(main):007:0> scan 'test'
    ROW        COLUMN+CELL
    row1       column=cf:a, timestamp=1288380727188, value=value1
    row2       column=cf:b, timestamp=1288380738440, value=value2
    row3       column=cf:c, timestamp=1288380747365, value=value3
    3 row(s) in 0.0590 seconds

现在，disable并drop掉你的表，这会把上面的所有操作清零。

    hbase(main):012:0> disable 'test'
    0 row(s) in 1.0930 seconds
    hbase(main):013:0> drop 'test'
    0 row(s) in 0.0770 seconds 

退出shell，

    hbase(main):014:0> exit

###1.6 停止

    $ ./bin/stop-hbase.sh
    stopping hbase...............


##2 伪分布式模式(Pseudo-distributed mode)

###前提

HBase集群需要一个正在运行的zookeeper集群，要么用自带的，要么用外部的。

用自带的很方便，不需要任何其他操作。

如果用外部的，要先安装并启动一个ZK集群，参考我的这篇博客，[在CentOS上安装ZooKeeper集群](http://www.yanjiuyanjiu.com/blog/20140207)。并在 conf/hbase-env.sh 里，设置`HBASE_MANAGES_ZK=false`。这个值默认为true，HBase自带了一个zk，启动HBase的时候也会先启动zk，如果把这个值设置为false，那么HBase就不会自己管理zk集群了。

一般用外部的zk。因为一般情况下，公司会在集群上安装好zookeeper集群，然后多个项目共用一个zk集群，有利于提高资源利用率。

HBase还需要一个正在运行的HDFS集群，如何搭建请参考我的这篇博客，[在CentOS上安装Hadoop 2.x 集群](http://www.yanjiuyanjiu.com/blog/20140205)。


###2.1 设置SSH无密码登录localhost
先检查一下是能够无密码登录本机，

    ssh localhost

如果提示输入密码，说明不能，按如下步骤设置。

    $ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
    $ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

###2.2 下载，解压

    $ wget wget http://mirror.esocc.com/apache/hbase/hbase-0.96.1.1/hbase-0.96.1.1-hadoop2-bin.tar.gz
    $ tar zxf hbase-0.96.1.1-hadoop2-bin.tar.gz -C ~/local/opt

###2.3 hbase-env.sh
在这个文件中要指明JDK 安装在了哪里

    $ echo $JAVA_HOME
    /usr/lib/jvm/java
    $ vim conf/hbase-env.sh

取消`JAVA_HOME`那一行的注释，设置正确的JDK位置

    export JAVA_HOME=/usr/lib/jvm/java

###2.4 conf/hbase-site.xml
内容如下

    <configuration>
      <property>
        <name>hbase.rootdir</name>
        <value>/home/hadoop/local/var/hadoop/hbase</value>
      </property>
      <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
      </property>
      
      <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
      </property>
      <property>
        <name>hbase.zookeeper.quorum</name>
        <value>zk01, zk02, zk03</value>
      </property>
      <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/home/zookeeper/local/var/zookeeper</value>
      </property>
    </configuration>


`hbase.rootdir`是HBase存放数据的目录，这个值应该从Hadoop集群的core-site.xml里的`fs.defaultFS`或`fs.default.name`拷贝过来。

接下来关于ZooKeeper的三项配置都是从ZooKeeper集群的zoo.cfg里拷贝过来的。


###2.5 启动

    $ ./bin/start-hbase.sh
    starting Master, logging to logs/hbase-user-master-example.org.out

查看一下进程，

    $ jps
    26142 HMaster
    26255 HRegionServer
    26360 Jps

启动了一个HMaster和一个HRegionServer。

###2.6 试用一下HBase shell
见第1.5节。

###2.7 停止

    $ ./bin/stop-hbase.sh
    stopping hbase...............


##3 完全分布式模式(Fully-distributed mode)

###3.1 准备3台机器
跟这篇文章[在CentOS上安装Hadoop 2.x 集群](http://www.yanjiuyanjiu.com/blog/20140205/)的第2.1节很类似。

设3台机器的hostname分别是master, slave01, slave02, master作为HMaster，而slave01,slave02作为HRegionServer。

###3.2 配置 master 无密码登陆到所有机器（包括master自己登陆自己）
参考我的另一篇博客，[SSH无密码登录的配置](http://www.yanjiuyanjiu.com/blog/20120102/)

###3.3 把HBase压缩包上传到所有机器，并解压
将 hbase-0.96.1.1-hadoop2-bin.tar.gz 上传到所有机器，然后解压。**注意，所有机器的hbase路径必须一致，因为master会登陆到slave上执行命令，master认为slave的hbase路径与自己一样。**

下面开始配置，配置好了后，把`conf/`目录scp到所有其他机器。

###3.4 修改配置文件
在第2节的基础上，增加下列修改。

####3.4.1 conf/regionservers
在这个文件里面添加slave，一行一个。

    slave01
    slave01

####3.4.2 将conf/目录拷贝到所有slaves

    $ scp -r conf/ hbase@slave01:$HBASE_HOME/
    $ scp -r conf/ hbase@slave02:$HBASE_HOME/

###3.5 启动HBase集群

####3.5.1 启动
在master上执行：

    $ ./bin/start-hbase.sh

####3.5.2 检查是否启动成功
用`jps`查看java进程。

在master上，应该有一个HMaster进程，在每台slave上，应该有一个HRegionServer进程。

####3.5.3 Web UI

* HMaster: <http://master:60010>
* HRegionServer: <http://slave:60030>

##参考资料

1. [1.2. Quick Start](http://hbase.apache.org/book/quickstart.html)
1. [2.2 HBase run modes: Standalone and Distributed](http://hbase.apache.org/book/standalone_dist.html)
1. [2.4. Example Configurations](http://hbase.apache.org/book/example_config.html)
1. [Chapter 17. ZooKeeper](http://hbase.apache.org/book/zookeeper.html)
1. [CentOS分布式环境安装HBase-0.96.0](http://blog.csdn.net/iam333/article/details/16358087)

