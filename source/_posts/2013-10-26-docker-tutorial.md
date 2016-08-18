---
layout: post
title: "docker 快速入门"
date: 2013-10-26 23:41
comments: true
categories: Docker
---

**前提条件：**要安装好 docker，见我的另一篇博客，[docker 安装](http://www.yanjiuyanjiu.com/blog/20131025)


## 交互式命令行入门教程
首先强烈建议玩一遍官方的一个交互式命令行入门教程，[Interactive commandline tutorial](http://www.docker.io/gettingstarted/)。甚至要多玩几遍，加深印象。

初次过完这个教程，感觉docker用起来跟git很类似。

玩完了后，在自己的真实机器上，把上面的命令重新敲一遍，感受一下。


## Hello World
参考官方文档[Hello World](http://docs.docker.io/en/latest/examples/hello_world/#running-examples)

首先下载官方的ubuntu image:

	sudo docker pull ubuntu

然后运行 hello world：

	sudo docker run ubuntu /bin/echo hello world

## 三种运行命令的模式
docker 有三种运行命令的方式，短暂方式，交互方式，daemon方式。

**短暂方式**，就是刚刚的那个"hello world"，命令执行完后，container就终止了，不过并没有消失，可以用 `sudo docker ps -a` 看一下所有的container，第一个就是刚刚执行过的container，可以再次执行一遍：

	sudo docker start container_id

不过这次看不到"hello world"了，只能看到ID，用`logs`命令才能看得到，

	sudo docker logs container_id

可以看到两个"hello world"，因为这个container运行了两次。


**交互方式**，

	sudo docker run -i -t image_name /bin/bash

**daemon方式**，即让软件作为长时间服务运行，这就是SAAS啊！

例如，一个无限循环打印的脚本（替换为memcached, apache等，操作方法仍然不变！）：

	CONTAINER_ID=$(sudo docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done")

在container外面查看它的输出

	sudo docker logs $CONTAINER_ID

或者连接上容器实时查看

	sudo docker attach $CONTAINER_ID

终止容器

	sudo docker stop $CONTAINER_ID

`sudo docker ps`看一下，已经没了


## docker ps 命令详解
`sudo docker ps`，列出当前所有正在运行的container

`sudo docker ps -l`，列出最近一次启动的，且正在运行的container

`sudo docker ps -a`，列出所有的container

其他用法请参考 `sudo docker ps -h`

还有一种方式可以让程序在daemon模式下运行，就是在Dockerfile里设置USER为daemon，见[Dockerfile tutorial Level2](http://www.docker.io/learn/dockerfile/level2/)。


## 添加http代理
在国内，pull或push的时候经常连不上docker.com（原因你懂的，或者在公司内部统一用一个代理上网的时候），可以在docker daemon进程启动的时候加个代理，例如

	sudo HTTP_PROXY=proxy_server:port docker -d &

docker貌似是不识别`http_proxy`, `https_proxy`和`no_proxy`环境变量的，因此要在命令行里指定，参考 [Github Issue #402 Using Docker behind a firewall](https://github.com/dotcloud/docker/issues/402)。

如果在命令行里指定了`HTTP_PROXY`，则要unset掉`http_proxy`和`https_proxy`环境变量。原因是：

* 首先， docker daemon进程是通过http协议与docker.com通信的
* 其次，docker的各种命令（例如 `run`, `login`等）也是通过http协议与docker daemon进程通信的（发送jasn字符串，daemon进程返回的也是json字符串），有时候docker客户端命令貌似能识别http_proxy变量，这时，客户端发送一个命令，路径是`localhost->http_proxy->daemon进程`，daemon进程返回的数据，路径是 `daemon进程->proxy->proxy->localhost`，其中，从`proxy->localhost`的路径是不通的，因为proxy连接不了内网IP。

之所以把这一步放在本文开始，是因为这一步不做的话，后面很多命令会出错，让人摸不着头脑，我在这里就掉进坑了，花了很长时间才搞明白，原来是网络连接不稳定。


## 熟悉一下 Dockerfile
完了几遍交互式入门教程后，你会好奇，怎么自己定制一个 image，例如把常用的软件装好后打包 ? 这时候该 Dockfile 登场了。Dockerfile 实质上是一个脚本文件，用于自动化创建image。

<!--more-->

请跟着官方教程走一遍，[Dockerfile Tutorial](http://www.docker.io/gettingstarted/)

到这里， 官网的 Getting started 的内容基本上消化完了，接下来就是翻官网的[Documentation](http://docs.docker.io/en/latest/)了。

## 我的第一个Dockerfile
文件名为update.dockerfile，内容如下：

	# use the ubuntu base image provided by dotCloud
	FROM ubuntu
	
	MAINTAINER Frank Dai soulmachine@gmail.com
	
	
	# if you're behind a government firewall or company proxy
	# ENV http_proxy http://proxy-prc.intel.com:911
	# ENV https_proxy https://proxy-prc.intel.com:911
	# RUN echo "Acquire::http::proxy \"proxy_server:port\";" >> /etc/apt/apt.conf
	# RUN echo "Acquire::https::proxy \"proxy_server:port\";" >> /etc/apt/apt.conf
	
	
	# choose a faster mirror, see http://t.cn/zWYrzCE
	RUN echo "deb mirror://mirrors.ubuntu.com/mirrors.txt precise main restricted universe multiverse" > /etc/apt/sources.list
	RUN echo "deb mirror://mirrors.ubuntu.com/mirrors.txt precise-updates main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb mirror://mirrors.ubuntu.com/mirrors.txt precise-backports main restricted universe multiverse" >> /etc/apt/sources.list
	RUN echo "deb mirror://mirrors.ubuntu.com/mirrors.txt precise-security main restricted universe multiverse" >> /etc/apt/sources.list
	
	RUN apt-get update -y
	# currently docker official ubuntu image has a problem with apt-get upgrade
	# RUN apt-get upgrade -y && apt-get dist-upgrade -y
	RUN apt-get clean all
	
	
	# install wget
	RUN apt-get install -y  wget
	RUN wget www.baidu.com
	RUN rm index.html
	
	#install vim editor
	RUN apt-get install -y vim

**注意一个坑：** `apt-get upgrade` 在当前官方的ubuntu image里是无法运行成功的，见 [Issue #1724 apt-get upgrade in plain Ubuntu precise image fails](https://github.com/dotcloud/docker/issues/1724)。所以，干脆放弃更新吧，能`apt-get install`软件就行了，不要有更新强迫症 :)


## 删除容器
**每一行命令都会产生一个新的容器**（无论是在`sudo docker run -i -t ubuntu /bin/bash` 模式下，还是Dockerfile里的RUN命令），玩了一会儿后，`sudo docker ps -a` 会看到很多容器，很是干扰视线，可以用一行命令删除所有容器：

	sudo docker rm `sudo docker ps -a -q`

## 创建image
有两种用方式，

* 写一个Dockerfile，然后用`docker build`创建一个image
* 在容器里交互式地（例如`sudo docker run -i -t ubuntu /bin/bash`）进行一些列操作，然后`docker commit`固化成一个image。

image相当于编程语言里的类，container相当于实例，不过可以动态给实例安装新软件，然后把这个container用commit命令固化成一个image。


使用前面写好的update.dockerfile，创建一个image：

	sudo docker build -t soulmachine/ubuntu:latest - < update.dockerfile


## 下载image
<https://index.docker.io/> 是官方的image仓库，也可以用 [Docker-Registry](https://github.com/dotcloud/docker-registry)创建自己的仓库，这就好比git，<https://index.docker.io/>相当于Github，也可以自己DIY搭建一个git服务器，把自己的代码托管到私有的服务器上。

`sudo docker pull ubuntu` 是从 <https://index.docker.io/_/ubuntu/> 下载名为 ubuntu 的repo，里面包含了几个tag，默认使用latest这个tag。这个repo是docker官方的。


## 上传并共享image
自己build了一个image，想要共享，怎么办？参考这篇文章，[How to build and publish base boxes for Docker?](http://crohr.me/journal/2013/docker-repository-create-base-boxes.html)

### 注册一个账号
首先，要去 <https://index.docker.io/> 注册一个账号，例如我的是 soulmachine。

### build一个image
build命令格式如下：

	sudo docker build -t username/repo:tag - < Dockerfile

如果没有tag，则默认为 latest。

### 登陆

	sudo docker login

输入自己的用户名和密码。

### push 到 Docker index

	sudo docker push username/repo

这条命令会把一个repo下面的所有tag都push到<https://index.docker.io/>


## 安装JDK7失败
我在container 里面安装jre7是可以的, `apt-get install openjdk-7-jre-headless` 成功。但是安装jdk7，`apt-get install openjdk-7-jdk`失败，错误消息如下：

	Creating fuse device...
	mknod: `fuse-': Operation not permitted
	makedev fuse c 10 229 root root 0660: failed
	chown: cannot access `/dev/fuse': No such file or directory
	dpkg: error processing fuse (--configure):
	 subprocess installed post-installation script returned error exit status 1
	Processing triggers for libc-bin ...
	ldconfig deferred processing now taking place
	Errors were encountered while processing:
	 fuse
	E: Sub-process /usr/bin/dpkg returned an error code (1)

原因是权限不够，见[Issue #963](https://github.com/dotcloud/docker/issues/963) 和 [Issue #514](https://github.com/dotcloud/docker/issues/514)

所以，需要在启动container时，添加一个`-priviledged`参数，

	sudo docker run -i -t -priviledged soulmachine/ubuntu /bin/bash

在里面执行`apt-get install openjdk-7-jdk`，这次成功了。

那如何在dockerfile里用RUN命令安装JDK7呢？dockerfile里没法指定`-priviledged`，目前没有办法，不过可以折中一下，安装openjdk6。


## ENTRYPOINT 和 CMD 的区别
见[How to Use Entrypoint in Docker Builder](http://www.kstaken.com/blog/2013/07/06/how-to-use-entrypoint-in-a-dockerfile/)


## docker 最佳实践
见[Dockerfile Best Practices](http://crosbymichael.com/dockerfile-best-practices.html)


## 关于docker 的书籍
* [The Docker Book](http://dockerbook.com/)


## 底层原理

image, container, fs layer，是什么关系？见这篇博客，[Solomon Hykes Explains Docker](http://www.activestate.com/blog/2013/06/solomon-hykes-explains-docker)


## 参考资料
1. [Official docs](http://docs.docker.io/)
2. [Docker 初探](https://log.qingcloud.com/?p=129)
