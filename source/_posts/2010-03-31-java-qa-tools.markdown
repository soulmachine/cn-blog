---
layout: post
title: "Java质量检测评估工具"
date: 2010-03-31 16:45
comments: true
categories: tools
---
Java代码质量检测评估工具
“五大” 代码分析领域：

* 编码风格
* 冗余代码
* 代码覆盖率
* 依赖项分析
* 复杂度监控

一下列举了一些目前比较流行的工具。网址后面列出了其PR值，可以反映此工具的流行度。

1、编码风格
CheckStyle
Home page: [http://checkstyle.sourceforge.net/](http://checkstyle.sourceforge.net/) (6)  
对应的eclipse插件有多个，其中eclipsecs最常用  
Home page: [http://eclipse-cs.sourceforge.net/](http://eclipse-cs.sourceforge.net/) (6)  
eclipse插件URL：[http://eclipse-cs.sf.net/update/](http://eclipse-cs.sf.net/update/)

2、冗余代码
Simian [http://www.redhillconsulting.com.au/products/simian/](http://www.redhillconsulting.com.au/products/simian/) (5)  
PMD 的 CPD [http://pmd.sourceforge.net/cpd.html](http://pmd.sourceforge.net/cpd.html) (5)
<!--more-->
3、代码覆盖率
EMMA  [http://emma.sourceforge.net/](http://emma.sourceforge.net/) (6)  
Cobertura  [http://cobertura.sourceforge.net/](http://cobertura.sourceforge.net/) (6)  
EclEmma [http://update.eclemma.org/](http://update.eclemma.org/) (5)  
Coverlipse [http://coverlipse.sourceforge.net/](http://coverlipse.sourceforge.net/) (3)

4、依赖项分析
JDepend [http://clarkware.com/software/JDepend.html](http://clarkware.com/software/JDepend.html) (6)

5、复杂度监控
Metrics [http://metrics.sourceforge.net/](http://metrics.sourceforge.net/) (4)

具有以上两项或两项以上的综合工具（也称为静态分析工具）：
1、PMD
Home page: [http://pmd.sourceforge.net/](http://pmd.sourceforge.net/) (6)  
eclipse插件URL：http://pmd.sourceforge.net/eclipse[http://pmd.sourceforge.net/](http://pmd.sourceforge.net/)

2、FindBugs [http://findbugs.sourceforge.net/](http://findbugs.sourceforge.net/) (6)  
eclipse插件URL：[http://findbugs.cs.umd.edu/eclipse](http://findbugs.cs.umd.edu/eclipse)

FindBugs 检查程序生成的class文件，即分析字节码
PMD 检查源码，分析源代码

3、Jtest [http://www.parasoft.com/jtest](http://www.parasoft.com/jtest)

4、Jlint [http://artho.com/jlint/](http://artho.com/jlint/) (5)

5、Lint4j [http://www.jutils.com/](http://www.jutils.com/) (4)

我个人平时最常用的是Checkstyle，其次是PMD，大家可以参考一下。

##参考资料
<http://blog.csdn.net/cb_121/archive/2009/05/22/4208792.aspx>
