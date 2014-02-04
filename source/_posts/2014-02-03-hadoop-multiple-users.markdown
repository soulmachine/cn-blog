---
layout: post
title: "Hadoop多用户的配置"
date: 2014-02-03 10:05
comments: true
categories: hadoop
---
假设我们以名为hadoop的用户，建好了集群，见[在CentOS上安装Hadoop集群](http://www.yanjiuyanjiu.com/blog/20140202)。通常，我们会把这个集群共享给多个用户，而不是让大家都登录为hadoop，这样做有几个好处：

* 一个用户不能修改另一个用户的的文件
* 在hadoop web管理页面，可以很方便的看到不同的用户的job

现在集群中有一台机器，上面有一个用户名为 hbase 的用户，他想要使用hadoop集群，怎么配置呢？

##1. 安装hadoop客户端

###1.1 下载，解压
下载跟hadoop集群一样的hadoop软件包，并解压，

    $ wget http://mirror.olnevhost.net/pub/apache/hadoop/common/stable1/hadoop-1.2.1-bin.tar.gz
    $ tar -zxf hadoop-1.2.1-bin.tar.gz -C ~/local/opt
    $ cd ~/local/opt/hadoop-1.2.1

###1.2 配置
在客户端只需配置集群namenode 和 jobtracker 的相关信息, 把namenode和jobtracker的信息告诉客户端即可。

把hadoop用户下的`conf/core-site.xml`和`conf/mapred-site.xml`拷贝到本用户的conf/目录下


    $ scp hadoop@localhost:~/local/opt/hadoop-1.2.1/conf/core-site.xml ./conf/
    $ scp hadoop@localhost:~/local/opt/hadoop-1.2.1/conf/mapred-site.xml ./conf/

<!-- more -->

##2. 在master上配置权限
以下操作均在hadoop集群的 namenode 这台机器上进行，且登录为hadoop，因为hadoop这个用户是整个hadoop集群权限最高的用户（但对于Linux系统本身，这个用户其实没有sudo权限）。

Hadoop关于用户权限方面，有很多高级的配置，这里我们简单的利用HDFS本身的文件权限检查机制，来配置多用户。

HDFS本身没有提供用户名、用户组的创建，在客户端调用hadoop 的文件操作命令时，hadoop 识别出执行命令所在进程的linux系统的用户名和用户组，然后使用这个用户名和组来检查文件权限。 用户名=linux命令中的`whoami`，而组名等于`groups`。 

启动hadoop hdfs系统的用户即为超级用户（在这里就是名为hadoop的这个用户），可以进行任意的操作。


###2.1 为客户端用户创建文件夹

    # 以hadoop用户登录namenode机器
    $ bin/hadoop fs -mkdir /user/hbase
    $ bin/hadoop fs -chown hbase /user/hbase
    # 在客户端机器上，用gropus命令看一下hbase所在的组
    $ bin/hadoop fs -chgrp hbase /user/hbase

###2.2 设置mapreduce.jobtracker.staging.root.dir，然后重启hadoop集群
客户端向集群提交任务时，需要把该job需要的文件打包，拷贝到HDFS上。在拷贝之前，得先确定这些资源文件存放在HDFS的什么地方。JobTracker设置有一个工作目录(Staging area, 也称数据中转站)，用来存储与每个job相关的数据。这个目录的前缀由`mapreduce.jobtracker.staging.root.dir` 参数来指定，默认是`${hadoop.tmp.dir}/mapred/staging`，每个client user可以提交多个job，在这个目录后就得附加user name的信息。所以这个工作目录(Staging area)默认是`${hadoop.tmp.dir}/mapred/staging/denny/.staging/`。

一般把前缀设置为`/user`，这是官方推荐的，见 <http://hadoop.apache.org/docs/r1.2.1/mapred-default.html> 里的`mapreduce.jobtracker.staging.root.dir`处：

> The root of the staging area for users' job files In practice, this should be the directory where users' home directories are located (usually /user)

    #以hadoop用户登录jobtracker机器
    $ vim conf/mapred-site.xml
    <property>
      <name>mapreduce.jobtracker.staging.root.dir</name>
      <value>/user</value>
    </property>
    $ bin/stop-all.sh
    $ bin/start-all.sh

##3. 测试一下
回到客户端机器。

将输入数据拷贝到分布式文件系统中:

    $ bin/hadoop fs -put conf input
    $ bin/hadoop fs -ls input

运行 Hadoop 自带的例子:

    $ bin/hadoop jar hadoop-examples-*.jar wordcount input output

查看输出文件:

    $ bin/hadoop fs -ls output
    $ bin/hadoop fs -cat output/part-r-00000

如果能看到结果，说明这个例子运行成功。

###4 （可选）设置别名，名称为hadoop，指向bin/hadoop
这样就不用每次都cd到Hadoop目录，执行命令了。

在 `~/.bashrc`中添加如下4行：

	unalias hadoop &> /dev/null
	alias hadoop="$HOME/local/opt/hadoop-1.2.1/bin/hadoop"
	unalias hls &> /dev/null
	alias hls="hadoop fs -ls"

source使之立刻生效，

	$ source ~/.bashrc

##参考资料
[hadoop远程客户端安装配置、多用户权限配置](http://blog.csdn.net/j3smile/article/details/7887826)

[hadoop如何创建多用户](http://blog.csdn.net/a999wt/article/details/8718707)

[关于多用户时hadoop的权限问题](http://blog.sina.com.cn/s/blog_605f5b4f0101897z.html)

[MapReduce: Job提交过程](http://langyu.iteye.com/blog/909170)

