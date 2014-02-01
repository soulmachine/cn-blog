---
layout: post
title: "SSH无密码登录的配置"
date: 2012-01-02 02:16
comments: true
categories: DevOps
---

根据公钥加密的思想，如果机器A想无密码登录其他N台机器，只需要在自己机器上生成一对公钥和密钥，然后把密钥给这N台机器，这样，这N台机器，有了A的公钥，就可以解密A的数据包，跟A正常通信了。

通常在一个集群中，我们会选择一台机器作为跳板机，在这台机器上登录其他机器，因此，无名需要在跳板机上生成一对公钥和密钥。一般的，我们也会把跳板板机作为整个集群的master，例如Hadoop的NameNode，因此最好选一台内存比较大的机器作为跳板机。

下面详细讲解如何配置从跳板机SSH无密码登录到所有机器（包括自己）。

根据公钥加密的思想，如果机器A想无密码登录其他N台机器，只需要在自己机器上生成一对公钥和密钥，然后把密钥给这N台机器，这样，这N台机器，有了A的公钥，就可以解密A的数据包，跟A正常通信了。

通常在一个集群中，我们会选择一台机器作为跳板机，在这台机器上登录其他机器，因此，无名需要在跳板机上生成一对公钥和密钥。一般的，我们也会把跳板板机作为整个集群的master，例如Hadoop的NameNode，因此最好选一台内存比较大的机器作为跳板机。

下面详细讲解如何配置从跳板机SSH无密码登录到所有机器（包括自己）。

##前提： 修改hosts文件
假设有三台机器，`192.168.1.131, 192.168.1.132, 192.168.1.133`，hostname分别是master, slave01, slave02

##1.在master上生成一对公钥和密钥

    dev@master:~$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
    
##2. 将公钥拷贝到自己

    dev@master:~$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

##3. 将公钥拷贝到其他机器

    dev@master:~$ scp ~/.ssh/id_dsa.pub dev@slave01:~
    dev@master:~$ scp ~/.ssh/id_dsa.pub dev@slave02:~
    #追加到authorized_keys
    dev@master:~$ ssh slave01
    dev@slave01:~$ mkdir .ssh
    dev@slave01:~$ cat id_dsa.pub >> .ssh/authorized_keys
    dev@slave01:~$ exit
    dev@master:~$ ssh slave02
    dev@slave02:~$ mkdir .ssh
    dev@slave02:~$ cat id_dsa.pub >> .ssh/authorized_keys
    dev@slave02:~$ exit

##4. 设置.ssh目录和authorized_keys文件的权限
在被登录的每台机器上，执行如下命令：

    chmod 755 .ssh
    chmod 600 ~/.ssh/authorized_keys

##5. 测试一下

    #在 master执行
    dev@master:~$ ssh slave01

第一次还是需要密码的，`exit`退出再试一次，就不需要密码了。

如果登陆不上，试试先关闭所有机器的防火墙，例如Ubuntu的命令是：

	dev@slave01:~$ sudo ufw disable

##参考资料
[Howto Linux / UNIX setup SSH with DSA public key authentication (password less login)](http://www.cyberciti.biz/faq/ssh-password-less-login-with-dsa-publickey-authentication/)

[HOWTO: Generating SSH Keys for Passwordless Login](http://hortonworks.com/kb/generating-ssh-keys-for-passwordless-login/)
