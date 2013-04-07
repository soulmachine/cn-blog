---
layout: post
title: "Hadoop 集群安装详细步骤"
date: 2012-01-03 18:54
comments: true
categories: "cloud"
---
本文所使用的版本是 hadoop 1.0.0，即 [2011年12月27日发布的1.0正式版](http://www.iteye.com/news/23874)。

详细安装步骤如下，有大步骤，每个步骤里面有小步骤，绝大部分是必选，只有2步是可选的，粗体表示要特别注意的地方。

##1. 用vmware workstation 新建三台虚拟机
首先用vmware workstation 新建一台ubuntu server，装好操作系统，安装各种必须的软件，包括安装好hadoop。安装好后然后用浅拷贝`Create a linked clone` 克隆出两台作为slave，这样有了三台ubuntu机器。启动三台机器，假设IP分别为`192.168.1.131, 192.168.1.132, 192.168.1.133`, 131做为master,132为 slave01, 133为slave02。

##2. 修改机器名
这一步的作用是让命令行提示看起来好看点，由默认的 dev@bogon变为 dev@master，这步可选，optional，可以跳过。

* 192.168.1.131上执行

``` bash
dev@bogon:~$ sudo vi /etc/hostname
```
输入`master`，重启，会发现命令提示符变为了 `dev@master:~$`

* 192.168.1.132上执行

``` bash
dev@bogon:~$ sudo vi /etc/hostname
```
输入`slave01`，重启，会发现命令提示符变为了 `dev@slave01:~$`

* 192.168.1.133上执行

``` bash
dev@bogon:~$ sudo vi /etc/hostname
```
输入`slave02`，重启，会发现命令提示符变为了 `dev@slave02:~$`

<!-- more -->

##3. 修改master的hosts文件，并拷贝到每台slave上

``` bash
dev@master:~$ sudo vi /etc/hosts
```
添加三行内容

> 192.168.1.131 master  
> 192.168.1.133 slave01  
> 192.168.1.134 slave02  

**注意一定要注释掉**

> \# 127.0.1.1      bogon.localdomain       bogon

最后hosts文件内容如下：

> 127.0.0.1       localhost
> \# 127.0.1.1      bogon.localdomain       bogon
> 192.168.1.131 master
> 192.168.1.133 slave01
> 192.168.1.134 slave02
> \# The following lines are desirable for IPv6 capable hosts
> ::1     ip6-localhost ip6-loopback
> fe00::0 ip6-localnet
> ff00::0 ip6-mcastprefix
> ff02::1 ip6-allnodes
> ff02::2 ip6-allrouters

* 将hosts文件拷贝到另外两台台机器上，覆盖原来的hosts文件

``` bash
dev@master:~$ scp /etc/hosts dev@192.168.1.132:~

dev@master:~$ scp /etc/hosts dev@192.168.1.133:~

```
* 在192.168.1.132上执行

``` bash
dev@slave01:~$ sudo mv hosts /etc/hosts
```
* 在192.168.1.133上执行

``` bash
dev@slave02:~$ sudo mv hosts /etc/hosts
```

##4. 配置 master 无密码登陆到所有机器（**包括本机**）

``` bash
dev@master:~$ ssh-keygen -t rsa

dev@master:~$ cat .ssh/id_rsa.pub >> .ssh/authorized_keys

dev@master:~$ scp .ssh/id_rsa.pub dev@192.168.1.132:~/

dev@master:~$ scp .ssh/id_rsa.pub dev@192.168.1.133:~/

dev@slave01:~$ cat id_rsa.pub >> .ssh/authorized_keys

dev@slave02:~$ cat id_rsa.pub >> .ssh/authorized_keys
```
测试一下，

``` bash
dev@master:~$ ssh slav01
```

如果登陆不上，试试先关闭slave01的防火墙，

``` bash
dev@slave01:~$ sudo ufw disable
```

##5. 复制hadoop安装包到所有机器
从hadoop.apache.org下载 hadoop-1.0.0-bin.tar.gz，上传到master中，解压，然后复制到其他机器，解压。

``` bash
dev@master:~$ tar -zxvf hadoop-1.0.0-bin.tar.gz

dev@master:~$ scp hadoop-1.0.0-bin.tar.gz dev@192.168.1.133:~

dev@master:~$ scp hadoop-1.0.0-bin.tar.gz dev@192.168.1.134:~

dev@slave01:~$ tar -zxvf hadoop-1.0.0-bin.tar.gz

dev@slave02:~$ tar -zxvf hadoop-1.0.0-bin.tar.gz
```

##6. 编辑配置文件

``` bash
dev@master:~$ cd hadoop-1.0.0/etc/hadoop

dev@master:~/hadoop-1.0.0/etc/hadoop$  vi hadoop-env.sh
```
仅需要设置JAVA_HOME，

``` bash
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-i386

dev@master:~/hadoop-1.0.0/etc/hadoop$  vi core-site.xml
```

``` xml
<configuration>
<property>
<name>fs.default.name</name>
<value>hdfs://master:9000</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/home/dev/hadoop_tmp/</value>
</property>
</configuration>
```

``` bash
dev@master:~/hadoop-1.0.0/etc/hadoop$  vi mapred-site.xml
```

``` xml
<configuration>
<property>
<name>mapred.job.tracker</name>
<value>master:9001</value>
</property>
</configuration>
```

``` bash
dev@master:~/hadoop-1.0.0/etc/hadoop$  vi hdfs-site.xml
```

``` xml
<configuration>
<property>
<name>dfs.replication</name>
<value>3</value>
</property>
</configuration>
```

``` bash
dev@master:~/hadoop-1.0.0/etc/hadoop$  vi masters

192.168.1.133

dev@master:~/hadoop-1.0.0/etc/hadoop$  vi slaves

192.168.1.133

192.168.1.134
```

##7. 将配置文件拷贝到各台slave

``` bash
dev@master:~/hadoop-1.0.0/etc/hadoop$ scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml masters slaves dev@192.168.1.133:~/hadoop-1.0.0/etc/hadoop

dev@master:~/hadoop-1.0.0/etc/hadoop$ scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml masters slaves dev@192.168.1.134:~/hadoop-1.0.0/etc/hadoop
```
（master和slaves这2个配置文件可以不拷贝到slave机器上，只在master上保存即可。待验证）

##8. 设置环境变量
* 设置master的环境变量

``` bash
dev@master:~$ vi .bashrc

export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-i386
export CLASSPATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_HOME=~/hadoop-1.0.0
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export CLASSPATH=$CLASSPATH:$HADOOP_HOME/share/hadoop/hadoop-core-1.0.0.jar

export HADOOP_HOME=~/hadoop-1.0.0

export PATH=$PATH:$HADOOP_HOME/bin

dev@master:~$ source .bashrc
```
* 将master上的.bashrc拷贝到其他机器，并source刷新。

``` bash
dev@master:~$ scp .bashrc dev@192.168.1.133:~

dev@master:~$ scp .bashrc dev@192.168.1.134:~

dev@slave01:~$ source .bashrc

dev@slave02:~$ source .bashrc
```

##9. 运行 hadoop

``` bash
dev@master:~$ hadoop  namenode -format  （只需一次，下次启动不再需要格式化，只需 start-all.sh）

dev@master:~$ start-all.sh
```

##10. 检查是否运行成功

``` bash
dev@master:~$ jps

2615 NameNode
2767 JobTracker
2874 Jps

dev@slave01:~$ jps

3415 DataNode
3582 TaskTracker
3499 SecondaryNameNode
3619 Jps

dev@slave02:~$ jps

3741 Jps
3618 DataNode
3702 TaskTracker
```

##11. 停止 hadoop集群

``` bash
dev@master:~$ stop-all.sh
```
让 slave 节点也 可以启动 整个hadoop集群


##注意
1. stat-all.sh 启动后，刚刚开始，namenode的日志里有些异常，是正常的，过一两分钟就好了，如果两分钟后，还有异常不断在打印，就有问题了。datanode的日志，从一开始，正常情况下，就没有异常，如果报了异常，说明有异常，要去排除。

2. masters文件，这个文件很容易被误解，它实际上存放的是secondarynamenode，而不是namenode。

	> An HDFS instance is started on a cluster by logging in to the NameNode machine and running$HADOOP_HOME/bin/start-dfs.sh (orstart-all.sh ). This script. starts a local instance of the NameNode process, logs into every machine listed in theconf/slaves file and starts an instance of the DataNode process, and logs into every machine listed in theconf/masters file and starts an instance of the SecondaryNameNode process. Themasters file does not govern which nodes become NameNodes or JobTrackers; those are started on the machine(s) wherebin/start-dfs.sh andbin/start-mapred.sh are executed. A more accurate filename might be “secondaries,” but that’s not currently the case.

	参考以下三篇文章：
	[Multi-host SecondaryNameNode Configuration](http://www.cloudera.com/blog/2009/02/multi-host-secondarynamenode-configuration/)  
	[SecondaryNamenode应用摘记](http://blog.csdn.net/dajuezhao/article/details/5987580)
	[hadoop下运行多个SecondaryNameNode的配置](http://blog.csdn.net/AE86_FC/article/details/5284181)

3. 一定要注释掉 hosts里面的 `#127.0.1.1      bogon.localdomain       bogon`，参考 [Hadoop集群机器命名机制](http://blog.sina.com.cn/s/blog_631ffec50100w700.html)，[hadoop集群环境安装中的hosts 配置问题](http://blog.csdn.net/singno116/article/details/6298995)。

4. 当测试ssh是否能连通时，如果连接不上，先记得要关闭防火墙，`sudo ufw disable`，参考[hadoop集群安装步骤](http://blog.csdn.net/make19830723/article/details/6230074)。

 
##未解决的疑惑
1. 有的很多文章，比如这篇，[Hadoop集群安装](http://blog.csdn.net/cuishi0/article/details/6824884)，在core-site.xml, mapred-site.xml，直接写上ip:port，这样就不用修改hosts文件了。可是我这样配置，老是不成功。为此我还[在stackoverflow上发了帖子](http://stackoverflow.com/questions/8702637/hadoop-conf-fs-default-name-cant-be-setted-ipport-format-directly)。[这篇文章](http://51mst.iteye.com/blog/1152439)又说只能用host:port的形式。因此ip:port的格式是否能成功还待验证。如果有高手用ip:port配置成功过，请在下面留言或给我发email，一起交流:)