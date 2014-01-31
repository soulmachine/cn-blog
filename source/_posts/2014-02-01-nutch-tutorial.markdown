---
layout: post
title: "Nutch快速入门"
date: 2014-02-01 04:11
comments: true
categories: Search Engine
---

##1. 安装并运行HBase
为了简单起见，使用Standalone模式，参考 [HBase Quick start](http://hbase.apache.org/book/quickstart.html)

###1.1 下载，解压

    wget http://archive.apache.org/dist/hbase/hbase-0.90.4/hbase-0.90.4.tar.gz
    tar zxf hbase-0.90.4.tar.gz

###1.2 修改 conf/hbase-site.xml
内容如下

    <configuration>
      <property>
        <name>hbase.rootdir</name>
        <value>file:///DIRECTORY/hbase</value>
      </property>
      <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/DIRECTORY/zookeeper</value>
      </property>
    </configuration>

`hbase.rootdir`目录是用来存放HBase的相关信息的，默认值是`/tmp/hbase-${user.name}/hbase； `hbase.zookeeper.property.dataDir`目录是用来存放zookeeper（HBase内置了zookeeper）的相关信息的，默认值是`/tmp/hbase-${user.name}/zookeeper。

###1.3 启动

    $ ./bin/start-hbase.sh
    starting Master, logging to logs/hbase-user-master-example.org.out

<!-- more -->

###1.4 试用一下shell
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

###1.5 停止

    $ ./bin/stop-hbase.sh
    stopping hbase...............

###1.6 再次启动
后面运行Nutch，需要把数据存储到HBase，因此需要启动HBase。

    $ ./bin/start-hbase.sh
    starting Master, logging to logs/hbase-user-master-example.org.out

##2 编译Nutch 2.2.1

###2.1 下载，解压
    wget http://www.apache.org/dyn/closer.cgi/nutch/2.2.1/apache-nutch-2.2.1-src.tar.gz
    tar zxf apache-nutch-2.2.1-src.tar.gz

###2.2 修改配置文件
参考[Nutch 2.0 Tutorial](http://wiki.apache.org/nutch/Nutch2Tutorial)

修改 `conf/nutch-site.xml`

    <property>
      <name>storage.data.store.class</name>
      <value>org.apache.gora.hbase.store.HBaseStore</value>
      <description>Default class for storing data</description>
    </property>

修改`ivy/ivy.xml`

    <!-- Uncomment this to use HBase as Gora backend. -->
    <dependency org="org.apache.gora" name="gora-hbase" rev="0.3" conf="*->default" />
    
修改 `conf/gora.properties`，确保`HBaseStore`被设置为默认的存储，

    gora.datastore.default=org.apache.gora.hbase.store.HBaseStore

###2.3 编译

    ant runtime

刚开始会下载很多jar，需要等待一段时间。

有可能你会得到如下错误：

    Trying to override old definition of task javac
      [taskdef] Could not load definitions from resource org/sonar/ant/antlib.xml. It could not be found.
    
    ivy-probe-antlib:
    
    ivy-download:
      [taskdef] Could not load definitions from resource org/sonar/ant/antlib.xml. It could not be found.

无所谓，不用管它。

要等一会儿才能编译结束。编译完后，多出来了 build 和 runtime两个文件夹。

##3 抓取网页

###3.1 添加种子url，并开始抓取

    mkdir ~/urls
    vim ～/urls/seed.txt
    http://www.douban.com

    cd runtime/local
    ./bin/nutch inject ~/urls -crawlId douban

###3.2 查看结果

    ./bin/nutch readdb -crawlId douban -stats

