---
layout: post
title: "Ansible快速入门"
date: 2013-10-16 16:53
comments: true
categories: DevOps
published: false
---
Ansible 是一个新工具，比Puppet,Chef要轻量很多。只需要在一台机器上安装，slave机器不需要安装。有点像一个增强版的pssh。

本文主要参考官方文档 [docs](http://www.ansibleworks.com/docs/)

环境：CentOS 6.4

## 安装 ansible
选一台机器作为master，在上面安装Ansible。

	sudo yum install ansible

## 配置SSH无密码登陆
要让master能够无密码登陆所有的slave机器。参考我另一篇博客，[在CentOS上安装Hadoop](http://www.yanjiuyanjiu.com/blog/20130612/)，里面的“配置 master 无密码登陆到所有机”这节。

## 测试一下

	ansible all -a "/bin/echo hello"

如果hosts文件里的所有机器都返回了结果，说明安装成功了。


