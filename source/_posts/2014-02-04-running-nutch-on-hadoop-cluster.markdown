---
layout: post
title: "把Nutch爬虫部署到Hadoop集群上"
date: 2014-02-04 22:36
comments: true
categories: Search-Engine
---

软件版本：Nutch 1.7, Hadoop 1.2.1, CentOS 6.5, JDK 1.7

前面的3篇文章中，[Nutch 快速入门(Nutch 1.7)](http://www.yanjiuyanjiu.com/blog/20140121)，[Nutch 快速入门(Nutch 2.2.1)](http://www.yanjiuyanjiu.com/blog/20140201)，[在Eclipse里运行Nutch](http://www.yanjiuyanjiu.com/blog/20140120)，Nutch都是跑在单机上，本文把Nutch部署到Hadoop集群上，在真正的分布式Hadoop集群上跑。


##前提

* 学会了搭建一个分布式Hadoop集群，见[在CentOS上安装Hadoop集群](http://www.yanjiuyanjiu.com/blog/20140202)
* 学会了单机跑Nutch，见[Nutch 快速入门(Nutch 1.7)](http://www.yanjiuyanjiu.com/blog/20140121)

##1 启动Hadoop集群
伪分布式或真分布式的Hadoop集群都可以，无所谓。

选择一台配置好了的Hadoop客户端的机器（见[Hadoop多用户的配置](http://www.yanjiuyanjiu.com/blog/20140203)），作为客户机，以下操作均在这台客户机上进行。

##2 下载Nutch源码
有两种方法，

1. 去官网首页下载apache-nutch-1.7-src.tar.gz
1. 用svn checkout

        $ svn co https://svn.apache.org/repos/asf/nutch/tags/release-1.7

##3 把Hadoop的6个配置文件拷贝到Nutch的conf/目录
将Hadoop的六个配置文件，拷贝到Nutch的conf/目录，相当于把Hadoop集群的配置信息告诉Nutch，

<!-- more -->

在伪分布式模式下，

    $ cd ~/local/opt/hadoop-1.2.1/conf
    $ cp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml masters slaves /home/soulmachine/local/src/apache-nutch-1.7/conf

在分布式模式下，

    $ ssh hadoop@localhost
    $ cd ~/local/opt/hadoop-1.2.1/conf
    $ scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml masters slaves soulmachine@localhost:~/local/src/apache-nutch-1.7/conf
    $ exit


##4 修改Nutch的配置文件
修改 conf/nutch-site.xml:

    <property>
      <name>http.agent.name</name>
      <value>My Nutch Spider</value>
    </property>

修改 `regex-urlfilter.txt`, 见[Nutch 快速入门(Nutch 1.7)](http://www.yanjiuyanjiu.com/blog/20140121/) 第4节，

    #注释掉这一行
    # skip URLs containing certain characters as probable queries, etc.
    #-[?*!@=]
    # accept anything else
    #注释掉这行
    #+.
    +^http:\/\/movie\.douban\.com\/subject\/[0-9]+\/(\?.+)?$

##5 重新编译Nutch
每次修改了`$NUTCH_HOME/conf`下的的文件，都需要重新编译Nutch，重新打包生成一个nutch-x.x.x.job文件，见这里，[Running Nutch in (pseudo) distributed-mode](http://wiki.apache.org/nutch/NutchHadoopSingleNodeTutorial)。也可以打开build.xml看看里面的"runtime"这个task干了什么，就明白了。

    $ ant runtime

这会在`runtime/deploy`下生成一个Job文件，`apache-nutch-1.7.job`，它本质上是一个zip压缩包，可以打开看一下它里面的内容。可以看到它包含了很多编译好的class文件，以及从conf/目录下的拷贝出来的xml配置文件。

##6 向Hadoop集群提交Job，进行抓取

首先，要在con/hadoop-env.sh 添加`HADOOP_CLASSPATH`，让Hadoop知道去哪里找Nutch所依赖的jar包，

    export HADOOP_CLASSPATH=/home/soulmachine/local/opt/apache-nutch-1.7/runtime/local/lib/*.jar

上传种子URL列表，

    $ hadoop fs -put ~/urls urls
    $ hadoop fs -lsr urls

提交Job,

    $ hadoop jar ./runtime/deploy/apache-nutch-1.7.job org.apache.nutch.crawl.Crawl urls -dir TestCrawl -depth 1 -topN 5

可以打开web页面监控job的进度，

* Jobtracer: <http://master:50030>
* Namenode: <http://master:50070>

把Nutch运行在伪分布式Hadoop集群上，比Standalone模式要好，因为可以通过web页面监控job。

查看结果

    $ hadoop fs -ls TestCrawl

    Found 3 items
    drwxr-xr-x   - soulmachine supergroup          0 2014-02-04 02:17 /user/soulmachine/TestCrawl/crawldb
    drwxr-xr-x   - soulmachine supergroup          0 2014-02-04 02:18 /user/soulmachine/TestCrawl/linkdb
    drwxr-xr-x   - soulmachine supergroup          0 2014-02-04 02:16 /user/soulmachine/TestCrawl/segments

##7 注意
如果出现`java.io.IOException: No valid local directories in property: mapred.local.dir`的错误，说明你的客户机的mapred-site.xml是从hadoop集群拷贝过来的，没有修改过，`mapred.local.dir`是一个本地目录，集群上的机器有这个目录，但是你的本机上没有，所以出现了这个错误。解决办法是，在本地新建一个目录，然后把`mapred.local.dir`设置为这个路径。

如果出现`org.apache.hadoop.security.AccessControlException: Permission denied: user=soulmachine, access=WRITE, inode="tmp"`的错误，多半是因为你没有给这个用户创建`hadoop.tmp.dir`文件夹，见[Hadoop多用户的配置](http://www.yanjiuyanjiu.com/blog/20140203)第2.2节。


##参考资料
1. [Web Crawling and Data Mining with Apache Nutch 的第3.2节](http://packtlib.packtpub.com/library/web-crawling-and-data-mining-with-apache-nutch/ch03lvl1sec20)
1. [Install Nutch 1.7 and Hadoop 1.2.0](http://nutchhadoop.blogspot.com/)

1. [hadoop mapred(hive)执行目录 文件权限问题](http://blog.csdn.net/azhao_dn/article/details/6921398)


##废弃的资料
1. [Nutch and Hadoop Tutorial](http://wiki.apache.org/nutch/NutchHadoopTutorial)，讲的是Nutch 1.3的，太老了，完全不适用Nutch 1.7

1. [Running Nutch in (pseudo) distributed-mode](http://wiki.apache.org/nutch/NutchHadoopSingleNodeTutorial)，太短了，没什么内容
