---
layout: post
title: "Spark开发环境的配置"
date: 2014-01-30 16:36
comments: true
categories: Spark
---

**软件版本**：Spark 0.9

配置Spark开发环境，其实分为三个层次，一种是针对运维人员，把Spark安装部署到集群；一种是针对普通开发者，引入Spark的jar包，调用Spark提供的接口，编写分布式程序，写好后编译成jar，就可以提交到Spark集群去运行了；第三种是针对Spark开发者，为了给Spark贡献代码，需要git clone Spark的代码，然后导入IDE，为Spark开发代码。

##1 部署Spark集群
这种是运维人员在生产环境下，搭建起一个Spark集群。

###（可选）创建新用户 Spark
一般我倾向于把需要启动daemon进程，对外提供服务的程序，即服务器类的程序，安装在单独的用户下面。这样可以做到隔离，运维方面，安全性也提高了。

创建一个新的group,

    $ sudo groupadd spark

创建一个新的用户，并加入group,

    $ sudo useradd -g spark spark

给新用户设置密码，

    $ sudo passwd spark

在每台机器上创建 spark 新用户，并配置好SSH无密码，参考我的另一篇博客，[SSH无密码登录的配置](http://www.yanjiuyanjiu.com/blog/20120102/)


假设有三台机器，hostname分别是 master, worker01, worker02。

###1.1 下载 Spark 预编译好的二进制包
如果你需要用到HDFS，则要针对Hadoop 1.x 和Hadoop 2.x 选择不同的版本。这里我选择 Hadoop 2.x 版。

    spark@master $ wget http://d3kbcqa49mib13.cloudfront.net/spark-0.9.0-incubating-bin-hadoop1.tgz
    spark@master $ tar zxf spark-0.9.0-incubating-bin-hadoop1.tgz -C ~/local/opt

###1.2 将tgz压缩包scp到所有机器，解压到相同的路径

    spark@master $ scp spark-0.9.0-incubating-bin-hadoop1.tgz spark@worker01:~
    spark@master $ ssh worker01
    spark@worker01 $ tar zxf spark-0.9.0-incubating-bin-hadoop1.tgz -C ~/local/opt
    spark@worker01 $ exit
    spark@master $ scp spark-0.9.0-incubating-bin-hadoop1.tgz spark@worker02:~
    spark@master $ ssh worker02
    spark@worker02 $ tar zxf spark-0.9.0-incubating-bin-hadoop1.tgz -C ~/local/opt
    spark@worker02 $ exit

###1.3 修改配置文件
Spark 0.9 以后，配置文件简单多了，只有一个必须要配置，就是 `conf/slaves` 这个文件。在这个文件里添加slave的hostname。

###1.4 拷贝配置文件到所有slave

    spark@master $ spark@master $ scp ./conf/slaves spark@worker01:~/local/opt/spark-0.9.0-incubating-bin-hadoop1/conf
    spark@master $ spark@master $ scp ./conf/slaves spark@worker02:~/local/opt/spark-0.9.0-incubating-bin-hadoop1/conf

###1.5 启动Spark集群

    spark@master $ ./sbin/start-all.sh

<!-- more -->

也可以一台一台启动，先启动 master

    spark@master $ ./sbin/start-master.sh

启动两台 slave，

    spark@worker01 $ ./sbin/start-slave.sh 1 spark://master:7077
    spark@worker02 $ ./sbin/start-slave.sh 2 spark://master:7077

其中，`1`, `2` 是 worker的编号，可以是任意数字，只要不重复即可，`spark://master:7077` 是 master 的地址。以后向集群提交作业的时候，也需要这个地址。

###1.6 测试一下，向集群提交一个作业

    spark@master $ ./bin/run-example org.apache.spark.examples.SparkPi spark://master:7077


##2 配置普通开发环境
TODO

##3 配置Spark开发环境
当你需要修改Spark的代码，或给Spark添加代码，就需要阅读本节了。

### 3.1 git clone 代码

    git clone git@github.com:apache/incubator-spark.git

###3.2 编译
Spark脚本会自动下载对应版本的sbt和scala编译器，因此机器事先不需要安装sbt和scala

按照 github 官方repo首页的文档，输入如下一行命令即可开始编译，

    ./sbt/sbt assembly

###3.3 运行一个例子

    ./run-example org.apache.spark.examples.SparkPi local

说明安装成功了。

###3.4 试用 spark shell
<!-- more -->

./spark-shell

会出现`scala>`提示符号，可见spark脚本自动下载了scala编译器，其实就是一个jar，例如`scala-compiler-2.10.3.jar`。


###3.5 安装scala
开发Spark的时候，由于Intellij Idea 需要调用外部的sbt和scala，因此机器上还是需要安装scala和sbt。

打开 `projects/SparkBuild.scala`，搜索`scalaVersion`，获得spark所使用的scala编译器版本，然后去scala官网<http://www.scala-lang.org/>，下载该版本的scala编译器，并设置`SCALA_HOME`环境变量，将bin目录加入PATH。例如下载scala-2.10.3.tgz，解压到/opt，设置环境变量如下：

    sudo vim /etc/profile
    export SCALA_HOME=/opt/scala-2.10.3
    export PATH=$PATH:$SCALA_HOME/bin

###3.6 安装sbt

打开`projects/build.properties`，可以看到spark所使用的sbt版本号，去
官网<http://www.scala-sbt.org/>下载该版本的sbt，双击安装。并设置`SBT_HOME`环境变量，将bin目录加入PATH。

###3.7 下载并安装idea

Spark核心团队的hashjoin曾在我博客上留言，说他们都使用idea在开发spark，我用过[Scala IDE](www.scala-ide.org)和idea，两者各有优劣，总的来说，idea要好用一些，虽然我是老牌eclipse用户，但我还是转向了idea。

去idea官网下载idea的tar.gz包，解压就行。运行idea，安装scala插件。

###3.8 生成idea项目文件
在源码根目录，使用如下命令

    ./sbt/sbt gen-idea

就生成了idea项目文件。

###3.9 Open Project
使用 idea，点击`File->Open project`，浏览到 `incubator-spark`文件夹，打开项目，就可以修改Spark代码了。

