---
layout: post
title: "Rdesktop 快速入门"
date: 2014-01-25 22:05
comments: true
categories: SA
---

rdesktop是一款Linux下兼容Windows Remote Desktop Protocal(RDP)协议的客户端，可以用它连接开启了3389的windows机器。输入`rdesktop`可以看到该命令的所有选项，其中最常用的选项如下：
> -u: user name
> -p: password (- to prompt)
> -f: full-screen mode
> -g: desktop geometry (WxH)
> -x: RDP5 experience (m[odem 28.8], b[roadband], l[an] or hex nr.)

举几个例子，

    rdesktop -u feng -p 123456 -xl -f 192.168.1.250

这条命令表示，-xl表示客户端和win机器在同一个局域网，因此可以画质调节到最好，-f表示全屏，这条命令最好在局域网下使用。

    rdesktop -u feng -p 123456 -xm -f 192.168.1.250

跟上一条命令相比，把-xl换成了-xm，画质调节到最差

    rdesktop -u feng -p 123456 -xm -g 1024x768 192.168.1.250

跟上一条命令相比，将全屏改成了分辨率1024x768