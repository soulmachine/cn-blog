---
layout: post
title: "在Centos 6.5上安装docker"
date: 2014-01-22 15:25
comments: true
categories: Docker
---

## 1 Enable EPEL Repo on CentOS

参考 [Enable EPEL Repo on CentOS 5 and CentOS 6](http://www.centosblog.com/enable-epel-repo-on-centos-5-and-centos-6/)

	rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

## 2 Install docker

	yum install docker-io --enablerepo=epel

## 3 启动 docker daemon 进程

	sudo docker -d &

这时，有警告，说内核版本过低，


> WARNING: You are running linux kernel version 2.6.32-431.el6.x86_64, which might be unstable running docker. Please upgrade your kernel to 3.8.0.

如果你在公司，且公司内部都是通过代理上网，则可以把代理服务器告诉docker，用如下命令(参考[这里](https://github.com/dotcloud/docker/issues/402))：

	sudo HTTP_PROXY=http://xxx:port docker -d &

## 4 升级内核

见我的另一篇博客，[CentOS 6.4 升级内核到 3.11.6](http://www.yanjiuyanjiu.com/blog/20131024)

## 5 下载 ubuntu 镜像

	sudo docker pull ubuntu

## 6 运行 hello world

	sudo docker run ubuntu /bin/echo hello world
	hello world

安装成功了！！
