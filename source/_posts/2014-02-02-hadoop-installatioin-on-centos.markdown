---
layout: post
title: "在CentOS上安装Hadoop集群"
date: 2014-02-02 12:39
comments: true
categories: hadoop
---
Ubuntu上安装，请参考我的另一篇博客，[在Ubuntu上安装Hadoop](http://www.yanjiuyanjiu.com/blog/20120103/)。

**环境**：CentOS 6.5, OPenJDK 1.7, Hadoop 1.2.1

本文主要参考官网的文档，[Hadoop 1.2.1 Getting Started](http://hadoop.apache.org/docs/r1.2.1/#Getting+Started)

##1 单机模式(Standalone Mode)
为了能顺利安装成功，我们先练习在单台机器上安装Hadoop。在单台机器上，可以配置成单机模式(Standalone Mode)和伪分布式模式(Pseudo-Distributed Mode)，参考官方文档[Single Node Setup](http://hadoop.apache.org/docs/r1.2.1/single_node_setup.html)。

###1.1 下载Hadoop 1.2.1，解压
用浏览器下载或wget,

    $ wget http://mirror.olnevhost.net/pub/apache/hadoop/common/stable1/hadoop-1.2.1-bin.tar.gz
    $ tar -zxf hadoop-1.2.1-bin.tar.gz
    $ cd hadoop-1.2.1

###1.2 编辑 conf/hadoop-env.sh，设置 `JAVA_HOME`

    $ echo $JAVA_HOME
    /usr/lib/jvm/java
    $ vim conf/hadoop-env.sh

取消`JAVA_HOME`那一行的注释，设置正确的JDK位置

    export JAVA_HOME=/usr/lib/jvm/java

###1.3 运行一个job
默认情况下，Hadoop就被配置为Standalone模式了，此时Hadoop的所有模块都运行在一个java进程里。

我们运行一个例子测试一下。下面的几行命令，把 `conf`下的所有xml文件作为输入，在文件中查找指定的正则表达式，把匹配的结果输出到`output`目录。

    $ mkdir input 
    $ cp conf/*.xml input 
    $ bin/hadoop jar hadoop-examples-*.jar grep input output 'dfs[a-z.]+' 
    $ cat output/*

可以看到正常的结果，说明单机模式运行成功了，下面开始配置伪分布式模式。

<!-- more -->

##2 伪分布式模式(Pseudo-Distributed Mode)
Hadoop能在单台机器上以伪分布式模式运行，即每个Hadoop模块运行在单独的java进程里。


###2.1 编辑 conf/hadoop-env.sh，设置 `JAVA_HOME`

    $ echo $JAVA_HOME
    /usr/lib/jvm/java
    $ vim conf/hadoop-env.sh

###2.2 设置无密码SSH登录
先检查一下是能够无密码登录本机，

    ssh localhost

如果提示输入密码，说明不能，按如下步骤设置。

    $ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
    $ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

###2.3 配置

conf/core-site.xml:

    <configuration>
         <property>
             <name>fs.default.name</name>
             <value>hdfs://localhost:9000</value>
         </property>
    </configuration>

conf/hdfs-site.xml:

    <configuration>
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
    </configuration>



###2.4 启动Hadoop

格式化namenode

	$ bin/hadoop namenode -format

启动 Hadoop 后台进程

	$ bin/start-all.sh

Hadoop的log写入到了`${HADOOP_HOME}/logs`目录下。

现在可以用浏览器打开NameNode和JobTracker的web界面了。

> NameNode - <http://localhost:50070/>
> JobTracker - <http://localhost:50030/>

###2.5 运行一个例子
运行的例子跟单机模式下的例子相同。

将输入数据拷贝到分布式文件系统中:

    $ bin/hadoop fs -put conf input

运行 Hadoop 自带的例子:

    $ bin/hadoop jar hadoop-examples-*.jar grep input output 'dfs[a-z.]+'

查看输出文件:

    $ bin/hadoop fs -cat output/*

    1	dfs.replication
    1	dfs.server.namenode.
    1	dfsadmin

结束后，关闭 Hadoop:

    $ bin/stop-all.sh

    stopping jobtracker
    localhost: stopping tasktracker
    stopping namenode
    localhost: stopping datanode
    localhost: stopping secondarynamenode

##3 分布式模式(Fully-Distributed Mode)
主要参考官方文档[Cluster Setup](http://hadoop.apache.org/docs/r1.2.1/cluster_setup.html).

##3.1 准备3台机器
这里，我们用vmware workstation 创建3台虚拟机。首先用vmware workstation 新建一台CentOS 6.5，装好操作系统，选择 Basic Server，安装JDK，参考我的另一篇博客，[安装和配置CentOS服务器的详细步骤](http://www.yanjiuyanjiu.com/blog/20120423/)。安装好后然后用**浅拷贝**`Create a linked clone` 克隆出2台，这样有了3台虚拟机。启动3台机器，假设IP分别为`192.168.1.131, 192.168.1.132, 192.168.1.133`, 131做为NameNode,JobTracker和SecondaryNameNode，身兼3个角色，这3个角色应该放到3台不同的机器上，这里为了简化，用一台机器来做3个角色；132和133为 slave。三台机器上的用户名是`dev`。

##3.2 关闭防火墙
临时关闭防火墙

	$ sudo service iptables stop

下次开机后，防火墙还是会启动。

永久关闭防火墙

	$ sudo chkconfig iptables off

由于这几台虚拟机是开发机，不是生产环境，因此不必考虑安全性，可以永久关闭防火墙，还能给开发阶段带来很多便利。

##3.3 修改hostname
如果集群中的每一台机器事先已经有了hostname，则这一步可以跳过。

这一步看起来貌似不必要，其实是必须的，否则最后运行wordcount等例子时，会出现“Too many fetch-failures”。因为HDFS用hostname而不是IP，来相互之间进行通信（见后面的注意1）。

在CentOS上修改hostname，包含两个步骤(假设将hostname1改为hostname2，参考[这里](http://www.ichiayi.com/wiki/tech/linux_hostname)，但不需要第一步)：

1. 将 `/etc/sysconfig/network` 內的 HOSTNAME 改成 yourhostname
2. 用`hostname`命令，临时修改机器名， `sudo hostname yourhostname`

用`exit`命令退出shell，再次登录，命令提示字符串就会变成`[username@yourhostname ~]$`。

用上述方法，将131改名为master，132改名为slave01，133改名为slave02。

**注意**，对于有的Ubuntu/Debia 系统，会把本机的hostname解析成 127.0.1.1，例如：

    127.0.0.1       localhost
    127.0.1.1       master

将第二行改为(参考[利用Cloudera实现Hadoop](http://wiki.ubuntu.org.cn/%E5%88%A9%E7%94%A8Cloudera%E5%AE%9E%E7%8E%B0Hadoop))

    127.0.0.1       master

###3.4 修改所有机器的`/etc/hosts`文件
在所有机器的`/etc/hosts`文件中，添加所有hostname对应的IP，一般在一台机器上设置好，然后scp到所有机器。例如

	192.168.1.131 master
	192.168.1.132 slave01
	192.168.1.133 slave02

##3.5 配置 master 无密码登陆到所有机器（包括master自己登陆自己）
参考我的另一篇博客，[SSH无密码登录的配置](http://www.yanjiuyanjiu.com/blog/20120102/)

##3.6 把Hadoop压缩包上传到所有机器，并解压
将 hadoop-1.2.1-bin.tar.gz 上传到所有机器，然后解压。**注意，所有机器的hadoop路径必须一致，因为master会登陆到slave上执行命令，master认为slave的hadoop路径与自己一样。**

下面开始配置，配置好了后，把conf 目录scp到所有其他机器。


###3.7 修改6个配置文件
Hadoop的配置文件比较多，其设计原则可以概括为如下两点：

* 尽可能模块化。例如core-xxx.xml是针对基础公共库core的，hdfs-xxx.xml是针对分布式文件系统HDFS的，mapred-xxx.xml是针对分布式计算框架MapReduce的。
* 动静分离。例如，在Hadoop 1.0.0之前，作业队列权限管理相关的配置被放在mapred-site.xml里，而该文件爱你是不可以动态加载的，每次修改后必须重启Hadoop，但从 1.0.0后，这些配置选项被剥离出来放到独立的配置文件mapred-queue-acls.xml中，该文件可以通过Hadoop命令动态加载。在conf下，core-default.xml, hdfs-default.xml和mapred-default.xml是只读的，core-site.xml, hdfs-site.xml和mapred-site.xml才是用户可以修改的。要想覆盖默认配置，就在xxx-site.xml里修改。

在 master 上修改6个配置文件。

在 conf/hadoop-env.sh里，设置 `JAVA_HOME`。如果集群中，每台机器的JDK不一定统一安装在同一个路径，那就要在每个节点的hadoop-env.sh里分别设置`JAVA_HOME`。且要设置`HADOOP_PID_DIR`，这里我们令其为`HADOOP_PID_DIR=/home/dev/hadoop/pid`，参考[Hadoop的pid配置](http://www.iteye.com/topic/299219)。

注意，还要**禁用IPv6**，用命令`cat /proc/sys/net/ipv6/conf/all/disable_ipv6`检查一下系统是否启用了IPv6，如果是0,说明启用了。Hadoop在IPv6的情况下不正常，我们需禁用IPv6。不过我们不用真的禁用IPv6，让java选择IPv4即可，在conf/hadoop-env.sh 里添加如下一行，

    export HADOOP_OPTS="-server -Djava.net.preferIPv4Stack=true $HADOOP_OPTS"

参考[Disabling IPv6](http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/#disabling-ipv6)，以及Web Crawling and Data Mining with Apache Nutch这本书的第66页。

conf/masters:

	master

conf/slaves:

	slave01
	slave02

这里解释一下，masters文件，存放的其实是SecondaryNameNode。关于masters和slaves两个配置文件，更精确的说明见这个StackOverflow答案，[hadoop conf/masters and conf/slaves on jobtracker?](http://stackoverflow.com/a/19779590/381712)

conf/core-site.xml:

	<configuration>
	    <property>
	        <name>fs.default.name</name>
	        <value>hdfs://master:9000</value>
	    </property>
	    <property>
	        <name>fs.checkpoint.dir</name>
	        <value>/home/dev/hadoop/dfs/namesecondary</value>
	    </property>
	</configuration>

conf/hdfs-site.xml:

	<configuration>
	     <property>
	         <name>dfs.name.dir</name>
	         <value>/home/dev/hadoop/dfs/name</value>
	     </property>
	     <property>
	         <name>dfs.data.dir</name>
	         <value>/home/dev/hadoop/dfs/data</value>
	     </property>
             <property>
               <name>dfs.replication</name>
               <value>2</value>
             </property>
	</configuration>

我们只有2台slave，因此`dfs.replication`设置为2。

Hadoop会自动在master创建 /home/dev/hadoop/dfs/name 目录，在 slaves上创建 /home/dev/hadoop/dfs/data 目录。

conf/mapred-site.xml:

	<configuration>
	    <property>
	        <name>mapred.job.tracker</name>
	        <value>master:9001</value>
	    </property>
	    <property>
	        <name>mapred.local.dir</name>
	        <value>/home/dev/hadoop/mapred/local</value>
	    </property>
	    <property>
	        <name>mapred.system.dir</name>
	        <value>/home/dev/hadoop/mapred/system</value>
	    </property>
	</configuration>

###3.8 将配置文件拷贝到所有slaves

	$ cd ~/hadoop-1.2.1/conf/
	$ scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml masters slaves dev@slave01:~/hadoop-1.2.1/conf/
	$ scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml masters slaves dev@slave02:~/hadoop-1.2.1/conf/

###3.9 运行 hadoop
在master上执行以下命令，启动hadoop

	$ cd ~/hadoop-1.2.1/
	#只需一次，下次启动不再需要格式化，只需 start-all.sh
	$ bin/hadoop namenode -format
	$ bin/start-all.sh

###3.10 检查是否启动成功

在master上执行：

	$ jps
	
	2615 NameNode
	2767 JobTracker
	2874 Jps

在一台slave上执行：

	$ jps
	
	3415 DataNode
	3582 TaskTracker
	3499 SecondaryNameNode
	3619 Jps

在另一台slave上执行：

	$ jps
	
	3741 Jps
	3618 DataNode
	3702 TaskTracker

可见进程都启动起来了，说明hadoop运行成功。

###3.11 运行wordcount例子，进一步测试是否安装成功
将输入数据拷贝到分布式文件系统中:

	$ cd ~/hadoop-1.2.1/
	$ bin/hadoop fs -put conf input

运行 Hadoop 自带的例子:

	$ bin/hadoop jar hadoop-examples-*.jar wordcount input output

查看输出文件:

	$ bin/hadoop fs -ls output
	$ bin/hadoop fs -cat output/part-r-00000

如果能看到结果，说明这个例子运行成功。

###3.12 停止 hadoop集群
在master上执行：

	$ bin/stop-all.sh

###3.13 （可选）在master上设置环境变量HADOOP\_PREFIX，并将`$HADOOP_PREFIX/bin`加入PATH
这一步是为了将bin目录加入PATH，这样可以在任何位置执行hadoop的各种命令。这步是可选的。

Hadoop不推荐使用`$HADOOP_HOME`，你可以试一下，当设置了`$HADOOP_HOME`后，执行`bin/start-all.sh`，第一行会打印出一行警告信息，`Warning: $HADOOP_HOME is deprecated.` 应该用`$HADOOP_PREFIX`代替，见邮件列表里的这封[邮件](http://mail-archives.apache.org/mod_mbox/hadoop-common-user/201202.mbox/%3CCB4ECC21.33727%25evans@yahoo-inc.com%3E)。

给所有机器设置环境变量HADOOP\_PREFIX，并将`$HADOOP_PREFIX/bin`加入PATH。

	$ vim ~/.bashrc
	# add the following lines at the end
	export HADOOP_PREFIX=$HOME/hadoop-1.2.1
	export PATH=$PATH:$HOME/bin:$HADOOP_PREFIX/bin::$HADOOP_HOME/sbin
	export CLASSPATH=$CLASSPATH:$HADOOP_PREFIX/hadoop-core-1.2.1.jar
	# make the bash profile take effect immediately
	$ source ~/.bashrc

###3.14 （可选）在master上设置别名，名称为hadoop，指向bin/hadoop
像bin目录下的start-all.sh, stop-all.sh其实不常用，对于一个hadoop使用者而言，最常用的命令是hadoop，例如`hadoop fs -ls`。前面5.4节将bin目录加入PATH，相当于一股脑儿将所有的命令加入了PATH，其实大可不必，我们只需要设置一个别名，名称为hadoop，指向bin/hadoop就可以了。

在master上设置hadoop 别名，步骤如下：

	$ vim ~/.bashrc
	# add the following line at the end
	alias hadoop='~/hadoop-1.2.1/bin/hadoop'
	#make the bash profile take effect immediately
	$ source ~/.bashrc

##4. 排除错误
本文已经尽可能的把步骤详细列出来了，但是我相信大部分人不会一次成功。这时候，查找错误就很重要了。查找错误最重要的手段是查看hadoop的日志，一般在logs目录下。把错误消息复制粘贴到google，搜索一下，慢慢找错误。

##注意
1. 所有配置文件只能用hostname，不能用IP。两年前我不懂，还为此[在stackoverflow上发了帖子](http://stackoverflow.com/questions/8702637/hadoop-conf-fs-default-name-cant-be-setted-ipport-format-directly)。hadoop会反向解析hostname，即使是用了IP，也会使用hostname 来启动TaskTracker。参考[hdfs LAN ip address hostname resolution](http://stackoverflow.com/questions/15230946/hdfs-lan-ip-address-hostname-resolution)，[hadoop入门经验总结- 杨贵堂的博客](http://www.makenotes.net/?p=337004)，[hadoop集群配置](http://51mst.iteye.com/blog/1152439)。
1. 在第2.5步骤，如果出现 `SafeModeException` 异常，不用担心，等待几分钟即可。因为hadoop刚刚启动时，会进入安全模式进行自检，这需要花点时间。
1. 如果在任何一步失败，可以`stop-all.sh`, 然后`hadoop  namenode -format`，重试几次，一般可以成功。如果还是不成功，多看看 logs目录下的日志文件，把错误消息复制粘贴到google，搜索答案。


