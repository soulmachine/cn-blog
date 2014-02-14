---
layout: post
title: "CentOS上编译 Hadoop 2.2.0"
date: 2014-02-14 11:56
comments: true
categories: Hadoop
---

下载了Hadoop预编译好的二进制包，hadoop-2.2.0.tar.gz，启动起来后，总是出现这种警告：

    WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

原因是apache官网提供的二进制包，里面的native库，是32位的，坑跌啊，现在服务器谁还有32位的啊。

    $ file $HADOOP_PREFIX/lib/native/libhadoop.so.1.0.0
    libhadoop.so.1.0.0: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, BuildID[sha1]=0x9eb1d49b05f67d38454e42b216e053a27ae8bac9, not stripped

我们需要下载Hadoop 2.2.0源码，在 64 位Linux下重新编译，然后把32位的native库用64位的native库替换。

<!-- more -->

##1. 下载Hadoop 2.2.0 源码包，并解压

    $ wget http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.2.0/hadoop-2.2.0-src.tar.gz
    $ tar zxf hadoop-2.2.0-src.tar.gz

##2. 安装下面的软件

     $ sudo yum install lzo-devel  zlib-devel  gcc autoconf automake libtool   ncurses-devel openssl-deve

##3. 安装Maven
不要使用最新的Maven 3.1.1。Hadoop 2.2.0的源码与Maven3.x存在兼容性问题，所以会出现

    java.lang.NoClassDefFoundError: org/sonatype/aether/graph/DependencyFilter

之类的错误。

安装 Maven 3.0.5

    $ wget http://mirror.esocc.com/apache/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
    $ sudo tar zxf apache-maven-3.0.5-bin.tar.gz -C /opt
    $ sudo vim /etc/profile
    export MAVEN_HOME=/opt/apache-maven-3.0.5
    export PATH=$PATH:$MAVEN_HOME/bin

注销并重新登录，让环境变量生效。

##4. 安装Ant

    $ wget http://apache.dataguru.cn//ant/binaries/apache-ant-1.9.3-bin.tar.gz
    $ sudo tar zxf apache-ant-1.9.3-bin.tar.gz -C /opt
    $ sudo vim /etc/profile
    export ANT_HOME=/opt/apache-ant-1.9.3
    export PATH=$PATH:$ANT_HOME/bin

##5. 安装Findbugs

    $ wget http://prdownloads.sourceforge.net/findbugs/findbugs-2.0.3.tar.gz?download
    $ sudo tar zxf findbugs-2.0.3.tar.gz -C /opt
    $ sudo vim /etc/profile
    export FINDBUGS_HOME=/opt/findbugs-2.0.3
    export PATH=$PATH:$FINDBUGS_HOME/bin

##6. 安装protobuf
编译Hadoop 2.2.0，需要protobuf的编译器protoc。一定需要protobuf 2.5.0以上，yum里的是2.3，太老了。因此下载源码，编译安装。

    $ wget https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz
    $ tar zxf protobuf-2.5.0.tar.gz
    $ cd protobuf-2.5.0
    $ ./configure
    $ make
    $ sudo make install

##7. 给Hadoop源码打一个patch
最新的Hadoop 2.2.0 的Source Code 压缩包解压出来的code有个bug 需要patch后才能编译。否则编译hadoop-auth 会提示下面错误：

    [ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:2.5.1:testCompile (default-testCompile) on project hadoop-auth: Compilation failure: Compilation failure:
    [ERROR] /home/chuan/trunk/hadoop-common-project/hadoop-auth/src/test/java/org/apache/hadoop/security/authentication/client/AuthenticatorTestCase.java:[84,13] cannot access org.mortbay.component.AbstractLifeCycle
    [ERROR] class file for org.mortbay.component.AbstractLifeCycle not found

Patch: <https://issues.apache.org/jira/browse/HADOOP-10110>

##8. 编译 Hadoop

    cd hadoop-2.2.0-src
    mvn package -DskipTests -Pdist,native -Dtar

##9. 替换掉32位的native库
用 `hadoop-2.2.0-src/hadoop-dist/target/hadoop-2.2.0/lib/native` 替换掉 `hadoop-2.2.0/lib/native`。

    rm -rf ~/local/opt/hadoop-2.2.0/lib/native
    cp ./hadoop-dist/target/hadoop-2.2.0/lib/native ~/local/opt/hadoop-2.2.0/lib/

然后重启Hadoop集群，会看到控制台下不再有警告信息了。
    
##10 解决Ubuntu下启动失败的问题
在Ubuntu上，那就不是一点WARN了，而是启动不起来，会出错，原因在于，在`./sbin/start-dfs.sh`第55行，

    NAMENODES=$($HADOOP_PREFIX/bin/hdfs getconf -namenodes)

在shell里单独运行这样命令，

    ./bin/hdfs getconf -namenodes

    OpenJDK 64-Bit Server VM warning: You have loaded library /home/soulmachine/local/opt/hadoop-2.2.0/lib/native/libhadoop.so which might have disabled stack guard. The VM will try to fix the stack guard now.
    It's highly recommended that you fix the library with 'execstack -c <libfile>', or link it with '-z noexecstack'.
    14/02/14 13:14:50 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
    localhost
    
最后一行的localhost，才是有效的namenode，但是由于前面有一大堆warning，脚本把这一大堆字符串，按空格隔开，每个单词都看作是namenode，接下来就错的稀里哗啦。

根本原因，还是因为32位native库。

把自带的32位native目录删除，用编译好的64位native目录拷贝过去，再运行

    ./bin/hdfs getconf -namenodes
    localhost

这下就对了！

##参考资料

1. [YARN加载本地库抛出Unable to load native-hadoop library解决办法](http://blog.csdn.net/lalaguozhe/article/details/10580727)
1. [CentOS编译Hadoop 2.2.0 Pass 总结](http://blog.csdn.net/zwj0403/article/details/16855555)

