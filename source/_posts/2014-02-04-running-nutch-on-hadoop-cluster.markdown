---
layout: post
title: "把Nutch爬虫部署到Hadoop集群上"
date: 2014-02-03 22:36
comments: true
categories: Search-Engine
---

软件版本：Nutch 1.7, Hadoop 1.2.1, CentOS 6.5, JDK 1.7

前面的3篇文章中，[Nutch 快速入门(Nutch 1.7)](http://www.yanjiuyanjiu.com/blog/20140121)，[Nutch 快速入门(Nutch 2.2.1)](http://www.yanjiuyanjiu.com/blog/20140201)，[在Eclipse里运行Nutch](http://www.yanjiuyanjiu.com/blog/20140120)，Nutch都是跑在单机上，本文把Nutch部署到Hadoop集群上，在真正的分布式Hadoop集群上跑。


##前提

* 学会了搭建一个分布式Hadoop集群，见[在CentOS上安装Hadoop集群](http://www.yanjiuyanjiu.com/blog/20140202)
* 学会了单机跑Nutch，见[Nutch 快速入门(Nutch 1.7)](http://www.yanjiuyanjiu.com/blog/20140121)

##1 将Nutch跑在伪分布式Hadoop集群上
现在单机上实验，用伪分布式Hadoop，先练练手，因为伪分布式的步骤和真分布式很类似。

##1.1 伪分布式Hadoop集群的配置

conf/masters:

    localhost


conf/slaves:

    localhost

conf/hadoop-env.sh，新增了如下三行

    export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-amd64
    export HADOOP_OPTS="-server -Djava.net.preferIPv4Stack=true $HADOOP_OPTS"
    export HADOOP_PID_DIR=/home/soulmachine/local/var/hadoop/pids

<!-- more -->

conf/core-site.xml:

    <configuration>
        <property>
            <name>fs.default.name</name>
            <value>hdfs://localhost:9000</value>
        </property>
        <property>
            <name>fs.checkpoint.dir</name>
            <value>/home/soulmachine/local/var/hadoop/dfs/namesecondary</value>
        </property>
    </configuration>

conf/hdfs-site.xml:

    <configuration>
         <property>
             <name>dfs.name.dir</name>
             <value>/home/soulmachine/local/var/hadoop/dfs/name</value>
         </property>
         <property>
             <name>dfs.data.dir</name>
             <value>/home/soulmachine/local/var/hadoop/dfs/data</value>
         </property>
         <property>
             <name>dfs.replication</name>
             <value>1</value>
         </property>
    </configuration>

conf/mapred-site.xml:

    <configuration>
         <property>
             <name>mapred.job.tracker</name>
             <value>localhost:9001</value>
         </property>
        <property>
            <name>mapred.local.dir</name>
            <value>/home/soulmachine/local/var/hadoop/mapred/local</value>
        </property>
        <property>
            <name>mapred.system.dir</name>
            <value>/home/soulmachine/local/var/hadoop/mapred/system</value>
        </property>
    </configuration>

启动，

    $ cd ~/local/opt/hadoop-1.2.1
    $ bin/hadoop namenode -format
    $ bin/start-all.sh

###1.2 重新编译Nutch
每次修改了`$NUTCH_HOME/conf`下的的文件，都需要重新编译Nutch，重新打包生成一个nutch-x.x.x.job文件，见这里，[Running Nutch in (pseudo) distributed-mode](http://wiki.apache.org/nutch/NutchHadoopSingleNodeTutorial)

修改 conf/nutch-site.xml:

    <property>
      <name>http.agent.name</name>
      <value>My Nutch Spider</value>
    </property>

修改 `regex-urlfilter.txt`, 见[Nutch 快速入门(Nutch 1.7)](http://www.yanjiuyanjiu.com/blog/20140121/) 第4节，

    # accept anything else
    #注释掉这行
    #+.
    +^http://movie.douban.com/subject/[0-9]*/$

将Hadoop的六个配置文件，拷贝到Nutch的conf/目录，相当于把Hadoop集群的配置信息告诉Nutch，

    $ cd ~/local/opt/hadoop-1.2.1/conf
    $ cp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml masters slaves /home/soulmachine/local/opt/apache-nutch-1.7/conf

重新编译，

    $ ant runtime

这会在`runtime/deploy`下生成一个Job文件，`apache-nutch-1.7.job`，它本质上是一个zip压缩包，可以打开看一下它里面的内容。可以看到它包含了很多编译好的class文件，以及从conf/目录下的拷贝出来的xml配置文件。

###1.3 向Hadoop集群提交Job，进行抓取
首先，要在con/hadoop-env.sh 添加`HADOOP_CLASSPATH`，让Hadoop知道去哪里找Nutch所依赖的jar包，

    export HADOOP_CLASSPATH=/home/soulmachine/local/opt/apache-nutch-1.7/runtime/local/lib/*.jar

上传种子URL列表，

    $ bin/hadoop fs -put ~/urls urls
    $ bin/hadoop fs -lsr

提交Job,

    $ bin/hadoop jar ../apache-nutch-1.7/runtime/deploy/apache-nutch-1.7.job urls -dir TestCrawl -depth 1 -topN 5

可以打开web页面监控job的进度，

* Jobtracer: http://localhost:50030
* Namenode: http://localhost:50070

把Nutch运行在伪分布式Hadoop集群上，比Standalone模式要好，因为可以通过web页面监控job。

###1.4 查看结果

    $ bin/hadoop fs -ls TestCrawl

    Found 3 items
    drwxr-xr-x   - soulmachine supergroup          0 2014-02-04 02:17 /user/soulmachine/TestCrawl/crawldb
    drwxr-xr-x   - soulmachine supergroup          0 2014-02-04 02:18 /user/soulmachine/TestCrawl/linkdb
    drwxr-xr-x   - soulmachine supergroup          0 2014-02-04 02:16 /user/soulmachine/TestCrawl/segments


##2 将Nutch跑在分布式Hadoop集群上
选择一台能够向Hadoop集群提交Job的机器，即它需要配置Hadoop客户端，见[Hadoop多用户的配置](http://www.yanjiuyanjiu.com/blog/20140203)

然后，在这台机器上进行操作，步骤与上面完全一模一样。只是Hadoop集群换了而已。

##参考资料
1. [Web Crawling and Data Mining with Apache Nutch 的第3.2节](http://packtlib.packtpub.com/library/web-crawling-and-data-mining-with-apache-nutch/ch03lvl1sec20)
1. [Install Nutch 1.7 and Hadoop 1.2.0](http://nutchhadoop.blogspot.com/)

##废弃的资料
1. [Nutch and Hadoop Tutorial](http://wiki.apache.org/nutch/NutchHadoopTutorial)，讲的是Nutch 1.3的，太老了，完全不适用Nutch 1.7

1. [Running Nutch in (pseudo) distributed-mode](http://wiki.apache.org/nutch/NutchHadoopSingleNodeTutorial)，太短了，没什么内容

