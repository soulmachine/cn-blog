---
layout: post
title: "java命令行下运行class文件"
date: 2010-02-15 16:27
comments: true
categories: Language
---
今天碰到了一个很变态的问题，写了一个很简单的HelloWord.java，内容如下：

``` java
package com.yanjiuyanjiu;

public class HelloWorld {
public static void main(String args[]) {
System.out.println(“Hello World!”);
}
}
```
在eclipse中运行是可以的，但是在命令行下运行总是失败。我的工程位置为 d:workspaceHelloWorld。

尝试了很多次，如下：

<!--more-->

``` bash
d:workspaceHelloWorldbincomyanjiuyanjiu>java HelloWorld

Exception in thread “main” java.lang.NoClassDefFoundError: HelloWorld (wrong name: com/yanjiuyanjiu/HelloWorld)
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClassCond(Unknown Source)
at java.lang.ClassLoader.defineClass(Unknown Source)
at java.security.SecureClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.access$000(Unknown Source)
at java.net.URLClassLoader$1.run(Unknown Source)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
Could not find the main class: HelloWorld.  Program will exit.
Exception in thread “main”

d:workspaceHelloWorldbincomyanjiuyanjiu>java -classpath .; HelloWorld
Exception in thread “main” java.lang.NoClassDefFoundError: HelloWorld (wrong name: com/yanjiuyanjiu/HelloWorld)
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClassCond(Unknown Source)
at java.lang.ClassLoader.defineClass(Unknown Source)
at java.security.SecureClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.access$000(Unknown Source)
at java.net.URLClassLoader$1.run(Unknown Source)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
Could not find the main class: HelloWorld.  Program will exit.
Exception in thread “main”

d:workspaceHelloWorldbincomyanjiuyanjiu>cd..

d:workspaceHelloWorldbincom>cd..

d:workspaceHelloWorldbin>java -classpath .; com/yanjiuyanjiu/HelloWorld    只有这个成功

Hello World!

d:workspaceHelloWorldbin>java -classpath .; comyanjiuyanjiuHelloWorld    换了个斜杠就不行了

Exception in thread “main” java.lang.NoClassDefFoundError: comyanjiuyanjiuHelloWorld (wrong name: com/yanjiuyanjiu/HelloWorld)
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClassCond(Unknown Source)
at java.lang.ClassLoader.defineClass(Unknown Source)
at java.security.SecureClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.access$000(Unknown Source)
at java.net.URLClassLoader$1.run(Unknown Source)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
Could not find the main class: comyanjiuyanjiuHelloWorld.  Program will exit.
Exception in thread “main”

d:workspaceHelloWorldbin>java -classpath ./com/yanjiuyanjiu/; HelloWorld

Exception in thread “main”java.lang.NoClassDefFoundError: HelloWorld (wrong name: com/yanjiuyanjiu/HelloWorld)
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClassCond(Unknown Source)
at java.lang.ClassLoader.defineClass(Unknown Source)
at java.security.SecureClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.access$000(Unknown Source)
at java.net.URLClassLoader$1.run(Unknown Source)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
Could not find the main class: HelloWorld.  Program will exit.
Exception in thread “main”
```

在网上搜索了大半天，大部分说是环境 变量，classpath或JDK 版本的问题，还有执行时文件名 class后缀不要。我一一试过，都没有解决。最后无意中换了一下命令符的位置，成功了。总结如下：

1. 环境变量，CLASSPATH当然要设置好，执行时不要带class后缀；  
1. 路径中的斜杠用“/”而不是“”；  
1. 命令符的当前目录要在包的起点。比如这里应该在 d:workspaceHelloWorldbin>，如果在 d:workspaceHelloWorldbincomyanjiuyanjiu>，反而不行，有点“近水楼台不得月”的意思，不知 道为什么，还请高手解释一下。
