---
layout: post
title: "install texlive 2013 iso"
date: 2014-02-12 02:01
comments: true
categories: Tools
published: false
---

下载光盘

sudo mkdir /mnt/cdrom
sudo mount -o loop path-to-iso /mnt/cdrom

cd /mnt/cdrom
sudo ./install-tl

如果想图形界面安装，则先安装perl-tk

sudo apt-get install perl-tk

默认都是full-scheme安装的，即完整版

默认安装在 /usr/local/texlive

将/usr/local/texlive/2013/bin加入PATH



sudo mkdir /usr/share/fonts/AdobeFonts
拷贝四个字体文件到这个目录
sudo chmod 644 /usr/share/fonts/AdobeFonts/*
刷新字体缓存

　　　　sudo  mkfontscale

　　　　sudo mkfontdir

　　　　sudo fc-cache -fsv

首先，查看系统中安装的中文字体的名字。

　　　　fc-list :lang=zh | sort

http://blog.csdn.net/ustc_dylan/article/details/6196129


