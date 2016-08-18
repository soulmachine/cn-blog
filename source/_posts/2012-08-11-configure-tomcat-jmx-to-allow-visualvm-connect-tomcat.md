---
layout: post
title: "用VisualVM连接 tomcat 服务器时，如何配置tomcat启动JMX"
date: 2012-08-11 21:37
comments: true
categories: Tools
---
用VisualVM连接 tomcat 服务器时，需要让tomcat启动JMX，在catalina.sh 中添加一行代码即可：

``` bash
JAVA_OPTS=”$JAVA_OPTS -Djava.rmi.server.hostname=192.168.0.123 -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=8086 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false
```
注意，用hostname -i 查看是否为127.0.01，这步非常重要,否则会连接失败，如果是，必须要配置-Djava.rmi.server.hostname。

参考：   
[Using VisualVM to fix live Tomcat and JVM problems](http://blog.tty.nl/2010/09/03/using-visualvm-to-fix-live-tomcat-and-jvm-problems/)  
[JVM内存监控:visualVM jconsole jstatd jmap](http://blog.csdn.net/linghunhong/article/details/6438572)
