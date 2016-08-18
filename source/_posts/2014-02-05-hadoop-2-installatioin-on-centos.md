---
layout: post
title: "在CentOS上安装Hadoop 2.x 集群"
date: 2014-02-05 12:39
comments: true
categories: Hadoop
---

**环境**：CentOS 6.5, OPenJDK 1.7, Hadoop 2.2.0

本文主要参考官网的文档，[Hadoop 2.2.0 Single Node Setup](http://hadoop.apache.org/docs/r2.2.0/hadoop-project-dist/hadoop-common/SingleCluster.html)， [Hadoop 2.2.0  Cluster Setup](http://hadoop.apache.org/docs/r2.2.0/hadoop-project-dist/hadoop-common/ClusterSetup.html)

##（可选）创建新用户
一般我倾向于把需要启动daemon进程，对外提供服务的程序，简单的说，就是服务器类程序，安装在单独的用户下面。这样可以做到隔离，运维方面，安全性也提高了。

创建一个新的group,

    $ sudo groupadd hadoop

创建一个新的用户，并加入group,

    $ sudo useradd -g hadoop hadoop

给新用户设置密码，

    $ sudo passwd hadoop

##1 伪分布式模式(Pseudo-Distributed Mode)
Hadoop能在单台机器上以伪分布式模式运行，即每个Hadoop模块运行在单独的java进程里。

###1.1 设置SSH无密码登录localhost
先检查一下是能够无密码登录本机，

    ssh localhost

如果提示输入密码，说明不能，按如下步骤设置。

    $ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
    $ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

###1.2 下载已经编译好的二进制包，解压
用浏览器下载或wget,

    $ wget http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.2.0/hadoop-2.2.0.tar.gz
    $ tar -zxf hadoop-2.2.0.tar.gz -C ~/local/opt
    $ cd ~/local/opt/hadoop-2.2.0

###1.3 设置环境变量

    $ vim ~/.bashrc
    export HADOOP_PREFIX=$HOME/local/opt/hadoop-2.2.0
    export HADOOP_COMMON_HOME=$HADOOP_PREFIX
    export HADOOP_HDFS_HOME=$HADOOP_PREFIX
    export HADOOP_MAPRED_HOME=$HADOOP_PREFIX
    export HADOOP_YARN_HOME=$HADOOP_PREFIX
    export HADOOP_CONF_DIR=$HADOOP_PREFIX/etc/hadoop
    export PATH=$PATH:$HADOOP_PREFIX/bin:$HADOOP_PREFIX/sbin

<!-- more -->

###1.4 修改配置文件
配置文件的位置在 `$HADOOP_PREIFIX/etc/hadoop`下面。

###1.4.1 hadoop-env.sh
在这个文件中要告诉hadoop JDK 安装在了哪里

    $ echo $JAVA_HOME
    /usr/lib/jvm/java
    $ vim conf/hadoop-env.sh

取消`JAVA_HOME`那一行的注释，设置正确的JDK位置

    export JAVA_HOME=/usr/lib/jvm/java

####1.4.2 HDFS的配置
为了简单，我们仍采用Hadoop 1.x中的HDFS工作模式（不配置HDFS Federation）。

core-site.xml:

    <configuration>
      <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost</value>
      </property>
    </configuration>

hdfs-site.xml:

    <configuration>
      <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/hadoop/local/var/hadoop/hdfs/datanode</value>
      </property>
      <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/hadoop/local/var/hadoop/hdfs/namenode</value>
      </property>
      <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>file:///home/hadoop/local/var/hadoop/hdfs/namesecondary</value>
      </property>
      <property>
        <name>dfs.replication</name>
        <value>1</value>
      </property>
    </configuration>

Hadoop会自动创建目录。

####1.4.3 YARN的配置
yarn-site.xml，不用修改，保持为空。

####1.4.4 MapReduce的配置
Yarn不仅仅只支持MapReduce这种计算模式，还支持Spar, Tez, MPI等框架，因此，要显式地配置Yarn，使其支持mapreduce。在Hadoop 1.x中，只支持MapReduce，因此需要而且必须配置mapred-site.xml，到了Hadoop 2.x，MapReduce的配置是可选的。

在yarn-site.xml中添加：

    <property>
       <name>yarn.nodemanager.aux-services</name>
       <value>mapreduce_shuffle</value>
    </property>

**解释**：为了能够运行MapReduce程序，需要让各个NodeManager在启动时加载shuffle server，shuffle server实际上是Jetty/Netty Server，Reduce Task通过该server从各个NodeManager上远程拷贝Map Task产生的中间结果。上面增加的这个配置用于指定shuffle server。

将 mapred-site.xml.template 复制一份，重命名为mapred-site.xml。

mapred-site.xml:

    <property>
       <name>mapreduce.framework.name</name>
       <value>yarn</value>
    </property>

**解释**：用mapreduce.framework.name指定采用的框架名称，默认是将作业提交到MRv1的JobTracker端。

###1.5 测试

####1.5.1 启动HDFS

    $ hdfs namenode -format
    $ start-dfs.sh

####1.5.2 启动Yarn

    $ start-yarn.sh

####1.5.3 启动MapReduce
在Hadoop 2.x中，MapReduce Job不需要额外的daemon进程，在MapReduce Application Master启动的时候会自动启动JobTracker和TaskTracker进程，Job结束的时候会自动被关闭。

####1.5.4 运行一个DistributedShell的例子
运行一个Hadoop自带的例子，名称为`DistributedShell`，可以同时在多台机器上运行shell命令。

    $ hadoop jar $HADOOP_PREFIX/share/hadoop/yarn/hadoop-yarn-applications-distributedshell-2.2.0.jar org.apache.hadoop.yarn.applications.distributedshell.Client --jar $HADOOP_PREFIX/share/hadoop/yarn/hadoop-yarn-applications-distributedshell-2.2.0.jar --shell_command date --num_containers 2

运行完成后，看看倒数第三行，有类似与`application_1391783685869_0001`的字符串，这是application ID。查看该application的每个container的输出，执行下面的命令，

    $ grep "" $HADOOP_PREFIX/logs/userlogs/<APPLICATION ID>/**/stdout

####1.5.5 运行wordcount

    $ cd $HADOOP_PREFIX
    $ hdfs dfs -put ./etc/hadoop/ input
    $ hadoop jar $HADOOP_PREFIX/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar wordcount input output
    $ hdfs dfs -lsr output
    $ hdfs dfs -cat output/part-r-00000

结束后，关闭 Hadoop:

    $ stop-dfs.sh
    $ stop-yarn.sh


##2 分布式模式(Fully-Distributed Mode)

###2.1 准备3台机器
如果你已经有了三台机器，这一步可以省略。

如果没有，则可以用VMware Workstation 或 VirtualBox创建3台虚拟机。首先用vmware workstation 新建一台CentOS 6.5，装好操作系统，选择 Basic Server，安装JDK，参考我的另一篇博客，[安装和配置CentOS服务器的详细步骤](http://www.yanjiuyanjiu.com/blog/20120423/)。安装好后然后用**浅拷贝**`Create a linked clone` 克隆出2台，这样有了3台虚拟机。启动3台机器，假设IP分别为`192.168.1.131, 192.168.1.132, 192.168.1.133`, 131做为NameNode,JobTracker和SecondaryNameNode，身兼3个角色，这3个角色应该放到3台不同的机器上，这里为了简化，用一台机器来做3个角色；132和133为 slave。假设三台机器上的用户名是`hadoop`，也可以用其他用户名，但必须三台机器都相同。

####2.1.1 关闭防火墙
临时关闭防火墙

	$ sudo service iptables stop

下次开机后，防火墙还是会启动。

永久关闭防火墙

	$ sudo chkconfig iptables off

由于这几台虚拟机是开发机，不是生产环境，因此不必考虑安全性，可以永久关闭防火墙，还能给开发阶段带来很多便利。

####2.1.2 修改hostname
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

####2.1.3 修改所有机器的`/etc/hosts`文件
在所有机器的`/etc/hosts`文件中，添加所有hostname对应的IP，一般在一台机器上设置好，然后scp到所有机器。例如

	192.168.1.131 master
	192.168.1.132 slave01
	192.168.1.133 slave02

###2.2 配置 master 无密码登陆到所有机器（包括master自己登陆自己）
参考我的另一篇博客，[SSH无密码登录的配置](http://www.yanjiuyanjiu.com/blog/20120102/)

###2.3 把Hadoop压缩包上传到所有机器，并解压
将 hadoop-1.2.1-bin.tar.gz 上传到所有机器，然后解压。**注意，所有机器的hadoop路径必须一致，因为master会登陆到slave上执行命令，master认为slave的hadoop路径与自己一样。**

下面开始配置，配置好了后，把`./etc/hadoop`目录scp到所有其他机器。

###2.4 修改配置文件
在第1节的基础上，增加下列修改。

####2.4.1 指定NameNode
在core-site.xml中，`fs.defaultFS`要改为运行NameNode的那台机器的hostname，不再是localhost。

    <configuration>
      <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master</value>
      </property>
    </configuration>
    
####2.4.2 指定ResourceManager
在yarn-site.xml中增加，

    <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>master</value>
    </property>

####2.4.3 添加Slave，即NodeManager
在 `etc/hadoop/slaves`中添加，

	slave01
	slave02

####2.4.4 设置 hadoop.tmp.dir
在core-site.xml里添加：

    <property>
      <name>hadoop.tmp.dir</name>
      <value>/home/hadoop/local/var/hadoop/tmp/hadoop-${user.name}</value>
    </property>

####2.4.4 修改mapred-site.xml
添加如下内容：

    <property>
        <name>mapreduce.jobtracker.staging.root.dir</name>
        <value>/user</value>
    </property>

这是为以后的多用户支持做准备。

####2.4.4 设置pid文件的存放位置
在hadoop-env.sh中添加

    export HADOOP_MAPRED_PID_DIR=/home/hadoop/local/var/hadoop/pids

在 mapred-env.sh中添加

    export HADOOP_PID_DIR=/home/hadoop/local/var/hadoop/pids

####2.4.5 将dfs.replication设置为slave的个数
我们这里有2台slave，就设置为2。在hdfs-site.xml里添加：

    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>

####2.4.6 将配置文件拷贝到所有slaves

    $ cd $HADOOP_PREFIX/etc/hadoop
    $ scp hadoop-env.sh mapred-env.sh core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml slaves hadoop@slave01:$HADOOP_PREFIX/etc/hadoop
    $ scp hadoop-env.sh mapred-env.sh core-site.xml hdfs-site.xml mapred-site.xml yarn-site.xml slaves hadoop@slave02:$HADOOP_PREFIX/etc/hadoop

###2.5 设置环境变量
在所有机器上添加环境变量，与第1.3节相同。

###2.6 启动 hadoop

####2.6.1 启动HDFS
在NameNode这个机器（在这里是master）上执行下列命令，

    #只需一次，下次启动不再需要格式化，只需 start-dfs.sh
    $ hdfs namenode -format
    $ start-dfs.sh

####2.6.2 启动Yarn
在ResourceManager这台机器（在这里仍然是master）上执行，

    $ start-yarn.sh

####2.6.3 启动MapReduce
在Hadoop 2.x中，MapReduce Job不需要额外的daemon进程，在Job开始的时候，NodeManager会启动一个MapReduce Application Master（相当与一个精简的JobTracker），Job结束的时候自动被关闭。

####2.6.4 检查是否启动成功
用`jps`查看java进程。

在master上，应该有三个进程，NameNode, SecondaryNameNode, ResourceManger；在每台slave上，应该有两个进程，DataNode, NodeManager。

####2.6.5 Web UI
可以用浏览器打开NameNode, ResourceManager和各个NodeManager的web界面，

* NameNode web UI, <http://master:50070/>
* ResourceManager web UI, <http://master:8088/>
* NodeManager web UI, <http://slave01:8042>

还可以启动JobHistory Server，能够通过Web页面查看集群的历史Job，执行如下命令：

    mr-jobhistory-daemon.sh start historyserver

默认使用19888端口，通过访问<http://master:19888/>查看历史信息。

终止JobHistory Server，执行如下命令：

    mr-jobhistory-daemon.sh stop historyserver

###2.8 运行wordcount
将输入数据拷贝到HDFS中:

	$ cd $HADOOP_PREFIX
	$ hdfs dfs -put ./etc/hadoop input

这一步会报错，"No such file or directory", 用`hdfs dfs -ls /`查看，是空的，难怪了。我们需要手动建立"/user/hadoop"目录，

    $ hdfs dfs -mkdir /user/hadoop

再上传文件，就可以了。

运行WordCount:

	$ hadoop jar $HADOOP_PREFIX/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar wordcount input output

查看结果：

    $ hdfs dfs -lsr output
    $ hdfs dfs -cat output/part-r-00000

如果能看到结果，说明这个例子运行成功。

###2.9 停止 hadoop集群
在master上执行：

    $ stop-yarn.sh
    $ stop-hdfs.sh

##4. 排除错误
本文已经尽可能的把步骤详细列出来了，但是我相信大部分人不会一次成功。这时候，查找错误就很重要了。查找错误最重要的手段是查看hadoop的日志，一般在logs目录下。把错误消息复制粘贴到google，搜索一下，慢慢找错误。


##参考资料

1. [core-default](http://hadoop.apache.org/docs/r2.2.0/hadoop-project-dist/hadoop-common/core-default.xml), [hdfs-default](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml), [yarn-default](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-common/yarn-default.xml), [mapred-default](http://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)
1. [Hadoop YARN安装部署初探](http://dongxicheng.org/mapreduce-nextgen/hadoop-yarn-install/)
1. [Hadoop YARN Installation: The definitive guide](http://www.alexjf.net/blog/distributed-systems/hadoop-yarn-installation-definitive-guide)
1. [Setup newest Hadoop 2.x (2.2.0) on Ubuntu](http://codesfusion.blogspot.com/2013/10/setup-hadoop-2x-220-on-ubuntu.html)
1. [Hadoop-2.2.0集群安装配置实践](http://shiyanjun.cn/archives/561.html)
1. [Hadoop YARN配置参数剖析(1)—RM与NM相关参数](http://dongxicheng.org/mapreduce-nextgen/hadoop-yarn-configurations-resourcemanager-nodemanager/)

##相关文章

1. [在CentOS上安装Hadoop集群](http://www.yanjiuyanjiu.com/blog/20140202)
1. [在Ubuntu上安装Hadoop](http://www.yanjiuyanjiu.com/blog/20120103/)

