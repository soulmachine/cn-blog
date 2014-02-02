---
layout: post
title: "集群时间同步--架设内网NTP服务器"
date: 2014-01-24 16:40
comments: true
categories: DevOps
---

环境：CentOS 6.5

对于一个Linux集群，集群内的机器保持时间同步是很重要的，不然会出现很多问题。

本文主要描述如何在集群内架设一台NTP服务器，其他机器都与这台服务器保持时间同步。

##1 安装NTP
在所有机器上执行，

    $ sudo yum install ntp

###2 调整时区
把所有机器的时区调整为上海时区，即"+8000"时区。

先看一下机器的时区是否是对的，

    $ date -R

如果不是"+8000"，则要修改时区，

    $ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

###3 （可选）同步BIOS时间
Linux系统上面BIOS时间与linux系统时间是分开的，所以调整了时间之后，还需要使用`hwclock`才能将修改过的时间写入BIOS中。

在`/etc/sysconfig/ntpd`中添加一行:

    SYNC_HWCLOCK=yes

##4 配置NTP服务器
选择一台能够上网的机器作为NTP服务器，以后这台服务器提供时间同步服务，集群内的其他机器不需要上网去跟公共的NTP服务器同步了。

<!-- more -->

###4.1 修改/etc/ntp.conf
ntp只有一个配置文件, `/etc/ntp.conf`.

只需修改一行，找到下面这行，取消注释，

    #restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

变成了

    restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

这一行的含义是允许所有IP为`192.168.1.x`的机器与本服务器同步，这样就把这台机器变成了一台NTP服务器，对别的机器提供NTP同步服务。

###4.2 开机启动ntpd

    $ sudo chkconfig ntpd on

###4.3 启动ntpd

    $ sudo service ntpd start

###4.4 验证与状态检查

####4.4.1 查看ntp的端口

    $ netstat -unlnp

应该看到123端口

####4.4.2 查看ntp服务器有无和上层连通

    $ ntpstat

    synchronised to NTP server (218.75.4.130) at stratum 3 
       time correct to within 598 ms
       polling server every 64 s

####4.4.3 查看ntp服务器与上层间的联系

    $ ntptrace -n 127.0.0.1

    127.0.0.1: stratum 3, offset -0.001095, synch distance 0.532610
    116.193.83.174: timed out, nothing received

####4.4.4 查看ntp服务器与上层ntp服务器的状态

    $ ntpq -p
         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
    +dns.sjtu.edu.cn 202.112.31.197   3 u   47   64    3   28.873   76.001 114.194
    +Hshh.org        209.51.161.238   2 u   48   64    3   43.694   75.042 131.058
    +202.118.1.130   202.118.1.46     2 u   46   64    3   14.640   75.636 116.999
    *dns1.synet.edu. 202.118.1.46     2 u   44   64    3   13.968   74.514 128.913

其中:

* remote - 本机和上层ntp的ip或主机名，“+”表示优先，“*”表示次优先
* refid - 参考上一层ntp主机地址
* st - stratum阶层
* t: 这个.....我也不知道啥意思^_^
* when - 多少秒前曾经同步过时间
* poll - 下次更新在多少秒后
* reach - 已经向上层ntp服务器要求更新的次数
* delay - 网络延迟
* offset - 时间补偿
* jitter - 系统时间与bios时间差

如果所有远程服务器的jitter值是4000并且delay和reach的值是0，那么说明时间同步是有问题的，可能的原因是防火墙阻断了与server之间的通讯，检查一下123端口是否正常开放。

##5 配置客户机
服务器配置好了，接下来就要配置所有的客户端机器，从该服务器同步时间。

* 方法1，使用`ntpdate`与上面配置的NTP服务器定时同步（参考资料2），不推荐此方法
* 方法2，安装ntpd，指定NTP服务器为上面配置的服务器地址，推荐。

下面详细讲述方法2。以下操作适用于所有客户端机器。

###5.1 指定NTP服务器

删除 `/etc/ntp.conf` 里的所有公网ntp服务器，换成上面配置的服务器，

    #server 0.centos.pool.ntp.org iburst
    #server 1.centos.pool.ntp.org iburst
    #server 2.centos.pool.ntp.org iburst
    #server 3.centos.pool.ntp.org iburst
    server techwolf-01 iburst

用hostname或ip都可以。

###5.2 开机启动ntpd

    $ sudo chkconfig ntpd on

###5.3 启动ntpd

    $ sudo service ntpd start


##参考资料

1. [CentOS配置时间同步NTP](http://www.crsay.com/wiki/wiki.php/server/centos/ntp-set)
1. [CentOS系统时间同步（NTP）](http://www.cnblogs.com/thinksasa/p/3479980.html)


