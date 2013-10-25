---
layout: post
title: "docker安装"
date: 2013-10-25 23:41
comments: true
categories: docker
---

## 1 在 CentOS 6.4 上安装 docker

docker当前官方只支持Ubuntu，所以在 CentOS 安装Docker比较麻烦。

docker官方文档说要求Linux kernel至少3.8以上，CentOS 6.4是2.6的内核，于是我哼哧哼哧的[编译安装了最新的kernel 3.11.6](http://www.yanjiuyanjiu.com/blog/20131024)，重启后运行docker还是失败，最后找到原因，是因为编译时忘记集成aufs模块了。aufs 需要和 kernel 一起编译，很麻烦。

不过不需要这么麻烦，有强人已经编译好了带aufs模块的内核，见这里[Installing docker.io on centos 6.4 (64-bit)](http://nareshv.blogspot.com/2013/08/installing-dockerio-on-centos-64-64-bit.html)

### 1.1 取消selinux，因为它会干扰lxc的正常功能

	sudo vim /etc/selinux/config 
	SELINUX=disabled
	SELINUXTYPE=targeted

### 1.2 安装 Fedora EPEL 

	sudo yum install http://ftp.riken.jp/Linux/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm

### 1.3 添加 hop5 repo地址

	cd /etc/yum.repos.d
	sudo wget http://www.hop5.in/yum/el6/hop5.repo

### 1.4 安装 docker-io

	sudo yum install docker-io

会自动安装带aufs模块的3.10内核，以及docker-io包。

### 1.5 将 cgroup 文件系统添加到 `/etc/fstab` , 只有这样docker才能正常工作

	sudo echo "none                    /sys/fs/cgroup          cgroup  defaults        0 0" >> /etc/fstab

### 1.6 修改grub引导顺序

	sudo vim /etc/grub.conf
	default=0

设置default为新安装的内核的位置，一般是0

### 1.7 重启

	sudo reboot

### 1.8 检查新内核是否引导成功

重启后，检查一下新内核是否引导起来了

	uname -r
	3.10.5-3.el6.x86_64

说明成功了

看一下 aufs是否存在

	grep aufs /proc/filesystems 
	nodev   aufs

说明存在

### 1.9 启动 docker daemon 进程

	sudo docker -d &

如果你在公司，且公司内部都是通过代理上网，则可以把代理服务器告诉docker，用如下命令(参考[这里](https://github.com/dotcloud/docker/issues/402))：

	sudo HTTP_PROXY=http://xxx:port docker -d &

### 1.10 下载 ubuntu 镜像

	sudo docker pull ubuntu

### 1.11 运行 hello world

<--- more -->

	sudo docker run ubuntu /bin/echo hello world
	hello world

安装成功了！！


## 2 在 Ubuntu 上安装 docker

见官方文档，[Ubuntu Linux](http://docs.docker.io/en/latest/installation/ubuntulinux/)