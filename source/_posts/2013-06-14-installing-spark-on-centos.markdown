---
layout: post
title: "Installing Spark on CentOS"
date: 2013-06-14 19:06
comments: true
categories: spark
---
**Environment**:CentOS 6.4, Hadoop 1.1.2, JDK 1.7, Spark 0.7.2, Scala 2.9.3

After a few days hacking , I have found that installing a Spark cluster is exteremely easy :)

#1. Install JDK 1.7
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

#2. Install Scala 2.9.3
Spark 0.7.2 depends on Scala 2.9.3, So we must install Scala of version 2.9.3.

Download [scala-2.9.3.tgz](http://www.scala-lang.org/downloads/distrib/files/scala-2.9.3.tgz) and save it to home directory.

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

#3. Download prebuilt packages
Download prebuilt packages, [spark-0.7.2-prebuilt-hadoop1.tgz](http://www.spark-project.org/download-spark-0.7.2-prebuilt-hadoop1). 

If you want to compile it from scratch, download the source package, but I don’t recommend this way, because in Chinese Mainland the GFW has blocked one of maven repositories, twitter4j.org, which makes the compilation an impossible mission unless you can conquer GFW.

#4. Local Mode

##4.1 Untar the tarball

	$ tar -zxf spark-0.7.2-prebuilt-hadoop1.tgz

##4.2 Set the SPARK\_EXAMPLES\_JAR environment variable
	$ vim ~/.bash_profile
	# add the following lines at the end
	export SPARK_EXAMPLES_JAR=$SPARK_HOME/examples/target/scala-2.9.3/spark-examples_2.9.3-0.7.2.jar
	# save and exit vim
	# make environment variables take effect immediately
	$ source /etc/profile

This is the most important step that must be done , but unfortunately the official docs and most web blogs haven’t mentioned this. I found this step when I bumped into these posts, [Running SparkPi](https://groups.google.com/forum/?fromgroups#!topic/spark-users/nQ6wB2lcFN8), [Null pointer exception when running ./run spark.examples.SparkPi local](https://groups.google.com/forum/#!msg/spark-users/x5UczgI-Xm8/wzMm3Mb77-oJ).

##4.3 (Optional)Set SPARK\_HOME and add SPARK\_HOME/bin to PATH

	$ vim ~/.bash_profile
	# add the following lines at the end
	export SPARK_HOME=$HOME/spark-0.7.2
	export PATH=$PATH:$SPARK_HOME/bin
	# save and exit vim
	# make environment variables take effect immediately
	$ source /etc/profile

##4.4 Now you can run SparkPi.

	$ cd ~/spark-0.7.2
	$ ./run spark.examples.SparkPi local 

#5. Cluster Mode

<!-- more -->

##5.1 Install hadoop
Use VMware Workstation to create three CentOS virtual machines, which's hostnames are master, slave01, slave02, setup password-less ssh to the slaves, install hadoop on the three machines and start up the hadoop cluster. For more details please read another blog of mine, [在CentOS上安装Hadoop](http://www.yanjiuyanjiu.com/blog/20130612).

##5.2 Install JDK and Scala
Install JDK 1.7 and Scala 2.9.3 on the three machines, according to section 1 and section 2.

##5.3 Install and configure Spark on master
Untar

	$ tar -zxf spark-0.7.2-prebuilt-hadoop1.tgz

Set the SPARK\_EXAMPLES\_JAR environment variable

	$ vim ~/.bash_profile
	# add the following lines at the end
	export SPARK_EXAMPLES_JAR=$SPARK_HOME/examples/target/scala-2.9.3/spark-examples_2.9.3-0.7.2.jar
	# save and exit vim
	# make environment variables take effect immediately
	$ source /etc/profile

Set `SCALA_HOME` in `conf/spark-env.sh`

	$ cd $SPARK_HOME/conf
	$ mv spark-env.sh.template spark-env.sh
	$ vim spark-env.sh
	export SCALA_HOME=/usr/lib/scala-2.9.3
	# save and exit

In`conf/slaves`, add hostnames of Spark workers, one per line.

	$ vim slaves
	slave01
	slave02
	# save and exit

(Optional)Set SPARK\_HOME and add SPARK\_HOME/bin to PATH

	$ vim ~/.bash_profile
	# add the following lines at the end
	export SPARK_HOME=$HOME/spark-0.7.2
	export PATH=$PATH:$SPARK_HOME/bin
	# save and exit vim
	# make environment variables take effect immediately
	$ source /etc/profile

##5.4 Install and configure Spark on workers
Copy the spark directory to all slaves. **Remark，the spark directories must locat at the the same path on all machines，because the master will login to work to execute spark commands, it assumes that workers have the same path as itself**

	scp -r spark-0.7.2 dev@slave01:~
	scp -r spark-0.7.2 dev@slave02:~

Set environment variables on all slaves as section 5.3.


##5.5 Start Spark cluster
On master

	$ cd ~/spark-0.7.2
	$ bin/start-all.sh

Check whether the processes have been started.

	$ jps
	11055 Jps
	2313 SecondaryNameNode
	2409 JobTracker
	2152 NameNode
	4822 Master

Look at the master’s web UI (<http://localhost:8080> by default). You should see the new node listed there, along with its number of CPUs and memory (minus one gigabyte left for the OS).

##5.6 run the SparkPi example in cluster mode

	$ cd ~/spark-0.7.2
	$ ./run spark.examples.SparkPi spark://master:7077

##5.7 read files from HDFS and run WordCount

	$ cd ~/spark-0.7.2
	$ hadoop fs -put README.md .
	$ ./spark-shell
	scala> val file = sc.textFile("hdfs://master:9000/user/dev/README.md")
	scala> val count = file.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_+_)
	scala> count.collect()

##5.8 Stop Spark cluster

	$ cd ~/spark-0.7.2
	$ bin/stop-all.sh

#References
1. [Spark Standalone Mode](http://spark-project.org/docs/latest/spark-standalone.html)
1. [Running A Spark Standalone Cluster](https://github.com/mesos/spark/wiki/Running-A-Spark-Standalone-Cluster)
1. [Lightning-Fast WordCount using Spark Alongside Hadoop](http://sprism.blogspot.com/2012/11/lightning-fast-wordcount-using-spark.html)

The following posts are outdated.

1. [Installing Spark on Fedora 18](http://chapeau.freevariable.com/2013/04/installing-spark-on-fedora-18.html)
1. [Spark随谈（二）—— 安装攻略](http://rdc.taobao.com/team/jm/archives/1823)
1. [Spark安装与学习](http://www.cnblogs.com/jerrylead/archive/2012/08/13/2636115.html)
