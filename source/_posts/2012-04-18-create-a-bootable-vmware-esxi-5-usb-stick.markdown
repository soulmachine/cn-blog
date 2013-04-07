---
layout: post
title: "制作 VMware ESXI 5 U盘安装盘"
date: 2012-04-18 20:17
comments: true
categories: tools
---
有两种方法，使用 unetbootin ，或使用 LinuxLive USB Creator刻录可启动U盘。 
##使用 unetbootin
这个方法最自动化，点击两下按钮即可，但是有时候会失败(我用EXSi 5.0 的ISO失败，但是用 EXSi 5.0U1可以成功)，U盘启动不了。 

1. 单击 光盘镜像，选择ISO文件VMware-VMvisor-Installer-5.0.0.update01-623860.x86_64.iso。 
2. 选择U盘，点击“确定”开始刻录。刻录后用U盘启动机器开始安装即可。如下图所示。
{% img http://yanjiuyanjiu-wordpress.stor.sinaapp.com/uploads/2012/04/unetbootin1.jpg %}

##使用 LinuxLive USB Creator

1. 格式化U盘，文件系统为FAT32，并设置为主分区，命令如下：
		# 使用管理员权限运行cmd 
		diskpart 
		list disk 
		select disk USB number （例如 select dist 1） 
		clean 
		create partition primary 
		active 
		format fs=fat32 quick 
		assign 
		exit

<!-- more -->

2. 下载，安装 LinuxLive USB Creator([http://www.linuxliveusb.com/](http://www.linuxliveusb.com/)) 
3. 按照上图中的步骤 1,2,4，选择ISO文件`VMware-VMvisor-Installer-5.0.0.update01-623860.x86_64.iso`，然后点击 5 ，开始创建U盘安装盘。等待U盘刻录结束。

	{% img http://yanjiuyanjiu-wordpress.stor.sinaapp.com/uploads/2012/04/liliusb_thumb1.png %}

	大功告成，是不是很简单？！ 

4. 编辑U盘根目录下的BOOT.CFG文件。注意，不要添加 "ks=usb"，因为下面会用交互模式来安装。 
5. 大功告成
6. 注意，本文主要参考了末尾的参考资料。但是不需要原文的第4步和第5步。因为用普通的 “interactive installation”安装就很方便了。第4步和第5步用于一键自动化安装，适用于大量安装的情况，这里不详细讨论。 
见文章末尾的评论，

> @Cesar: if you do not edit the boot.cfg with the “ks=usb” option and select a interactive installation it will work。

**参考资料**  
[Create a bootable VMware ESXi 5 USB stick in Windows and perform a scripted installation](http://www.ivobeerens.nl/2011/09/17/create-a-bootable-vmware-esxi-5-usb-stick-in-windows-and-perform-a-scripted-installation/)