---
layout: post
title: "在Eclipse里运行Nutch"
date: 2014-01-20 04:11
comments: true
categories: Search-Engine
---

环境：Ubuntu Desktop 12.04，JDK 1.7, Nutch 1.7

本文主要参考[Running Nutch in Eclipse](http://wiki.apache.org/nutch/RunNutchInEclipse)

##前提

* 机器上安装了Ant, Eclipse
* Eclipse安装了subclipse, update site 是 <http://subclipse.tigris.org/update_1.10.x>
* Eclipse安装了IvyDE, update site 是 <http://www.apache.org/dist/ant/ivyde/updatesite>
* Eclipse安装了m2e插件，update site 是 <http://download.eclipse.org/technology/m2e/releases>

##1 下载源码
有两种方法，

1. 官网下载apache-nutch-1.7-src.tar.gz
1. 用svn checkout

推荐用第2种方法，因为用SVN checkout出来的有`pom.xml`文件，即maven文件，但是压缩包里没有，只有ant的`build.xml`文件。

    $ svn co https://svn.apache.org/repos/asf/nutch/tags/release-1.7

##2 配置
把 conf/ 下的 `nutch-site.xml.template`复制一份，命名为`nutch-site.xml`，在里面添加如下配置：

<property>
  <name>http.agent.name</name>
  <value>My Nutch Spider</value>
</property>
 <property>
   <name>plugin.folders</name>
   <value>$NUTCH_DIR/build/plugins</value>
 </property>

`$NUTCH_DIR`是指nutch源码的根目录，例如我的是`/home/soulmachine/local/src/release-1.7`.

##3 生成Eclipse项目文件，即.project文件

    $ ant eclipse

<!--more-->

如果发现一直卡在 `ivy:resolve .....`这里，原因很有可能是 <http://repo1.maven.org>被墙，参考[这里](http://blog.csdn.net/majian_1987/article/details/17004531)，这时候要设置HTTP代理，

    $ Ctrl+C
    $ export HTTP_PROXY=localhost:8087    #我用的goagent
    $ ant runtime

##4 装载到Eclipse
启动Eclipse，点击"File->Import->General->Existing projects into workspace"，浏览到`$NUTCH_DIR`，点击"Finish"。

![](http://wiki.apache.org/nutch/RunNutchInEclipse?action=AttachFile&do=get&target=importproject.png)

请等待一会儿，让Eclipse解析该项目，可以在右下角看到状态。

##5 把conf/目录加入到class folder
在Package Explorer里右击项目，选择"Build Path->Configure Build Path->Order and Export"，下拉滚动条，找到conf/目录，打上勾，点击"Top"按钮。

![](http://wiki.apache.org/nutch/RunNutchInEclipse?action=AttachFile&do=get&target=order_and_export.png)

##6 在Eclipse里运行org.apache.nutch.crawl.Crawl来抓取网页
在Package Explorer里找到`org.apache.nutch.crawl.Crawl`，右击，选择"Run as -> Java Application"，发现输出如下：

    Usage: Crawl <urlDir> -solr <solrURL> [-dir d] [-threads n] [-depth i] [-topN N]

因为我们没有给main()函数提供必要的参数。

###6.1 添加种子URL

    $ cd PROJECT_DIR
    $ mkdir urls
    $ vim ./urls/seed.txt
    http://movie.douban.com/subject/5323968/

###6.2 设置URL过滤规则
如果只想抓取某种类型的URL，可以在 `conf/regex-urlfilter.txt`设置正则表达式，于是，只有匹配这些正则表达式的URL才会被抓取。

例如，我只想抓取豆瓣电影的数据，可以这样设置：
    
    # accept anything else
    #注释掉这行
    #+.
    +^http://movie.douban.com/subject/[0-9]*/$

**注意**，每次修改了conf目录中的配置文件，必须重新编译，修改才能生效，原因是ant里有拷贝文件的任务，将conf/下的xml文件拷贝到`runtie/local/conf`和`runtime/deploy/conf`下。

这里，我们不再使用命令行编译，而是在eclipse里右击build.xml，选择`Run as -> Ant Build`.

###6.3 安装Solr
见我的这篇文章，[Nutch 快速入门(Nutch 1.7)](http://www.yanjiuyanjiu.com/blog/20140121/) 第6节。

###6.4 给main()函数提供参数
右击`org.apache.nutch.crawl.Crawl`，选择"Run as -> Run Configurations"，在`Program arguments`里填写：

    urls -solr http://localhost:8983/solr/ -dir TestCrawl -depth 2 -topN 5

在"VM Arguments"里填写`-Dhadoop.log.dir=logs -Dhadoop.log.file=hadoop.log`

###6.5 运行
最后点击"Apply", "Run"，就开始抓取了。

###6.6 查看结果

####6.6.1 查看segments目录
右击`org.apache.nutch.segment.SegmentReader`，选择"Run as -> Run Configurations"，在`Program arguments`里填写：

    -dump TestCrawl/segments/* TestCrawl/segments/dump

然后点击"Run"。

用文本编辑器打开文件 `TestCrawl/segments/dump/dump`查看segments中存储的信息。

####6.6.2 查看crawldb目录
右击`org.apache.nutch.crawl.CrawlDbReader`，选择"Run as -> Run Configurations"，在`Program arguments`里填写：

    TestCrawl/crawldb -stats

然后点击"Run"，控制台会输出 crawldb统计信息。

####6.6.3 查看linkdb目录
右击`org.apache.nutch.crawl.LinkDbReader`，选择"Run as -> Run Configurations"，在`Program arguments`里填写：

    TestCrawl/linkdb -dump TestCrawl/linkdb_dump

然后点击"Run"。

用文本编辑器打开文件 `TestCrawl/linkdb_dump/part-00000`查看linkdb中存储的信息


###6.7 每个nutch命令对应的java类
怎么知道每个nutch命令对应的java类呢？打开`src/bin/nutch`并滚动到底部，就会找到每个命令对应的java类。

##7 在Eclipse里单步调试
以上费了这么大劲，为的是什么？就是为了利用Eclipse这个强大的IDE，能够进行单步调试。写代码其实不花时间，调试才是最花时间的，因此一定要有好的调试工具，磨刀不误砍柴功。

如果想调试某一个类的代码，在Eclipse里下断点，然后"Debug as ..."就可以了。

由于有Hadoop job的关系，比较难以难以在哪些地方下断点好。下面给出了Nutch 1.x 的代码中，一些比较好的断点位置：

* Fetcher [line: 1115] - run
* Fetcher [line: 530] - fetch
* Fetcher$FetcherThread [line: 560] - run()
* Generator [line: 443] - generate
* Generator$Selector [line: 108] - map
* OutlinkExtractor [line: 71 & 74] - getOutlinks


##相关文章
[Nutch 快速入门(Nutch 1.7)](http://www.yanjiuyanjiu.com/blog/20140121/)

[Nutch 快速入门(Nutch 2.2.1)](http://www.yanjiuyanjiu.com/blog/20140201/)

