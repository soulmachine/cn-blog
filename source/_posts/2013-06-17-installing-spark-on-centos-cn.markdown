---
layout: post
title: "安装Spark集群(在CentOS上)"
date: 2013-06-17 22:16
comments: true
categories: spark
---
**环境**:CentOS 6.4, Hadoop 1.1.2, JDK 1.7, Spark 0.7.2, Scala 2.9.3

折腾了几天，终于把Spark 集群安装成功了，其实比hadoop要简单很多，由于网上搜索到的博客大部分都还停留在需要依赖mesos的版本，走了不少弯路。


#1. 安装 JDK 1.7
	yum search openjdk-devel
	sudo yum install java-1.7.0-openjdk-devel.x86_64
	/usr/sbin/alternatives --config java
	/usr/sbin/alternatives --config javac
	sudo vim /etc/profile
	# add the following lines at the end
	export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.19.x86_64
	export JRE_HOME=$JAVA_HOME/jre
	export PATH=$PATH:$JAVA_HOME/bin
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
	# save and exit vim
	# make environment variables take effect immediately
	$ source /etc/profile
	# test
	$ java -version

参考我的另一篇博客，[安装和配置CentOS服务器的详细步骤](http://www.yanjiuyanjiu.com/blog/20120423/)。

#2. 安装 Scala 2.9.3
Spark 0.7.2 依赖 Scala 2.9.3, 我们必须要安装Scala 2.9.3.

下载 [scala-2.9.3.tgz](http://www.scala-lang.org/downloads/distrib/files/scala-2.9.3.tgz) 并 保存到home目录.

	$ tar -zxf scala-2.9.3.tgz
	$ sudo mv scala-2.9.3 /usr/lib
	$ sudo vim /etc/profile
	# add the following lines at the end
	export SCALA_HOME=/usr/lib/scala-2.9.3
	export PATH=$PATH:$SCALA_HOME/bin
	# save and exit vim
	# make environment variables take effect immediately
	source /etc/profile
	# test
	$ scala -version

#3. 下载预编译好的Spark
下载预编译好的Spark, [spark-0.7.2-prebuilt-hadoop1.tgz](http://www.spark-project.org/download-spark-0.7.2-prebuilt-hadoop1). 

如果你想从零开始编译，则下载源码包，但是我不建议你这么做，因为有一个Maven仓库，twitter4j.org, 被墙了，导致编译时需要翻墙，非常麻烦。如果你有DIY精神，并能顺利翻墙，则可以试试这种方式。

#4. 本地模式

##4.1 解压

	$ tar -zxf spark-0.7.2-prebuilt-hadoop1.tgz
	
##4.2 设置SPARK\_EXAMPLES\_JAR 环境变量

	$ vim ~/.bash_profile
	# add the following lines at the end
	export SPARK_EXAMPLES_JAR=$SPARK_HOME/examples/target/scala-2.9.3/spark-examples_2.9.3-0.7.2.jar
	# save and exit vim
	# make environment variables take effect immediately
	$ source /etc/profile

这一步其实最关键，很不幸的是，官方文档和网上的博客，都没有提及这一点。我是偶然看到了这两篇帖子，[Running SparkPi](https://groups.google.com/forum/?fromgroups#!topic/spark-users/nQ6wB2lcFN8), [Null pointer exception when running ./run spark.examples.SparkPi local](https://groups.google.com/forum/#!msg/spark-users/x5UczgI-Xm8/wzMm3Mb77-oJ)，才补上了这一步，之前死活都无法运行SparkPi。

##4.3 （可选）设置 SPARK\_HOME环境变量，并将SPARK\_HOME/bin加入PATH
	$ vim ~/.bash_profile
	# add the following lines at the end
	export SPARK_HOME=$HOME/spark-0.7.2
	export PATH=$PATH:$SPARK_HOME/bin
	# save and exit vim
	# make environment variables take effect immediately
	$ source /etc/profile

##4.4 现在可以运行SparkPi了

	$ cd ~/spark-0.7.2
	$ ./run spark.examples.SparkPi local 

#5. 集群模式

<!-- more -->

##5.1 安装Hadoop
用VMware Workstation 创建三台CentOS 虚拟机，hostname分别设置为 master, slave01, slave02，设置SSH无密码登陆，安装hadoop，然后启动hadoop集群。参考我的这篇博客，[在CentOS上安装Hadoop](http://www.yanjiuyanjiu.com/blog/20130612). 

##5.2 Scala
在三台机器上都要安装 Scala 2.9.3 , 按照第2节的步骤。JDK在安装Hadoop时已经安装了。

##5.3 在master上安装并配置Spark
解压

	$ tar -zxf spark-0.7.2-prebuilt-hadoop1.tgz

设置SPARK\_EXAMPLES\_JAR 环境变量

	$ vim ~/.bash_profile
	# add the following lines at the end
	export SPARK_EXAMPLES_JAR=$SPARK_HOME/examples/target/scala-2.9.3/spark-examples_2.9.3-0.7.2.jar
	# save and exit vim
	# make environment variables take effect immediately
	$ source /etc/profile

在 in `conf/spark-env.sh` 中设置`SCALA_HOME`

	$ cd $SPARK_HOME/conf
	$ mv spark-env.sh.template spark-env.sh
	$ vim spark-env.sh
	export SCALA_HOME=/usr/lib/scala-2.9.3
	# save and exit

在`conf/slaves`, 添加Spark worker的hostname, 一行一个。

	$ vim slaves
	slave01
	slave02
	# save and exit

（可选）设置 SPARK\_HOME环境变量，并将SPARK\_HOME/bin加入PATH

	$ vim ~/.bash_profile
	# add the following lines at the end
	export SPARK_HOME=$HOME/spark-0.7.2
	export PATH=$PATH:$SPARK_HOME/bin
	# save and exit vim
	# make environment variables take effect immediately
	$ source /etc/profile

##5.4 在所有worker上安装并配置Spark
既然master上的这个文件件已经配置好了，把它拷贝到所有的worker。**注意，三台机器spark所在目录必须一致，因为master会登陆到worker上执行命令，master认为worker的spark路径与自己一样。**

	scp -r spark-0.7.2 dev@slave01:~
	scp -r spark-0.7.2 dev@slave02:~

按照第5.3节设置环境变量和配置文件。


##5.5 启动 Spark 集群
在master上执行

	$ cd ~/spark-0.7.2
	$ bin/start-all.sh

检测进程是否启动

	$ jps
	11055 Jps
	2313 SecondaryNameNode
	2409 JobTracker
	2152 NameNode
	4822 Master

浏览master的web UI(默认<http://localhost:8080>). 这是你应该可以看到所有的word节点，以及他们的CPU个数和内存等信息。
##5.6 运行SparkPi例子

	$ cd ~/spark-0.7.2
	$ ./run spark.examples.SparkPi spark://master:7077

##5.7 从HDFS读取文件并运行WordCount

	$ cd ~/spark-0.7.2
	$ hadoop fs -put README.md .
	$ ./spark-shell
	scala> val file = sc.textFile("hdfs://master:9000/user/dev/README.md")
	scala> val count = file.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_+_)
	scala> count.collect()

##5.8 停止 Spark 集群

	$ cd ~/spark-0.7.2
	$ bin/stop-all.sh

#参考资料
1. [Spark Standalone Mode](http://spark-project.org/docs/latest/spark-standalone.html)
1. [Running A Spark Standalone Cluster](https://github.com/mesos/spark/wiki/Running-A-Spark-Standalone-Cluster)
1. [Lightning-Fast WordCount using Spark Alongside Hadoop](http://sprism.blogspot.com/2012/11/lightning-fast-wordcount-using-spark.html)

以下博客都已经过时了：

1. [Installing Spark on Fedora 18](http://chapeau.freevariable.com/2013/04/installing-spark-on-fedora-18.html)
1. [Spark随谈（二）—— 安装攻略](http://rdc.taobao.com/team/jm/archives/1823)
1. [Spark安装与学习](http://www.cnblogs.com/jerrylead/archive/2012/08/13/2636115.html)