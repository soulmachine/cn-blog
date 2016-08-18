---
title: 安装 Windows 10 和 Ubuntu 16.04 双系统
date: 2016-08-14 22:23:11
tags: 深度学习
categories: 深度学习
---

本文是上一篇文章[我的深度学习工作站攒机过程记录](http://cn.soulmachine.me/2016-08-13-my-deep-learning-workstation-assemble-process-note/)的续集。


## 前提条件

1. 主板BIOS是 UEFI 模式

    首先确认下你的主板的BIOS是UEFI模式，例如 Asus X99-E WS/USB 3.1 主板的BIOS默认就是UEFI模式。

1. 两个U盘，一个是可启动的Ubuntu 16.04.1 安装盘，一个是Windows 10 1607 x64 安装盘

    制作可启动安装盘很简单，下载 `cn_windows_10_multiple_editions_version_1511_updated_apr_2016_x64_dvd_8712460.iso` 和 `ubuntu-16.04.1-desktop-amd64.iso` ，找一台 Windows 电脑，安装 UltraISO这个小软件，启动软件，点击 `打开`，打开操作系统的ISO文件，点击菜单`启动->写入硬盘映像`，即可开始刻录U盘。

    如果你只有一个U盘，有没问题，先刻录Windows，装完了Windows后，格式化U盘，再刻录一个Ubuntu 进去。

1. 关闭主板的 Fast Startup

    进入 UEFI BIOS 界面，点击 Boot 菜单，找打 Fast Boot, 改为 `Disabled`

1. 关闭主板的 SRT（Intel Smart Response Technology)

    进入 UEFI BIOS 界面，点击 Advanced菜单，找到 Intel Rapid Storage Technology, 点击进去，如果你没有RAID模式的磁盘，这个RST一般是灰色的，没有启用。

1. 禁用主板的 Secure Boot

    如果你想要装Windows和 Linux 双操作系统，那么必须要禁用主板的 Secure Boot ，因为主板内置的公钥只有一个，就是微软的，因为微软影响力大。更具体信息请参考 [反Secure Boot垄断：兼谈如何在Windows 8电脑上安装Linux - 阮一峰](http://www.ruanyifeng.com/blog/2013/01/secure_boot.html)

    Ubuntu 已经买了这个证书，所以微软的这个公钥，也能允许Ubuntu安装在主板上。看到这里，你会问，那岂不是不用禁用 Secure Boot 了？

    不尽然，在Ubuntu 下安装 GTX 1080 的 Nvidia 驱动的时候，会警告 `UEFI Secure Boot is not compatible with the use of third-party drivers.` 如果没有禁用Secure Boot, 安装了显卡驱动后，开机，输入密码时，会进不去系统，屏幕会闪一下，有回到了登录界面，死循环 。。。

    具体步骤请参考 [How to Disable or Enable Secure Boot on Your Computer via ASUS UEFI BIOS Utility](http://www.technorms.com/45538/disable-enable-secure-boot-asus-motherboard-uefi-bios-utility) ：

    * 进入 UEFI BIOS界面，选择 `Boot->Secure Boot-> Key Management -> Save Secure Boot Keys`，插入U盘，备份key到这个U盘，会有四个文件,  `PK`, `KEK`, `DB` 和 `DBX` 写入到U盘。
    * 删除 Platform Key. 选择 `Delete PK`，点击OK确认删除。删除后回到上一级菜单，可以看到 Secure Boot 已经变成 disabled 了。

1. 顺序上，最好是先安装 Windows 再安装 Ubuntu，本文的方法1和方法2都是这个顺序
1. 安装了Wiondows后，必须要关闭Windows的 Fast Startup。

    进入`控制面板->电源`，找到 Fast Startup，禁用掉。


## Windows 和 Ubuntu 安装在同一块硬盘上

开机，按住 DEL 键进入BIOS，点击Boot Menu(F8)菜单，选择从U盘启动，且注意要选择UEFI模式的U盘

<!-- more -->

开始安装Windows, 安装完后，重启，进入Windows 后，关闭Windows的 Fast Startup，即进入`控制面板->电源`，找到 Fast Startup，禁用掉。

把 Ubuntu U盘启动盘插上，开机，按DEL键进入BIOS，选择从这个U盘启动，要选择UEFI模式的U盘，开始安装，记得选择 `Install Ubuntu alongside Windows Boot Manager`


## Windows 和 Ubuntu 分别安装不同的硬盘上

先按照方法一中的步骤，安装好Windows，然后关机，把这块硬盘拆下来，当做它不存在。

开始安装Ubuntu，安装到另一块硬盘上。选择 `Erase disk and install Ubuntu`，接下来选择安装到 SM951 NVMe SSD 这块磁盘上。

把 Windows 那块硬盘在接入主板。

这样，两块硬盘就分别安装了独立的操作系统，具体进入哪个，由UEFI BIOS里的启动优先级来决定。你想默认启动Windows，那就把Windows那块硬盘拖动到第一的位置，如果你想默认启动Ubuntu，那就把Ubuntu那块硬盘拖动到第一的位置。

【可选项】如果你不想通过UEFI BIOS来切换操作系统，你还可以把Windows加入 Ubuntu的 grub 菜单，开机进入Ubuntu后，执行

    sudo update-grub

这个命令会自动扫描其他硬盘上的操作系统，并加入grub 开机启动菜单。


## 参考资料

* [UEFI - Ubuntu](https://help.ubuntu.com/community/UEFI)
* [在Win8基础上加装Ubuntu，得先搞清楚Win8是以何种方式安装的](http://forum.ubuntu.org.cn/viewtopic.php?t=467746)
* [How to dual-boot Windows 10 and Ubuntu 15.10 on two hard drives](http://linuxbsdos.com/2015/10/31/how-to-dual-boot-windows-10-and-ubuntu-15-10-on-two-hard-drives/)
