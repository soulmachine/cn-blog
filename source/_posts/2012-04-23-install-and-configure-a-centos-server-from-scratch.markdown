---
layout: post
title: "安装和配置CentOS服务器的详细步骤"
date: 2012-04-23 20:43
comments: true
categories: tools
---

这是我安装CentOS服务器的过程，记录下来，与大家一起分享。

##安装操作系统
CentOS 6.2 ，CentOS-6.2-i386-bin-DVD1.iso（32位） ，CentOS-6.2-x86_64-bin-DVD1.iso（64位）

安装 CentOS时，选择 "Basic Server"  
root密码：root123  
CentOS 自带了ssh  
 
安装完操作系统后，添加一个用户 dev

``` bash
[root@localhost ~]$ useradd dev

```
然后密码设为 dev123

``` bash
[root@localhost ~]$ passwd dev
```

给予 sudo 权限

``` bash
[root@localhost ~]$ chmod u+w /etc/sudoers
[root@localhost ~]$ vim /etc/sudoers
# 在root ALL=(ALL) ALL 下 添加dev ALL=(ALL) ALL
[root@localhost ~]$ chmod u-w /etc/sudoers 
```

##设置上网
安装完操作系统后，还不能上网，配置DHCP方式上网：

``` bash
vim /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
HWADDR="00:0C:29:BD:E1:19"
NM_CONTROLLED="yes"
ONBOOT="yes"
BOOTPROTO=dhcp
USECTL=no
TYPE=Ethernet
PEERDNS=yes
#保存退出
sudo service network restart
```

<!-- more -->

或者，配置静态IP

``` bash
DEVICE="eth0"
HWADDR="00:0C:29:10:F4:4C"
ONBOOT="yes"
BOOTPROTO=static
TYPE=Ethernet
IPADDR=192.168.0.162
NETMASK=255.255.255.0
BROADCAST=192.168.0.255
NETWORK=192.168.0.0
#保存退出  
#修改/etc/sysconfig/network
sudo vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=localhost.localdomain
GATEWAY=192.168.0.1
#保存退出，重启网络
sudo service network restart
```
如果失败，比如IP已被占用，换一个IP试试

修改DNS，即时生效

``` bash
sudo vim /etc/resolv.conf
nameserver 192.168.0.1
# google提供的域名服务器
nameserver 8.8.8.8
search localhost
```

##安装常用软件
有两种方式，方法一，去官网下载已经编译好的二进制文件，或源代码，编译安装
方法二，用yum 命令安装，安装官方yum源里已经编译好的程序包。  
第一种方式要敲很多命令，比yum麻烦，但是可以预先下载好文件，省略了下载的时间，整体速度比yum安装方式快很多，而且可以安装最新版。推荐第一种方式

第二种方式操作简单，敲打的命令少，但是往往yum源的更新速度跟不上各个软件的官网速度，用Yum安装的版本经常比较旧。

yum的命令形式一般是如下：`yum [options] [command] [package ...]`，其中的[options]是可选的，选项包括-h（帮助），-y（当安装过程提示选择全部为"yes"），-q（不显示安装的过程）等等。[command]为所要进行的操作，[package ...]是操作的对象。

``` bash
#yum search package-name # 在线搜索包 
#yum list installed # 列出所有已经安装的包
#
#sudo yum install package-name # 安装程序包 
#sudo yum groupinsall group-name 安装程序组
#
#sudo yum remove package-name 删除程序包
#sudo yum groupremove group-name 删除程序组
#
#yum update #全部更新
#yum update package-name #更新程序包
#sudo yum groupupdate groupn-name 升级程序组
#sudo yum upgrade # 更新源列表
#yum upgrade package-name #升级程序包
#sudo yum clean all # 清除缓存
#更新
sudo yum update
#清理缓存
sudo yum clean all && yum clean metadata && yum clean dbcache
```

##安装编译工具

###方法一
去 http://gcc.gnu.org/ 下载源码

``` bash
# TODO
```

###方法二

``` bash
sudo yum groupinstall "Development Tools"
```
该命令类似于 Ubuntu 下的`apt-get install build-essential`，会自动安装一下软件包：autoconf automake bison byacc cscope ctags diffstat doxygen flex gcc gcc-c++ gcc-gfortran git indent intltool libtool patchutils rcs redhat-rpm-config rpm-build subversion swig systemtap，同时安装了以下依赖包：apr, apr-util, 等等。


##安装JDK

``` bash
#删除旧的JDK
yum list installed | grep jdk
#复制显示出来的JDK，卸载
sudo yum remove java-1.6.0-openjdk.x86_64
#安装新的jdk
```

###方法一

``` bash
#从官网下载最新版的，当前是jdk6u32
#开始安装
chmod u+x chmod u+x jdk-6u32-linux-x64-rpm.bin
sudo ./jdk-6u32-linux-x64-rpm.bin
#设置环境变量，.bash_profile是当前用户，/etc/profile是所有用户的
sudo vim /etc/profile
#在末尾添加
export JAVA_HOME=/usr/java/default
export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
# 保存退出，输入以下命令使之立即生效：
source /etc/profile
# 测试
java -version
```

###方法二

``` bash
yum search jdk
# java-1.6.0-openjdk只包含了JRE，如果在这台机器上开发java程序，则需要安装JDK，
# 要选择 java-1.6.0-openjdk-devel，在服务器上我们只需要运行java程序，因此选择前者
sudo yum install java-1.6.0-openjdk-devel
# 使用 alternatives 工具设置默认JDK，参考：Installing a Java Development Kit on Red Hat Enterprise Linux
/usr/sbin/alternatives --config java
/usr/sbin/alternatives --config javac
# 设置环境变量
# 查询JDK路径
whereis java
ll /usr/bin/java
ll /etc/alternatives/java #这是可以看到JDK路径了
sudo vim /etc/profile
# 在末尾添加
export JAVA_HOME=/usr/lib/jvm/jre-1.6.0-openjdk.x86_64
export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
#保存退出，输入以下命令使之立即生效：
# source /etc/profile
# 测试
java -version
```

##安装 apache

###方法一
源码在官网 http://httpd.apache.org/ 下载。  
先下载apt, apr-util, pcre三个库，httpd 在编译时需要用到这三个库  
apr, apr-util官网 http://apr.apache.org , pcre官网为 [http://pcre.org ](http://pcre.org )

``` bash
# 编译，安装 apr
tar -jxf apr-1.4.6.tar.bz2
cd apr-1.4.6
./configure
make
sudo make install    # 默认会安装到 /usr/local/apr/
cd ~
#编译，安装 apr-util
tar -jxf apr-util-1.4.1.tar.bz2
cd apr-util-1.4.1
./configure --with-apr=/usr/local/apr/
make
sudo make install    # 默认会安装到 /usr/local/apr/
cd ~
#编译，安装 pcre
tar -jxf pcre-8.30.tar.bz2
cd  pcre-8.30
./configure --with-apr=/usr/local/apr/
make
# By default, `make install' installs the package's commands under
#`/usr/local/bin', include files under `/usr/local/include', etc. 
sudo make install
cd ~
#编译，安装 apache
tar -jxf httpd-2.2.22.tar.bz2
cd httpd-2.2.22
./configure
make
sudo make install    # 默认会安装到/usr/local/apache2/
cd ~
#添加防火墙规则，让防火墙允许 apache的端口 80通过
sudo vim /etc/sysconfig/iptables
#添加如下一行（实际上是拷贝了原来的一行，仅仅改变了端口号），位置必须#要放在 含有 "REJECT --reject-with" 的行的前面
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
sudo service iptables restart
#测试
sudo /usr/local/apache2/bin/apachectl start
#在浏览器输入 http://ip地址 ，如果看到“It works”，说明安装成功
/usr/local/apache2/bin/apachectl stop


#设置为开机启动
#将httpd注册为服务，通过chkconfig实现开机启动
#以apachectl 为模板
sudo cp /usr/local/apache2/bin/apachectl /etc/init.d/httpd
sudo vim /etc/init.d/httpd
# 在第一行 #!/bin/sh，添加如下一行，使其支持chkconfig命令
# chkconfig: 2345 85 15
# 保存，退出VIM编辑器
sudo chmod u+x /etc/init.d/httpd
sudo chkconfig --add httpd
sudo chkconfig httpd on
#检查一下，是否添加成功
chkconfig --list httpd 
```

###方法二

``` bash
sudo yum install httpd
#可选？sudo yum install httpd-devel
#测试
#启动 apache http server
sudo service httpd start
#添加规则，让防火墙允许 apache的端口 80
sudo vim /etc/sysconfig/iptables
#添加如下一行，位置必须要放在 含有 "REJECT --reject-with" 的行的前面
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
sudo service iptables restart
#可以在浏览器输入 http://ip地址 测试了
#设置为开机启动
sudo chkconfig httpd on
```

##安装 mysql

###方法一

``` bash
#去官网下载 Oracle & Red Hat 6的安装包，64位为MySQL-5.5.23-1.el6.x86_64.tar，
#32位为 MySQL-5.5.23-1.el6.i686.tar
tar -xf MySQL-5.5.23-1.el6.x86_64.tar
#加 --force 是因为可能会与mysqllib库冲突
sudo rpm -ivh --force  MySQL-server-5.5.23-1.el6.x86_64.rpm
sudo rpm -ivh MySQL-client-5.5.23-1.el6.x86_64.rpm
# 启动 mysql 服务器
sudo service mysql start
#设置为开机启动
sudo chkconfig mysql on
```

###方法二

``` bash
sudo yum install mysql-server
sudo chgrp -R mysql /var/lib/mysql
sudo chmod -R 770 /var/lib/mysql
# 启动 mysql 服务器
sudo service mysqld start
#设置为开机启动
sudo chkconfig mysqld on
```

###公共的操作

``` bash
# root 初始密码为空，修改root密码
mysql -u root
mysql> use mysql;
mysql> update user set password=password('root123') where user='root' AND host='localhost';
mysql> flush privileges;
# 打开MySQL中root账户的远程登录，参考：如何打开MySQL中root账户的远程登录mysql> GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "root";
mysql> update user set password=password('root123') where user='root' AND host='%';
mysql> flush privileges;
mysql> quit;
#添加防火墙规则，让防火墙允许 mysql 的端口 3306通过
sudo vim /etc/sysconfig/iptables
#添加如下一行（实际上是拷贝了原来的一行，仅仅改变了端口号），位置必须#要放在 含有 "REJECT --reject-with" 的行的前面
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
sudo service iptables restart
```

##安装 php5

###方法一
TODO

###方法二

``` bash
sudo yum install php php-pear
#重启 apache，以确保apache 加载PHP模块
sudo service httpd restart
# 在 /var/www/html/下新建一个index.php文件，用于测试
cd /var/www/html
sudo vim index.php
# 添加如下一行
<?php phpinfo(); ?>
# 在浏览器输入 http://xxx.xxx.xxx.xxx/index.php ，测试PHP是否成功安装

# 如果需要在PHP中支持mysql，则需要安装 php-mysql 模块
sudo yum install php-mysql
# 如果需要在PHP中支持memcached，则需要安装 php-pecl-memcache 模块
sudo yum install php-pecl-memcache
#安装一些常用的PHP扩展模块
sudo yum install php-devel php-gd php-mbstring php-xml

#可以安装一个wordpress进行测试，注意要修改文件夹权限
sudo chown -R apache.apache /var/www/html
```

## 安装 memcached

###方法一

``` bash
# memcached依赖libevent，首先要安装 libevent
# 去 http://libevent.org/ 下载libevent源码，然后编译，安装
tar -zxf libevent-2.0.18-stable.tar.gz
cd libevent-2.0.18-stable
./configure
make
sudo make install
# 对于64位操作系统(32位不需要)，还需要配置：
sudo ln -s /usr/local/lib/libevent-2.0.so.5 /usr/lib64//libevent-2.0.so.5
# 去 http://www.memcached.org/ 下载 memcached，然后编译，安装
tar -zxf memcached-1.4.13.tar.gz
cd memcached-1.4.13
./configure
make
sudo make install
# 启动, -p，端口,-m，内存, -u
memcached -p 11211 -m 512m -u root -d
# 开机启动
# centos设置开机启动有两种方式，一是把启动程序的命令添加到/etc/rc.d/rc.local文件中，二是把写好的启动脚本添加到目录/etc/rc.d/init.d/，然后使用命令chkconfig设置开机启动。第二种方式可以用 service xxx start|stop来启动或停止，所以推荐第二种。
#将 memcached启动命令注册为一个服务
cd /etc/init.d/
sudo vim memcached
#代码如下，参考：Linux中将memcached注册成服务并可以随机器启动时启动服务
#chkconfig: 345 60 60
#!/bin/bash

start()
{
        echo -n $"Starting memcached: "
        /usr/local/bin/memcached -p 11211 -m 512m -u root -d
        echo "[OK]"
}
stop()
{
        echo -n $"Shutting down memcached: "
        memcached_pid_list=`pidof memcached` 
        kill -9 $memcached_pid_list
        echo "[OK]"
}
case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart)
        stop
        sleep 3
        start
        ;;
    *)
             echo $"Usage: $0 {start|stop|restart}"
        exit 1
esac
exit 0
#保存退出
sudo chmod u+x memcached
sudo chkconfig --add memcached
sudo chkconfig  memcached on
```

###方法二
TODO

##安装 tomcat6

###方法一

``` bash
# 去 http://tomcat.apache.org 下载 apache-tomcat-6.0.35.tar.gz
tar -zxf apache-tomcat-6.0.35.tar.gz
sudo mv apache-tomcat-6.0.35 /usr/local/
cd /usr/local/apache-tomcat-6.0.35/bin
#【可选】添加环境变量
sudo vim /etc/profile
export CATALINA_HOME=/usr/local/apache-tomcat-6.0.35
#启动 tomcat 
sudo ./startup.sh
# 在浏览器输入 http://xxx.xxx.xxx.xxx:8080/ ，如果能看见tomcat页面，则表示安装成功了
#设置开机启动
#将 tomcat启动命令注册为一个服务
cd /etc/init.d/
sudo vim tomcatd
#代码如下
#chkconfig: 345 60 60
#!/bin/bash
CATALINA_HOME=/usr/local/apache-tomcat-6.0.35

start()
{
        echo -n $"Starting Tomcat: "
        $CATALINA_HOME/bin/startup.sh
        echo "[OK]"
}
stop()
{
        echo -n $"Shutting down Tomcat: "
        $CATALINA_HOME/bin/shutdown.sh
        echo "[OK]"
}
case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart)
        stop
        sleep 3
        start
        ;;
    *)
             echo $"Usage: $0 {start|stop|restart}"
        exit 1
esac
exit 0
#保存退出
sudo chmod u+x tomcatd
sudo chkconfig --add tomcatd
sudo chkconfig tomcatd on

#添加防火墙规则，让防火墙允许 tomcat 的端口 8080 通过
sudo vim /etc/sysconfig/iptables
#添加如下一行（实际上是拷贝了原来的一行，仅仅改变了端口号），位置必须#要放在 含有 "REJECT --reject-with" 的行的前面
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
sudo service iptables restart
```

###方法二

``` bash
#搜索一下 tomcat包的名字
yum search tomcat
sudo yum search tomcat6.noarch
```

##安装Python

###方法一：去[官网](http://www.python.org)下载源码，编译，安装

``` bash
#开始解压，编译，安装
tar -jxf Python-3.2.3.tar.bz2
cd Python-3.2.3
# 查看一下说明, vim README
./configure
make
#为了加快安装速度，这步可以省略
make test
#卸载旧的python，注意，不能用 yum remove python，这会卸载几百个包，最终损坏系统
sudo rpm -evf --nodeps python
sudo make install
#默认安装在 /usr/local/bin/python3
```

###方法二

``` bash
sudo yum install python
```

##安装ruby

###方法一

``` bash
# http://www.ruby-lang.org/en/downloads/ ，选择 "Stable Snapshot"
tar -zxf ruby-1.9-stable.tar.gz
cd  cd ruby-1.9.3-p194/
./configure
make
sudo make install
```

###方法二

``` bash
sudo yum install ruby
```

##安装go

``` bash
#去官网 http://code.google.com/p/go/downloads 下载，go1.0.1.linux-i386.tar.gz (32位)，go1.0.1.linux-amd64.tar.gz（64位）
tar -zxf go1.0.1.linux-amd64.tar.gz
sudo mv go/ /usr/local/
#设置环境变量
sudo vim /etc/profile
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
source /etc/profile
#测试一下
go version
```

##安装lua

``` bash
# 去官网下载源码，编译，安装。由于官网仅提供源码，故推荐源码编译安装方式。
# 去官网 http://www.lua.org/ 下载源码，lua-5.2.0.tar.gz
tar -zxf lua-5.2.0.tar.gz 
cd lua-5.2.0
# lua 依赖 readline.h 头文件
sudo yum install  readline-devel
make linux
sudo make install
#安装 google protobuf
#去官网 http://code.google.com/p/protobuf/下载
tar -jxf protobuf-2.4.1.tar.bz2
cd protobuf-2.4.1
./configure
make
sudo make install
#测试
protoc
```

##清理安装包

``` bash
cd ~
rm * -rf
```

##压缩打包
安装完后，可以Clone，压缩打包成一个zip文件，方便分享给别人。

在关机之前，有一件事需要做，

	sudo vim /etc/sysconfig/network-scripts/ifcfg-eth0, 把HWADDR=.... 这行删掉
	sudo rm /etc/udev/rules.d/70-persistent-net.rules
	sudo shutdown -h now

如果没有执行上述命令，克隆后的虚拟机，开机后无法上网，重启网络，`sudo service network restart`也没有效果，会出现错误“Device eth0 does not seem to be present, delaying initialization.” 

这是因为克隆后的虚拟机，它的MAC地址变了，即在它的.vmx文件里，MAC地址变了（`ethernet0.generatedAddress`这项），但是linux不知道这个变化，网络配置文件还是旧的，这样跟它的而真实mac不匹配，网络就无法启动。

执行上述命令，删除了`70-persistent-net.rules`后，相当于删除了旧的配置文件，在开机时会生成新的配置文件。

关机后，右击标签，选择"Manage->Clone"，选择"Create a full clone"，克隆完成后，关闭这台虚拟机的标签（否则文件夹里有一些临时垃圾文件），然后把文件夹压缩打包。以后就可以把这个zip包拷贝给周围的人，别人就不用经历一遍重装的过程了。


##参考资料
[LAMP Server on CentOS 6](http://library.linode.com/lamp-guides/centos-6)

[CentOS - Installing Apache and PHP5](http://articles.slicehost.com/2008/2/6/centos-installing-apache-and-php5)

[Setting up a LAMP stack](http://fedorasolved.org/server-solutions/lamp-stack)

[CentOS5.5使用yum来安装LAMP](http://myohmy.blog.51cto.com/140917/327310)

[Install Java JDK on CentOS without prompts using an automated script!](http://it.megocollector.com/?p=1719)
