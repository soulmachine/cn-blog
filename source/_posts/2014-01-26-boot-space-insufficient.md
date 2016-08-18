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

##3. 删除没有使用的内核

    rpm -e 2.6.32-431.el6.x86_64
    rpm -e xxx

将`rpm -q kernel`显示的内核复制粘贴到`xxx`位置。

## 4. 手动删除其他文件
把所有未使用的版本全部删除。

    sudo rm -rf /lib/modules/2.6.32-431.el6.x86_64
    sudo rm -rf /usr/src/kernels/2.6.32-431.el6.x86_64
    sudo rm -rf /usr/src/kernels/2.6.32-431.el6.x86_64.debug
    sudo rm /boot/*2.6.32-431*

##5. 删除grub里的条目
上面的步骤做完了后，最后，把grub里未使用的内核删掉，条目序号是从0开始编号的，删除条目后，记得把`default`设置为正确的序号。

##6. 再执行 yum update

    sudo yum -y update


##参考资料
[boot目录空间不足]( http://rajaruan.blog.51cto.com/2771737/868293)

[yum update -y，提示/boot 空间不足的解决方法]( http://www.xiaohuai.com/3301)

[如何卸载自己编译的内核？【已解决，方法见6L】](http://forum.ubuntu.org.cn/viewtopic.php?f=97&t=334647)
