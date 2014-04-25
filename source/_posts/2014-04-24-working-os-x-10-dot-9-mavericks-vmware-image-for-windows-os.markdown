---
layout: post
title: "在Windows上直接使用现成的 OS X 10.9 Mavericks VMware镜像"
date: 2014-04-24 21:58
comments: true
categories: DevOps
---

环境: Windows 7, VMware Workstation 10.0.2, Mac OS X 10.9 Mavericks

本文主要参考了这篇英文博客，[Working OS X 10.9 Mavericks VMware Image For Windows OS and Intel Processor](http://www.sysprobs.com/working-os-x-10-9-mavericks-vmware-image-for-windows-os-intel-processor)

**前提条件**：确保你的电脑支持`VT-x`技术，并在BIOS里启用它。

##1. 下载别人做好的镜像
由于版权原因，这篇英文博客的作者不再提供下载地址了，不过google一下 "OS X Mavericks 10.9 Retail VMware Image"，可以在海盗湾找到种子，[这里](http://thepiratebay.se/torrent/9012642/OS_X_Mavericks_10.9_Retail_VMware_Image)。用迅雷或BT客户端下载。还有一种用更快速的下载方法，用百度网盘的离线下载，把magnet链接复制粘贴到百度网盘，等百度网盘下载好了后，再用百度管家客户端下载下来。

这个文件夹里，只有`OS X Mavericks 10.9 Retail VMware Image.7z`是有用的额，其他小工具可以分别去下载最新的版本。

这个镜像就是别人制作好的“懒人版”，Google 搜索"OS X Mavericks VMware 懒人版"或"OS X Mavericks VMware 整合驱动版"，还可以搜到很多。

##2. 给 VMware Workstation 打补丁
Windows上的VMware Workstation在安装操作系统时不支持 Mac OS X，但 VMware Fusin(VMware Workstation在Mac上的等价物，叫做 VMware Fusion)是支持了，这两个软件基本是统一软件在不同操作系统上的版本，按道理VMware Workstation也支持Mac OS X，事实上的确如此，只需要给VMware Workstation打一个补丁，就可以支持Mac OS X了。

补丁名字叫做VMware Unlocker for OS X，在[这里下载](http://www.insanelymac.com/forum/files/file/20-vmware-unlocker-for-os-x/)。

下载完后，解压，浏览到`windows`，以管理员权限执行 `install.cmd`，然后启动VMware Workstation，就可以看到变化了。

<!-- more -->

打补丁之前，

![](/images/before-unlocker.png)

打补丁之后。

![](/images/after-unlocker.png)

##3. 启动虚拟机
解压 `OS X Mavericks 10.9 Retail VMware Image.7z`，双击`.vmx`文件，就可以打开虚拟机了。

在启动虚拟机之前，你可以修改一下虚拟机的设置，例如提高内存到4GB，设置CPU核数为2，视你的电脑硬件配置而定。

启动虚拟机后，要做一些设置，例如键盘，icloud账户，设置密码等等。

##4. 安装VMware Tools
VMware Tools for OS X最新版来自 <http://softwareupdate.vmware.com/cds/vmw-desktop/fusion/> ，当前最新版是 6.0.3。下载`com.vmware.fusion.tools.darwin.zip.tar` ，解压出里面的 `darwin.iso` ，然后 mount 到 Mac OS X 虚拟机，一定要勾选"已连接"，然后 Mac OS X 虚拟机会自动弹出安装对话框。

![](/images/mount-vmware-tools.png)

##5. 安装显卡驱动
点击全屏菜单，发现Mac OS X 虚拟机不能这是由于没有显卡驱动。去[vmsvga2官网](http://sourceforge.net/projects/vmsvga2/) 下载`VMsvga2_v1.2.5_OS_10.9.pkg`(显卡驱动)和`guestd_patches.pkg`(自动调整分辨率补丁)，然后安装，重启，再试试全屏，发现可以了。

##6. 更新软件，关机并压缩打包
点击左上角的苹果图标，选择"Software update"，更新所有，就会从 10.9 更新到 10.9.2。然后关机，用7z将整个虚拟机文件夹压缩成一个压缩包。

可以将这个压缩包共享给别人，也可以作为一个备份，一旦虚拟机装新软件或其它操作弄坏了，可以从这个压缩包解压，以这个镜像为起点，重新开始，而不用从零重新开始。

##参考资料
1. [Working OS X 10.9 Mavericks VMware Image For Windows OS and Intel Processor](http://www.sysprobs.com/working-os-x-10-9-mavericks-vmware-image-for-windows-os-intel-processor)
1. [超详细VMware Workstation 10安装OS X Mavericks](http://www.rshining.net/2013/10/%E8%B6%85%E8%AF%A6%E7%BB%86vmware-workstation-10%E5%AE%89%E8%A3%85os-x-mavericks/)
