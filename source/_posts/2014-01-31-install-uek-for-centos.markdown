---
layout: post
title: "给CentOS安装UEK内核"
date: 2014-01-31 22:05
comments: true
categories: DevOps
---

最近给CentOS 6.5 安装了docker，不过每次运行都会报警：

> WARNING: You are running linux kernel version 2.6.32-431.3.1.el6.x86_64, which might be unstable running docker. Please upgrade your kernel to 3.8.0.

给CentOS 升级内核，有三种途径，一种是yum官方源里有更新的版本，一种途径是自己编译，另一种途径是使用别人编译好了的内核。

CentOS yum源是除了名的更新慢，目前没有 3.8版内核，第二种途径很麻烦，工作量很大，因此本文用第三种。例如UEK，Oracle提供了一个公共的yum源，<http://public-yum.oracle.com/>

##添加yum源

UEK的稳定版还是 2.6 内核的，beta版的内核是3.8的，所以我们使用beta源

    sudo wget http://public-yum.oracle.com/beta/public-yum-ol6-beta.repo -P /etc/yum.repos.d

由于UEK3还没有加入到正式版本中，还目前属于测试阶段，，需要手工将 `enabled=0`改为 `enabled=1`

    sudo vim /etc/yum.repos.d/public-yum-ol6-beta.repo

##添加GPG key

    sudo wget http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6 -O /etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
    gpg --quiet --with-fingerprint /etc/pki/rpm-gpg/RPM-GPG-KEY-oracle

##更新yum源

    sudo yum update

##列出所有的kernel

    yum list kernel*
    
<!-- more -->

##安装 kernel

    sudo yum install kernel-uek

##修改/etc/grub.conf
修改 `/etc/grub.conf`，将`default`设置为UEK，一般新安装的内核会在一个，所以设置 `default=0`。

##重启

    sudo reboot
    uname -r
    3.8.13-16.el6uek.x86_64

内核切换成功了。
    
##测试一下，docker不再报警了

    sudo docker service docker stop
    sudo docker -d

`Ctrl+C` 终止 docker，然后用 `sudo service docker start` 再次启动docker。

##参考资料
[UEK R3升级手记](http://blog.liulantao.com/Technology/2013/09/23/kernel-uek-3813-upgrade-notes.html)

