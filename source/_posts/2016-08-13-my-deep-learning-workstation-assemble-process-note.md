---
title: 我的深度学习工作站攒机过程记录
date: 2016-08-13 09:26:11
tags: 深度学习
categories: 深度学习
---

一年前就有攒一台深度学习工作站的想法了，今年 6月29号在Amazon 上抢到了一块 GTX 1080后，正式开始了攒机，经过了漫长了的一个半月，到今天8约13号，终于凑齐了所有零件并开机点亮了。为甚么这么漫长？因为本屌为了省钱几乎所有零件都是在 eBay和Amazon 上买的二手（除了GTX 1080太新没二手货）。



## 硬件配置单


### 结论

懒人不想看过程的话，可以先看结论。

以下是我的配置单：

|    配件名   | 品牌型号   | 数量 | 价格 | 哪里买的 |
|:------------:|:-----------:|:-----------:|:-----------:|:-----------:|
| 机箱 | Corsair Carbide Air 540 | 1 | $130.57 | Amazon 二手 |
|    主板    | Asus X99-E WS/USB3.1 | 1 | $563.84 | Amazon 新 |
|    CPU    | Intel I7-5930K | 1 | $467.45 | eBay 二手 |
|    CPU 水冷头    | Corsair H60 Cooler | 1 | $65.24 | Amazon 新 |
| DDR4内存 | Kingston 32GB HX421C14FB2K4/32 | 4 | $139.19 | Amazon新 |
| 显卡 | Zotac GTX 1080 Founders Edition | 1 | $761.24 | Amazon 新 |
| 电源 | EVGA 1600W 80+ Gold 120-G2-1600-X1 | 1 | $241.74 | eBay二手 |
| SSD | Samsung SM951 256GB M.2 NVMe MZVPV256HDGL | 1 | $129.5 | eBay二手 |
| 机械硬盘 | WD Green 4TB | 1 | $162.36 | Amazon新 |

总计：$2661.13

以上价格已经包含了税。

这个配置单基本齐全了，唯一还没有到位的是GPU显卡了，目前只有一块 GTX 1080。我打算等一年后 GTX 1080 有二手货了，再收3块二手的，这样就全了。

接下来一项一项详细说明我为什么这么选择，展示我的决策过程。

<!-- more -->


### 机箱

直接选跟 Nvidia DevBox 同款的机箱，即 Corsair Carbide Air 540


### 主板

硬性要求：

1. X99平台
1. 四个 PCI-E Gen3 x16 接口

众所周知单机多卡进行训练时，总线带宽是瓶颈，所以主板有越多的PCI-e 3.0 接口越好，可以尽可能把显卡的性能发挥出来。计划上四块 GTX 1080，那么至少需要四路 PCI-e 3.0 的 x16 接口。

Nvidia DevBox 用的是 Asus X99-E WS 工作站主板，那我也在X99 主板里选了很久，

这三块板子都有 4路 PCI-e 3.0 x16 接口，

* Asus X99-E WS
* Asus X99-E WS/USB 3.1
* 技嘉 GA-X99P-SLI

当时我还请教了一下 [@硬哥](http://weibo.com/1775948951/DESoMxdRf) ，问上四块 GTX 1080 显卡的话，用哪个主板好，

![](/images/weibo-which-motherboard.png)

华硕 X99-E WS/USB 3.1 是 Asus X99-E WS 的新版，本着买新不买旧的原则，那就买  Asus  X99-E WS/USB 3.1 吧。


### CPU

众所周知单机多卡进行训练时，总线带宽是瓶颈，所以 CPU 的 PCI-e  lane 越多越好，一般消费级的CPU，PCI-e总线根数是 16, 28 或 40，最大就是40，再大就需要上服务器CPU或者双路CPU了。选40， 这个条件下，有两款 CPU 入围，I7-5930K和 I7-5960X, 5860X太贵了，一般机器学习训练中CPU不是瓶颈，所以选 5930K就可以了，这也是 Nvidia 官方推出的 DevBox 工作站所使用的CPU。

在知乎上看到不少人用 I7-5820K，这个CPU虽然很不错，可是只有 28条 PCI-e总线，所以还是加几百块钱上 5930k比较好。


### CPU水冷头

其实普通的风扇就可以了，不过水冷更加安静些，所以我选择了一个便宜的 Corsair H60 Cooler ，够用了。


### 内存

DDR4 64G

再大就没必要了，李沐的这篇博客[GPU集群折腾手记](http://mli.github.io/gpu/2016/01/17/build-gpu-clusters/)  末尾有说到过，单机4卡的机器，64G内存绰绰有余了。

况且消费级的I7 CPU，最大支持内存就是 64G。


### 显卡

这个最好选，GTX 1080

最新出来的 GTX 1080 吊打 Maxwell 架构的 Titan , 价钱又便宜很多，所以显卡最好选。


### 电源

一般 Nvidia 的旗舰显卡，功耗都是压着 300W的线的，四卡就 1200W了，加上主板，CPU等耗电，选一个 1600W的电源吧。

Nvidia DevBox 用的是 EVGA 1600W 80+ Gold 120-G2-1600-X1 ，那我也用它吧。


### SSD

起码需要一块SSD作为系统盘。

当前 SATA III 接口的SSD最普遍也最便宜，不过由于 Asus X99-E WS/USB 3.1 恰好有一个 M.2 接口，我决定买一个 M.2 NVMe 的SSD，不用就浪费了主板的这个接口了。在 eBay 上买了一块二手的 Samsung SM951 256GB M.2 NVMe MZVPV256HDGL 。


### 机械硬盘

随便选，我选择最便宜的绿盘。


### 一些波折

等了一个月终于凑齐了所有配件后，兴奋的开始装机，装完了后发现死活点不亮，<http://weibo.com/1663402687/E1vrO4VFn>

![](/images/weibo-cannot-boot.png)

尼玛，浪费了我2天时间在各种诊断，甚至一度怀疑自己的装机能力一直到怀疑人生。。。

只好把主板和CPU退货了，又在 eBay上竞拍了一个二手 I7-5930K，这是 eBay 上的 Asus X99-E WS/USB 3.1 竟然没有二手货了，Amazon 也没有二手货，只有一个 refurbishment , 不敢买，只要在 Amazon 买了个全新的主板。

等了一周多，今天终于齐了，可以开机进入操作系统了。不过有个小问题，按机箱上的开机键，没有反应，必须打开机箱按主板上的那个开机键。当时我内心是崩溃的，不可能每次开机需要打开机箱吧，那多烦啊。于是开始诊断，先用一根金属棒让 POWSER_SW 的两根针脚短路，可以开机，说明主板的POWSER_SW的两根针脚是好的，那么唯一的原因，就是机箱上的开机键不灵了，我打开机箱，把开机键后面焊接的细线按了几下，再开机，机箱上的开机键起作用了。可能是接触不良吧，我也没有深究了，盖上了机箱盖。


## 一些配置和 tunning

打开了TPU，EPU,DR.POWER , EZ_XMP四个开关，TPU推到了中间的 TPU I， 没敢到最右边的 TPU II

GTX 1080 要插到 PCIE 1,3,5,7的位置上，这几个插槽是X16模式的，其他是x8模式的

后续：

[安装 Windows 10 和 Ubuntu 16.04 双系统](http://cn.soulmachine.me/2016-08-14-dual-install-windows-ubuntu/)
[深度学习开发环境配置：Ubuntu1 6.04+Nvidia GTX 1080+CUDA 8.0](http://cn.soulmachine.me/2016-08-17-deep-learning-cuda-development-environment/)


## 参考资料

1. [Nvidia DevBox](https://developer.nvidia.com/devbox)
1. [32-TFLOP Deep Learning GPU Box - hackaday](https://hackaday.io/project/12070-32-tflop-deep-learning-gpu-box)
1. [如何配置一台适用于深度学习的工作站？- 知乎](https://www.zhihu.com/question/33996159)
1. [Exxact Deep Learning Workstation](http://exxactcorp.com/deep-learning-workstations-servers.php)
1. [GPU集群折腾手记 - 李沐](http://mli.github.io/gpu/2016/01/17/build-gpu-clusters/)
