---
layout: post
title: "LaTeX的各种发行版和编辑器的比较"
date: 2013-04-12 11:01
comments: true
categories: Tools
---
##发行版(distribution)
TeX类似于Linux，有很多不同的发行版(distribution)。

先看看各个发行版的流行程度。
 
|----------+-------------------------------+--------+-----------+------------------+-----------+-----------|  
| **名字** |            **官网**           | **PR** | **Alexa** |   **最后更新**   | **weibo** | **quora** |  
|:--------:|:-----------------------------:|:------:|:---------:|:----------------:|:---------:|:--------:|  
| TeX Live |     <http://www.tug.org/texlive/>    |    7   |    N/A    |    2012-07-01    |     43    |    N/A    |  
|  MiKTeX  |     <http://miktex.org/>      |    7   |  188,485  | 1.3.2 2012-09-24 |     54    |    N/A    |  
|   CTeX   |     <http://www.ctex.org/>    |    6   |  252,657  | 2.9.2 2012-03-30 |     344   |    N/A    |  
|  proTeXt | <http://www.tug.org/protext/> |    7   |    N/A    | 3.1.1 2012-07-23 |     10    |    N/A    |  
|----------+-------------------------------+--------+-----------+------------------+-----------+-----------|   

其中CTeX和proTeXt都是基于MiKTeX的，再次进行了打包。国内估计用CTeX比较多。

##编辑器(editor)
编辑器大概分为两种，一种为WYSIWYG，所见即所得，实时预览，类似于Word，另一种是纯文本编辑器，有语法高亮，没有预览功能，需要另外安装一个发行版，编译成PDF后才能预览。

先看看各个编辑器的流行程度。

|--------------+-------------------------------------+--------+-----------+--------------+--------------------------+-----------+-----------|  
|   **名字**   |                 **官网**            | **PR** | **Alexa** | **预览类型** |      **最后更新**        | **weibo** | **quora** |  
|:------------:|:-----------------------------------:|:------:|:---------:|:------------:|:------------------------:|:---------:|:---------:|  
|   TeXmaker   | <http://www.xm1math.net/texmaker/>  |    6   |  289,311  |    无预览    |      4.0.1 2013-03-16    |     60    |    N/A    |  
|   TeXworks   |   <http://www.tug.org/texworks/>    |    5   |  90,230   |    无预览    |      0.4.4  2012-04      |     23    |    N/A    |  
|   TeXstudio  | <http://texstudio.sourceforge.net/> |    6   |    N/A    |    无预览    |      2.5.2 2013-01-08    |     15    |    N/A    |  
| TeXnicCenter |   <http://www.texniccenter.org/>    |    7   |  884,570  |    无预览    |  v2.0 beta1 2012-11-03   |     10    |    N/A    |  
|     Lyx      |       <http://www.lyx.org/>         |    6   |  261,649  |   实时预览   |    2.0.5.1 2013-01-08    |     56    |     42    |  
|    Bakoma    |   <http://www.bakoma-tex.com/>      |    5   | 1,327,901 |   实时预览   |     10.10 2013-01-13     |      3    |    N/A    |  
|   TeXmacs    |     <http://www.texmacs.org>        |    6   | 1,525,373 |   实时预览   |   1.0.7.19 2013-03-27    |     27    |    N/A    |  
|     LEd      |   <http://www.latexeditor.org/>     |    6   |  624,564  |   实时预览   |     0.53 2009-10-09      |     0    |    N/A    |  
|--------------+-------------------------------------+--------+-----------+--------------+--------------------------+-----------+-----------|  

<!--more-->

##跨平台
下面看看各个发行版和编辑器的跨平台支持程度。

|--------------+---------+-----+-------|  
|   **名字**   | Windows | Mac | Linux |  
|:------------:|:-------:|:---:|:-----:|  
|    编辑器     |         |     |       |  
|   TeXmaker   |    √    |  √  |   √   |  
|   TeXworks   |    √    |  √  |   √   |  
|   TeXstudio  |    √    |  √  |   √   |   
| TeXnicCenter |    √    |  ×  |   ×   |  
|   WYSIWYG    |         |     |       |  
|     Lyx      |    √    |  √  |   √   |  
|    Bakoma    |    √    |  √  |   √   |  
|   TeXmacs    |    √    |  √  |   √   |  
|     LEd      |    √    |  ×  |   ×   |  
|     发行版    |         |     |       |  
|     MiKTeX   |     √   |  ×  |   ×   |  
|    TeX Live  |    √    |  √  |   √   |  
|     CTeX     |     √   |  ×  |   ×   |  
|     proTeXt  |     √   |  ×  |   ×   |  
|--------------+---------+-----+-------| 

Tex Live在Mac上，叫做MacTex，见[官网的一段话](http://www.tug.org/mactex/newfeatures.html)：

> MacTeX-2012 installs a completely unmodified copy of the full TeX Live 2012 distribution. This is exactly the same distribution that runs on OS X, Windows, GNU/Linux, various BSD Unix systems, and other systems.

##如何选择
四个发新版，只有 Tex Live 是跨平台的，故使用Tex Live，其他发行版抛弃。

TeXmaker, TeXstudio, TeXworks 来进行比较  
中文支持的程度，打开.tex文件是否有乱码

|-----------+--------------------------+------------------------|  
|  **名字** | **打开GB18030的tex文件** | **打开UTF8编码的文件** |  
|:---------:|:------------------------:|:----------------------:|  
| TeXworks  |         有乱码           |          无乱码        |  
| TeXmaker  |         无乱码           |          无乱码        |  
| TeXstudio |         有乱码           |          无乱码        |  
|-----------+--------------------------+------------------------|  

TeXmaker 界面丑陋，且中文支持不好，功能没有多，抛弃之。
TeXmaker 和 TeXstudio 界面比较美观，而且二者界面风格很类似。因为TeXstudio是在TeXmaker的基础上而来的，[见wikipedia的描述](http://en.wikipedia.org/wiki/TeXstudio)：

> Originally called TexMakerX, TeXstudio was started as a fork of Texmaker that tried to extend it with additional features while keeping its look and feel.

TeXnicCenter 安装时不会自动探测，第一次运行时会要求你指定 latex.exe 的路径。TeXnicCenter 界面风格是office的风格，很现代化。TeXnicCenter 只有 windows版，故放弃。

LyX 安装时会自动探测到TeX Live。这点比较方便，无需配置。  
LyX可以导入.tex文件，导入后，不能直接修改.tex源码，只能在上方的可视化区域直接输入内容，即LyX强迫你用类似word的方式来输入内容。因此抛弃LyX。

Bakoma 是商业软件，30天试用期，网上搜了一下，没有破解版，故放弃。

TeXmacs 1.0.7.19 在windows上安装完成后，双击后启动界面会闪退，完全没法用，换了1.0.7.18，可以启动了，目前发现两个问题：1. 打开（使用文件-->打开或导入）一个含有中文的.tex文件会崩溃；2. 关闭程序管不了，需要用任务管理器杀掉才行，可见TeXmacs 还很不完善，其次TeXmacs 有着自己的语法，不是一个标准的TeX发行版，因此放弃 TeXmacs 。

LEd已经很久不更新，且只有windows版，放弃。

大牛陈硕用的是Tex Live，他的书使用Tex Live来排版的。  
![](http://yanjiuyanjiu-wordpress.stor.sinaapp.com/uploads/2013/04/chenshuo_texlive.png)

##安装和配置
在windows下安装 Tex Live 2012，先下载DVD ISO，然后安装即可。假设安装到`D:\texlive`。

安装完后，将`D:\texlive\2012\bin\win32`添加到PATH环境变量。这样Texmaker，Texstudio就不用配置了，安装后即可正常编译。如果没有添加到PATH环境变量，则在Texmaker，Texstudio中指定一些exe文件的绝对路径。

