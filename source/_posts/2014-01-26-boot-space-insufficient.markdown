---
layout: post
title: "/boot 目录空间不足"
date: 2014-01-26 22:07
comments: true
categories: DevOps
---
今天在服务器上执行 `sudo yum -y update`的时候报错：

> ... needs 18MB on the /boot filesystem

## 1. 列出所有的内核文件

    rpm -q kernel
    kernel-2.6.32-431.el6.x86_64
    kernel-2.6.32-431.3.1.el6.x86_64

发现有多个内核，因此可以删除所有不再使用的内核文件，来释放空间。

##2. 查看当前正在使用的内核

    uname -r
    2.6.32-431.3.1.el6.x86_64

注意，如果你是刚刚 `yum -y update`过，需要重启一下，内核才会更新，不重启的话`uname -r`还是显示的旧的。

##3. 删除所有没有使用的内核

    rpm -e 2.6.32-431.el6.x86_64
    rpm -e xxx

将`rpm -q kernel`显示的内核复制粘贴到`xxx`位置。

## 4. 手动删除/boot下的其他文件
如果/boot下还有其他内核版本的文件，可以把他们全部删除。

    cd /boot
    sudo rm initramfs-2.6.32-431.el6.x86_64.img 
    sudo rm initramfs-2.6.32-431.el6.x86_64.debug.img
    sudo rm config-2.6.32-431.el6.x86_64.debug 
    sudo rm System.map-2.6.32-431.el6.x86_64.debug
    sudo rm vmlinuz-2.6.32-431.el6.x86_64.debug
    sudo rm symvers-2.6.32-431.el6.x86_64.debug.gz

##5. 再执行 yum update

    sudo yum -y update


##参考资料
[boot目录空间不足]( http://rajaruan.blog.51cto.com/2771737/868293)

[yum update -y，提示/boot 空间不足的解决方法]( http://www.xiaohuai.com/3301)
