---
layout: post
title: "编写Nutch插件"
date: 2014-02-20 16:53
comments: true
categories: Search-Engine
---

软件版本：Nutch 1.7

Nutch Plugin的所有资料，都在官网这里, [PluginCentral](http://wiki.apache.org/nutch/PluginCentral)

##前提
[在Eclipse里运行Nutch](http://www.yanjiuyanjiu.com/blog/20140120)

## Extension 和 Extension-point的关系
Extension point类似与Java语言里的接口(interface), extension 则是具体的实现(implementation)。

[About Plugins](http://wiki.apache.org/nutch/AboutPlugins)里有一句话，**Each extension-point defines an interface that must be implemented by the extension.**

##Nutch里的各种概念
Extension point, extension, plugin, 这些概念是什么意思？见 [Technical Concepts Behind the Nutch Plugin System](http://wiki.apache.org/nutch/WhichTechnicalConceptsAreBehindTheNutchPluginSystem)

##Nutch 1.7 有哪些 Extension-point
![](/images/extension-points.png)

ExtensionPoint 这个东西，本身也是一个插件，可以看看 `src/plugin/nutch-extensionpoints/plugin.xml`，里面定义了所有的扩展点，跟上图基本一致。

[AboutPlugins](http://wiki.apache.org/nutch/AboutPlugins)这里列出来的是 Nuch 1.4的扩展点，有点过时了。

##一个Nutch的组成文件
build.xml, plugins.xml 等等

##Nutch 插件例子

1. [WritingPluginExample](http://wiki.apache.org/nutch/WritingPluginExample)
1. [WritingPluginExample-1.2](http://wiki.apache.org/nutch/WritingPluginExample-1.2)，针对Nutch 1.2的，有点老，但是值得一看
1. [Writing a plugin to add dates by Ryan Pfister](http://www.ryanpfister.com/2009/04/how-to-sort-by-date-with-nutch/)

看来这3个例子，你应该就知道怎么开发插件了。

##Nutch 的缺点
在抓取的过程中，真正的难度在于, ip limit 和 user limit，可惜 Nutch 对这两个问题都没有解决方案。

1. Nutch 的 [HttpPostAuthentication](http://wiki.apache.org/nutch/HttpPostAuthentication) 现在还没有开发完，导致无法抓取需要登录的网站，例如新浪微波，豆瓣等UGC网站，都是需要登录的。没有这个`HttpPostAuthentication`，Nutch其实只能抓取不需要登录的网页，适用范围大打折扣，现在是web 2.0时代，真正优质的内容，几乎都是需要登录的。
1. 多个代理的管理。Nutch 没有提供多个代理的管理功能，只能在`nutch-site.xml`里配置一个代理。比如我在网上抓取了几百个免费的http代理，怎么让Nutch的各个线程均匀的使用这些代理，平能自动判断代理的速度，优先选择速度高的代理呢？