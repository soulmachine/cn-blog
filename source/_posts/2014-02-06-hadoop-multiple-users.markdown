---
layout: post
title: "Hadoop多用户的配置(Hadoop 2.x)"
date: 2014-02-06 10:05
comments: true
categories: Hadoop
---
假设我们以名为hadoop的用户，建好了集群，见[在CentOS上安装Hadoop 2.x 集群](http://www.yanjiuyanjiu.com/blog/20140205/)。通常，我们会把这个集群共享给多个用户，而不是让大家都登录为hadoop，这样做有几个好处：

* 一个用户不能修改另一个用户的的文件
* 在hadoop web管理页面，可以很方便的看到不同的用户的job

现在集群中有一台机器，上面有一个用户名为 hbase 的用户，他想要使用hadoop集群，怎么配置呢？

##1. 安装hadoop客户端

###1.1 下载，解压
下载跟hadoop集群一样的hadoop软件包，并解压，

    $ wget http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.2.0/hadoop-2.2.0.tar.gz
    $ tar -zxf hadoop-2.2.0.tar.gz -C ~/local/opt
    $ cd ~/local/opt/hadoop-2.2.0

###1.2 拷贝Hadoop集群的配置文件
将Hadoop集群的配置文件全部拷贝到客户端，相当与把集群的信息告诉客户端。

    $ scp -r hadoop@localhost:~/local/opt/hadoop-2.2.0/etc/hadoop ./etc/

修改conf/mapred-site.xml中的`mapreduce.cluster.local.dir`，改为本机上的某个目录，确保这个目录存在且有写权限。因为这个目录是本地目录，每台机器都可以不同。例如我的是：

    <property>
      <name>mapreduce.cluster.local.dir</name>
      <value>/home/soulmachine/local/var/hadoop/mapred/local</value>
    </property>

确保这个目录存在，

    $ mkdir -p ~/local/var/hadoop/mapred/local

<!-- more -->

还有另一种方法，由于`mapreduce.cluster.local.dir`默认值是`${hadoop.tmp.dir}/mapred/local`，也可以通过修改`hadoop.tmp.dir`达到目的，在`core-site.xml`中，确保`${hadoop.tmp.dir}/mapred/local`存在且有写权限。


##2. 在master上配置权限
以下操作均在hadoop集群的 namenode 这台机器上进行，且登录为hadoop，因为hadoop这个用户是整个hadoop集群权限最高的用户（但对于Linux系统本身，这个用户其实没有sudo权限）。

Hadoop关于用户权限方面，有很多高级的配置，这里我们简单的利用HDFS本身的文件权限检查机制，来配置多用户。

HDFS本身没有提供用户名、用户组的创建，在客户端调用hadoop 的文件操作命令时，hadoop 识别出执行命令所在进程的linux系统的用户名和用户组，然后使用这个用户名和组来检查文件权限。 用户名=linux命令中的`whoami`，而组名等于`groups`。 

启动hadoop hdfs系统的用户即为超级用户（在这里就是名为hadoop的这个用户），可以进行任意的操作。

在客户端机器上，用gropus命令看一下hbase所在的组，

    $ groups
    hbase

说明hbase这个用户所在的组为hbase。

###2.1 为客户端用户创建home文件夹

    $ hdfs dfs -mkdir /user/hbase
    $ hdfs dfs -chown hbase /user/hbase
    $ hdfs dfs -chgrp hbase /user/hbase

###2.2 设置HDFS上的/tmp目录的权限
客户端提交job的时候，需要往/tmp里写入文件，因此最好把/tmp设置为所有用户都有可读、可写和可执行的权限。 

    $ hdfs dfs -chmod -R 777 /tmp

###2.3 设置mapreduce.jobtracker.staging.root.dir
客户端向集群提交任务时，需要把该job需要的文件打包，拷贝到HDFS上。在拷贝之前，得先确定这些资源文件存放在HDFS的什么地方。JobTracker设置有一个工作目录(Staging area, 也称数据中转站)，用来存储与每个job相关的数据。这个目录的前缀由`mapreduce.jobtracker.staging.root.dir` 参数来指定，默认是`${hadoop.tmp.dir}/mapred/staging`，每个client user可以提交多个job，在这个目录后就得附加user name的信息。所以这个工作目录(Staging area)默认是`${hadoop.tmp.dir}/mapred/staging/denny/.staging/`。

一般把前缀设置为`/user`，这是官方推荐的，见 [mapred-default.xml](http://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml) 里的`mapreduce.jobtracker.staging.root.dir`处：

> The root of the staging area for users' job files In practice, this should be the directory where users' home directories are located (usually /user)

    #以hadoop用户登录jobtracker机器
    $ vim conf/mapred-site.xml
    <property>
      <name>mapreduce.jobtracker.staging.root.dir</name>
      <value>/user</value>
    </property>

###2.4 重启hadoop集群
将配置文件scp到所有机器，然后重启集群，

    $ ./sbin/stop-yarn.sh
    $ ./sbin/start-yarn.sh
    $ ./sbin/stop-dfs.sh
    $ ./sbin/start-dfs.sh

##3. 测试一下
回到客户端机器。

将输入数据拷贝到分布式文件系统中:

    $ ./bin/hdfs dfs -put /etc/hadoop input
    $ ./bin/hdfs dfs -ls input

运行 Hadoop 自带的例子:

    $ ./bin/$ hadoop jar $HADOOP_PREFIX/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar wordcount input output

查看输出文件:

    $ hdfs dfs -lsr output
    $ hdfs dfs -cat output/part-r-00000

如果能看到结果，说明这个例子运行成功。

###4 （可选）将bin目录加入PATH
这样就不用每次都cd到Hadoop目录，执行命令了。

在 `~/.bashrc`中添加如下4行：

    export HADOOP_PREFIX=$HOME/local/opt/hadoop-2.2.0
    export PATH=$PATH:$HADOOP_PREFIX/bin
    alias hls="hdfs dfs -ls"

source使之立刻生效，

	$ source ~/.bashrc

##参考资料

1. [hadoop远程客户端安装配置、多用户权限配置](http://blog.csdn.net/j3smile/article/details/7887826)
1. [hadoop如何创建多用户](http://blog.csdn.net/a999wt/article/details/8718707)
1. [关于多用户时hadoop的权限问题](http://blog.sina.com.cn/s/blog_605f5b4f0101897z.html)
1. [MapReduce: Job提交过程](http://langyu.iteye.com/blog/909170)
1. [hadoop中的dfs.name.dir,mapred.local.dir,mapred.system.dir和hadoop.tmp.dir说明](http://www.hadoopor.com/archiver/tid-481.html)
1. [Hadoop 參數設定 – mapred-site.xml](http://fenriswolf.me/2012/08/06/hadoop-%E5%8F%83%E6%95%B8%E8%A8%AD%E5%AE%9A-mapred-site-xml/)

