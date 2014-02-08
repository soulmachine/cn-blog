---
layout: post
title: "在CentOS上安装HBase 0.96"
date: 2014-02-08 16:42
comments: true
categories: Hadoop
---

环境：CentOS 6.5, jdk 1.7, HBase 0.96.1.1

本文主要参考 [HBase Quick start](http://hbase.apache.org/book/quickstart.html)

##1. 单机模式(Standalone)

###1.1 下载，解压

    $ wget wget http://mirror.esocc.com/apache/hbase/hbase-0.96.1.1/hbase-0.96.1.1-hadoop2-bin.tar.gz
    $ tar zxf hbase-0.96.1.1-hadoop2-bin.tar.gz -C ~/local/opt

###1.2 hadoop-env.sh
在这个文件中要告诉hadoop JDK 安装在了哪里

    $ echo $JAVA_HOME
    /usr/lib/jvm/java
    $ vim conf/hadoop-env.sh

取消`JAVA_HOME`那一行的注释，设置正确的JDK位置

    export JAVA_HOME=/usr/lib/jvm/java

###1.3 修改 conf/hbase-site.xml
内容如下

    <configuration>
      <property>
        <name>hbase.rootdir</name>
        <value>/home/hadoop/local/var/hadoop/hbase</value>
      </property>
      <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/home/hadoop/local/var/hadoop/zookeeper</value>
      </property>
    </configuration>

`hbase.rootdir`目录是用来存放HBase的相关信息的，默认值是`/tmp/hbase-${user.name}/hbase`； `hbase.zookeeper.property.dataDir`目录是用来存放zookeeper（HBase内置了zookeeper）的相关信息的，默认值是`/tmp/hbase-${user.name}/zookeeper`。

###1.4 启动

    $ ./bin/start-hbase.sh
    starting Master, logging to logs/hbase-user-master-example.org.out

<!-- more -->

###1.5 试用一下shell
$ ./bin/hbase shell
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.90.4, r1150278, Sun Jul 24 15:53:29 PDT 2011

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

##2 伪分布式模式(Pseudo-distributed)

