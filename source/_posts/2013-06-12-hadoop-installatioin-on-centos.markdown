---
layout: post
title: "在CentOS上安装Hadoop"
date: 2013-06-12 12:39
comments: true
categories: hadoop
---
Ubuntu上安装，请参考我的另一篇博客，[在Ubuntu上安装Hadoop](http://www.yanjiuyanjiu.com/blog/20120103/)。

**环境**：CentOS 6.4, JDK 1.7, Hadoop 1.1.2

##1. 用vmware workstation 创建三台虚拟机
首先用vmware workstation 新建一台CentOS 6.4，装好操作系统，选择 Basic Server，安装JDK，参考我的另一篇博客，[安装和配置CentOS服务器的详细步骤](http://www.yanjiuyanjiu.com/blog/20120423/)。安装好后然后用浅拷贝`Create a linked clone` 克隆出两台作为slave，这样有了三台虚拟机。启动三台机器，假设IP分别为`192.168.1.131, 192.168.1.132, 192.168.1.133`, 131做为master 和 SecondaryNameNode, 身兼两职， 132和133为 slaves。

##2 关闭防火墙
临时关闭防火墙

	$ sudo service iptables stop

下次开机后，防火墙还是会启动。

永久关闭防火墙

	$ sudo chkconfig iptables off

由于这几台虚拟机是开发机，不是生产环境，因此不必考虑安全性，可以永久关闭防火墙，还能给开发阶段带来很多便利。

##3. 修改hostname
这一步看起来貌似不必要，其实是必须的，否则最后运行wordcount等例子时，会出现“Too many fetch-failures”。因为HDFS用hostname而不是IP，来相互之间进行通信（见后面的注意1）。

在CentOS上修改hostname，包含两个步骤(假设将hostname1改为hostname2，参考[这里](http://www.ichiayi.com/wiki/tech/linux_hostname)，但不需要第一步)：

1. 将 `/etc/sysconfig/network` 內的 HOSTNAME 改成 hostname2
2. 用`hostname`命令，临时修改机器名， `sudo hostname hostname2`

用`exit`命令退出shell，再次登录，命令提示字符串就会变成`[dev@hostname2 ~]$`。

用上述方法，将131改名为master，132改名为slave01，133改名为slave02。

在三台机器的/etc/hosts文件中，添加以下三行内容

	192.168.1.131 master
	192.168.1.132 slave01
	192.168.1.133 slave02

##4. 本地模式和伪分布式模式

为了能顺利安装成功，我们先练习在单台机器上安装Hadoop。在单台机器上，可以配置成本地模式(local mode)和伪分布式模式(Pseudo-Distributed Mode)，参考官方文档[Single Node Setup](http://hadoop.apache.org/docs/r1.1.2/single_node_setup.html)。

将 hadoop-1.1.2-bin.tar.gz 上传到三台机器的 home目录下，然后解压。**注意，三台机器hadoop所在目录必须一致，因为master会登陆到slave上执行命令，master认为slave的hadoop路径与自己一样。**

###4.1 编辑 conf/hadoop-env.sh，设置 JAVA_HOME

    cd hadoop-1.1.2
    vim conf/hadoop-env.sh

注释掉第8行的JAVA_HOME，设置正确的JDK位置

	export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.19.x86_64

###4.2 测试本地模式是否正常
默认情况下，Hadoop就被配置为本地模式，现在就可以开始测试一下。

	$ mkdir input 
	$ cp conf/*.xml input 
	$ bin/hadoop jar hadoop-examples-*.jar grep input output 'dfs[a-z.]+' 
	$ cat output/*

可以看到正常的结果，说明本地模式运行成功了，下面开始配置伪分布式模式。

<!-- more -->

###4.3 配置SSH无密码登陆本机

	$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
	$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

修改sshd的配置文件(需要root权限)

    $ sudo vim /etc/ssh/sshd_config

找到以下三行，并去掉注释符”#“

    RSAAuthentication yes
    PubkeyAuthentication yes
    AuthorizedKeysFile      .ssh/authorized_keys

修改了配置文件需要重启sshd服务

	sudo service sshd restart

修改 `authorized_keys` 文件的权限，否则ssh登陆的时候还是需要密码。权限的设置非常重要,因为不安全的设置,会让你不能使用RSA功能，参考 <http://www.cnblogs.com/xia520pi/archive/2012/05/16/2504132.html>

	$ chmod 600 ~/.ssh/authorized_keys

###4.4 修改配置文件
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

###4.5 启动Hadoop，测试伪分布式模式

格式化namenode

	$ bin/hadoop namenode -format

启动 Hadoop 后台进程

	$ bin/start-all.sh

现在可以用浏览器打开NameNode和JobTracker的web界面了。  
NameNode - <http://localhost:50070/>  
JobTracker - <http://localhost:50030/>


将输入数据拷贝到分布式文件系统中:

	$ bin/hadoop fs -put conf input

如果这时出现 `SafeModeException` 异常，不用担心，等待几分钟即可。因为hadoop刚刚启动时，会进入安全模式进行自检。


运行 Hadoop 自带的例子:

	$ bin/hadoop jar hadoop-examples-*.jar grep input output 'dfs[a-z.]+'

查看输出文件:

	$ bin/hadoop fs -cat output/*

当你做完了后，关闭 Hadoop:

	$ bin/stop-all.sh


##5. 分布式模式
如果有多台机器，就可以把Hadoop 配置成分布式模式(或称为集群模式)。参考官方文档[Cluster Setup](http://hadoop.apache.org/docs/r1.1.2/cluster_setup.html).

##5.1 配置 master 无密码登陆到所有机器（**包括master自己登陆自己**）
首先在两台slave上修改sshd的配置文件，然后重启sshd服务。

	$ sudo vim /etc/ssh/sshd_config
 找到以下三行，并去掉注释符”#“

    RSAAuthentication yes
    PubkeyAuthentication yes
    AuthorizedKeysFile      .ssh/authorized_keys

修改了配置文件需要重启sshd服务

	$ sudo service sshd restart

在两台slaves机器上新建 ~/.ssh 目录

	$ mkdir ~/.ssh

在master机器上执行： 
 
	#生成公钥
	$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa

把公钥拷贝到所有机器上（包括自己）
	
	# 拷贝到自己
	$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
	# 拷贝到两台slaves机器上
	$ scp ~/.ssh/id_rsa.pub dev@192.168.1.132:~/.ssh/authorized_keys
	$ scp ~/.ssh/id_rsa.pub dev@192.168.1.133:~/.ssh/authorized_keys

修改 .ssh目录和authorized_keys 文件的权限

	#在三台机器上执行以下命令
	$ chmod 600 ~/.ssh/authorized_keys
	# 在两台 slaves 上执行以下命令
	$ chmod 700 ~/.ssh

测试，看看 master 是否可以无密码登陆两台slave

	#在 master执行
	$ ssh 192.168.1.132
	$ exit
	$ ssh 192.168.1.133
	$ exit

如果不需要密码，则说明配置成功了。


如果登陆不上，试试先关闭两台slaves的防火墙

	$ sudo service iptables stop

###5.2 修改6个配置文件
在 master 上修改配置文件。

编辑 conf/hadoop-env.sh，设置 JAVA_HOME。

在master上编辑 conf/hadoop-env.sh，注释掉第8行的JAVA_HOME，设置正确的JDK位置，确保集群中所有机器的JDK都安装在这个位置，这样待会儿后面可以用scp命令把master的6个配置文件(hadoop-env.sh, core-site.xml, hdfs-site.xml, mapred-site.xml, masters, slaves)拷贝到所有slaves机器上，这样其他机器就不用重复进行配置了。

conf/masters:

	master

conf/slaves:

	slave01
	slave02

这两个配置文件的作用是，指定131作为master, 132和133为 slaves。

conf/core-site.xml:

	<configuration>
	    <property>
	        <name>fs.default.name</name>
	        <value>hdfs://master:9000</value>
	    </property>
	</configuration>

conf/hdfs-site.xml:

	<configuration>
	     <property>
	         <name>dfs.name.dir</name>
	         <value>/home/dev/hdfs/name</value>
	     </property>
	     <property>
	         <name>dfs.data.dir</name>
	         <value>/home/dev/hdfs/data</value>
	     </property>
	</configuration>

要在master创建 /home/dev/hdfs/name 目录，在 slaves上创建 /home/dev/hdfs/data 目录，并`chmod g-w /home/dev/hdfs/data`（权限不对的话datanode无法启动）。

conf/mapred-site.xml:

	<configuration>
	    <property>
	        <name>mapred.job.tracker</name>
	        <value>master:9001</value>
	    </property>
	</configuration>

###5.3 将配置文件拷贝到所有slaves

	$ cd ~/hadoop-1.1.2/conf/
	$ scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml masters slaves dev@192.168.1.132:~/hadoop-1.1.2/conf/
	$ scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml masters slaves dev@192.168.1.133:~/hadoop-1.1.2/conf/

###5.4 （可选）设置环境变量HADOOP\_HOME，并将`$HADOOP_HOME/bin`加入PATH
这一步是为了将bin目录加入PATH，这样可以在任何位置执行hadoop的各种命令。这步是可选的。

Hadoop不推荐使用`$HADOOP_HOME`，你可以试一下，当设置了`$HADOOP_HOME`后，执行`bin/start-all.sh`，第一行会打印出一行警告信息，`Warning: $HADOOP_HOME is deprecated.`

给所有机器设置环境变量HADOOP\_HOME，并将`$HADOOP_HOME/bin`加入PATH。

	$ vim ~/.bash_profile
	# add the following lines at the end
	export HADOOP_HOME=$HOME/hadoop-1.1.2
	export PATH=$PATH:$HOME/bin:$HADOOP_HOME/bin::$HADOOP_HOME/sbin
	export CLASSPATH=$CLASSPATH:$HADOOP_HOME/hadoop-core-1.1.2.jar
	# make the bash profile take effect immediately
	$ source ~/.bash_profile


##5.5 （可选）设置别名，名称为hadoop，指向bin/hadoop
像bin目录下的start-all.sh, stop-all.sh其实不常用，对于一个hadoop使用者而言，最常用的命令是hadoop，例如`hadoop fs -ls`。前面5.4节将bin目录加入PATH，相当于一股脑儿将所有的命令加入了PATH，其实大可不必，我们只需要设置一个别名，名称为hadoop，指向bin/hadoop就可以了。

在所有机器上设置hadoop 别名，步骤如下：

	$ vim ~/.bash_profile
	# add the following line at the end
	alias hadoop='~/hadoop-1.1.2/bin/hadoop'
	#make the bash profile take effect immediately
	$ source ~/.bash_profile


###5.6 运行 hadoop
在master上执行以下命令，启动hadoop

	$ cd ~/hadoop-1.1.2/
	#只需一次，下次启动不再需要格式化，只需 start-all.sh
	$ bin/hadoop  namenode -format
	$ bin/start-all.sh

###5.7 检查是否启动成功

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

###5.8 运行wordcount例子，进一步测试是否安装成功
将输入数据拷贝到分布式文件系统中:

	$ cd ~/hadoop-1.1.2/
	$ bin/hadoop fs -put conf input

运行 Hadoop 自带的例子:

	$ bin/hadoop jar hadoop-examples-*.jar wordcount input output

查看输出文件:

	$ bin/hadoop s -ls output
	$ bin/hadoop fs -cat output/part-r-00000

如果能看到结果，说明这个例子运行成功。

###5.9 停止 hadoop集群
在master上执行：

	$ bin/stop-all.sh

##6. 排除错误
本文已经尽可能的把步骤详细列出来了，但是我相信大部分人不会一次成功。这时候，查找错误就很重要了。查找错误最重要的手段是查看hadoop的日志，一般在logs目录下。把错误消息复制粘贴到google，搜索一下，慢慢找错误。

##注意
1. 所有配置文件只能用hostname，不能用IP。两年前我不懂，还为此[在stackoverflow上发了帖子](http://stackoverflow.com/questions/8702637/hadoop-conf-fs-default-name-cant-be-setted-ipport-format-directly)。hadoop会反向解析hostname，即使是用了IP，也会使用hostname 来启动TaskTracker。参考[hdfs LAN ip address hostname resolution](http://stackoverflow.com/questions/15230946/hdfs-lan-ip-address-hostname-resolution)，[hadoop入门经验总结- 杨贵堂的博客](http://www.makenotes.net/?p=337004)，[hadoop集群配置](http://51mst.iteye.com/blog/1152439)。
2. 如果在第 3.7步有问题，可以`stop-all.sh`, 然后`hadoop  namenode -format`，反复多试几次，一般可以成功。如果不习惯，多看看 logs目录下的日志文件，把错误消息复制粘贴到google，搜索答案。
3. 在第2.5步骤，如果出现 `SafeModeException` 异常，不用担心，等待几分钟即可。因为hadoop刚刚启动时，会进入安全模式进行自检，这需要花点时间。
