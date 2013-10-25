---
layout: post
title: "docker 快速入门"
date: 2013-10-26 23:41
comments: true
categories: docker
---

**前提条件：**要安装好 docker，见我的另一篇博客，[docker 安装](http://www.yanjiuyanjiu.com/blog/20131025)


## 交互式命令行入门教程

首先强烈建议玩一遍官方的一个交互式命令行入门教程，[Interactive commandline tutorial](http://www.docker.io/gettingstarted/)。甚至要多玩几遍，加深印象。

初次过完这个教程，感觉docker用起来跟git很类似。

玩完了后，在自己的真实机器上，把上面的命令重新敲一遍，感受一下。


### 添加http代理

在国内，pull或push的时候经常连不上docker.com（原因你懂的，或者在公司内部统一用一个代理上网的时候），可以在docker daemon进程启动的时候加个代理，例如

	sudo HTTP_PROXY=proxy_server:port docker -d &

同时要注意，不要使用`http_proxy`和`https_proxy`环境变量。

docker貌似是不识别`http_proxy`, `https_proxy`和`no_proxy`环境变量的，因此要在命令行里指定。

如果在命令行里指定了`HTTP_PROXY`，则要unset掉`http_proxy`和`https_proxy`环境变量。原因是：

* 首先， docker daemon进程是通过http协议与docker.com通信的
* 其次，docker的各种命令（例如 `run`, `login`等）也是通过http协议与docker daemon进程通信的（发送jasn字符串，daemon进程返回的也是json字符串），有时候docker客户端命令貌似能识别http_proxy变量，这时，客户端发送一个命令，路径是`localhost->http_proxy->daemon进程`，daemon进程返回的数据，路径是 `daemon进程->proxy->proxy->localhost`，其中，从`proxy->localhost`的路径是不通的，因为proxy连接不了内网IP。

之所以把这一步放在本文开始，是因为这一步不做的话，后面很多命令会出错，让人摸不着头脑，我在这里就掉进坑了，花了很长时间才搞明白，原来是网络连接不稳定。


## 熟悉一下 Dockerfile

完了几遍交互式入门教程后，你会好奇，怎么自己定制一个 image，例如把常用的软件装好后打包 ? 这时候该 Dockfile 登场了。Dockerfile 实质上是一个脚本文件，用于自动化创建image。

<-- more -->

请跟着官方教程走一遍，[Dockerfile Tutorial](http://www.docker.io/gettingstarted/)

到这里， 官网的 Getting started 的内容基本上消化完了，接下来就是翻官网的[Documentation](http://docs.docker.io/en/latest/)了。


## 删除容器

每一行命令都会产生一个新的容器（无论是在`sudo docker run -i -t ubuntu /bin/bash` 模式下，还是Dockerfile里的RUN命令），玩了一会儿后，`sudo docker ps -a` 会看到很多容器，很是干扰视线，可以用一行命令删除所有容器：

	sudo docker rm `sudo docker ps -a -q`

## 创建image
有两种用方式，

* 写一个Dockerfile，然后用`docker build`创建一个image
* 在容器里交互式地（例如`sudo docker run -i -t ubuntu /bin/bash`）进行一些列操作，然后`docker commit`固化成一个image。

image相当于编程语言里的类，container相当于实例，不过可以动态给实例安装新软件，然后把这个container用commit命令固化成一个image。

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




