---
layout: post
title: "Nutch 快速入门(Nutch 1.7)"
date: 2014-01-21 04:11
comments: true
categories: Search-Engine
---
本文主要参考[Nutch Tutorial](http://wiki.apache.org/nutch/NutchTutorial)

Nutch 2.2.1目前性能没有Nutch 1.7好，参考这里，[NUTCH FIGHT! 1.7 vs 2.2.1](http://digitalpebble.blogspot.com/2013/09/nutch-fight-17-vs-221.html). 所以我目前还是使用的Nutch 1.7。

##1 下载已编译好的二进制包，解压
    $ wget http://psg.mtu.edu/pub/apache/nutch/1.7/apache-nutch-1.7-bin.tar.gz
    $ tar zxf apache-nutch-1.7-bin.tar.gz

##2 验证一下

    $ cd apache-nutch-1.7
    $ bin/nutch

如果出现"Permission denied"请运行下面的命令：

    $ chmod +x bin/nutch

如果有Warning说 `JAVA_HOME`没有设置，请设置一下`JAVA_HOME`.

##3 添加种子URL

    mkdir ~/urls
    vim ～/urls/seed.txt
    http://movie.douban.com/subject/5323968/

##4 设置URL过滤规则
如果只想抓取某种类型的URL，可以在 `conf/regex-urlfilter.txt`设置正则表达式，于是，只有匹配这些正则表达式的URL才会被抓取。

<!--more-->

例如，我只想抓取豆瓣电影的数据，可以这样设置：
    
    #注释掉这一行
    # skip URLs containing certain characters as probable queries, etc.
    #-[?*!@=]
    # accept anything else
    #注释掉这行
    #+.
    +^http:\/\/movie\.douban\.com\/subject\/[0-9]+\/(\?.+)?$

##5 设置agent名字

conf/nutch-site.xml:

    <property>
      <name>http.agent.name</name>
      <value>My Nutch Spider</value>
    </property>

这一步是从这本书上看到的，[Web Crawling and Data Mining with Apache Nutch](http://www.packtpub.com/web-crawling-and-data-mining-with-apache-nutch/book)，第14页。

##6 安装Solr
由于建索引的时候需要使用Solr，因此我们需要安装并启动一个Solr服务器。

参考[Nutch Tutorial](http://wiki.apache.org/nutch/NutchTutorial) 第4、5、6步，以及[Solr Tutorial](http://lucene.apache.org/solr/4_6_1/tutorial.html)。

###6.1 下载，解压

wget http://mirrors.cnnic.cn/apache/lucene/solr/4.6.1/solr-4.6.1.tgz
tar -zxf solr-4.6.1.tgz

###6.2 运行Solr

    cd example
    java -jar start.jar

验证是否启动成功

用浏览器打开 <http://localhost:8983/solr/admin/>，如果能看到页面，说明启动成功。

###6.3 将Nutch与Solr集成在一起

将`NUTCH_DIR/conf/schema-solr4.xml`拷贝到`SOLR_DIR/solr/collection1/conf/`，重命名为schema.xml，并在`<fields>...</fields>`最后添加一行(具体解释见[Solr 4.2 - what is _version_field?](http://stackoverflow.com/questions/15527380/solr-4-2-what-is-version-field))，

    <field name="_version_" type="long" indexed="true" stored="true" multiValued="false"/>

重启Solr，

    # Ctrl+C to stop Solr
    java -jar start.jar

##7 一步一步使用单个命令抓取网页

###7.1 基本概念
Nutch data is composed of:

- The crawl database, or `crawldb`. This contains information about every URL known to Nutch, including whether it was fetched, and, if so, when.
- The link database, or `linkdb`. This contains the list of known links to each URL, including both the source URL and anchor text of the link.
- A set of `segments`. Each segment is a set of URLs that are fetched as a unit. Segments are directories with the following subdirectories:
  + a `crawl_generate` names a set of URLs to be fetched
  + a `crawl_fetch` contains the status of fetching each URL
  + a `content` contains the raw content retrieved from each URL
  + a `parse_text` contains the parsed text of each URL
  + a `parse_data` contains outlinks and metadata parsed from each URL
  + a `crawl_parse` contains the outlink URLs, used to update the crawldb

###7.2 inject:使用种子URL列表，生成crawldb

    $ bin/nutch inject TestCrawl/crawldb ~/urls

###7.3 generate

    $ bin/nutch generate TestCrawl/crawldb TestCrawl/segments

这会生成一个fetch list，存放在一个segments目录下，该segment以被创建时间为名字，对应着一个同名目录。我们将这个segment的名字保存在shell变量`s1`里：

    $ s1=`ls -d TestCrawl/segments/2* | tail -1`
    $ echo $s1

###7.4 fetch

    $ bin/nutch fetch $s1

###7.5 parse

    $ bin/nutch parse $s1

###7.6 updatedb

    $ bin/nutch updatedb TestCrawl/crawldb $s1

###7.7 invertlinks

    $ bin/nutch invertlinks TestCrawl/linkdb -dir TestCrawl/segments

###7.8 查看结果

    $ bin/nutch readdb TestCrawl/crawldb/ -stats

###7.9 solrindex, 提交数据给solr，建立索引

    $ bin/nutch solrindex http://localhost:8983/solr TestCrawl/crawldb/ -linkdb TestCrawl/linkdb/ TestCrawl/segments/20140203004348/ -filter -normalize

###7.10 solrdedup, 给索引去重
有时重复添加了数据，导致索引里有重复数据，我们需要去重，

    $bin/nutch solrdedup http://localhost:8983/solr

###7.11 solrclean, 删除索引
如果数据过时了，需要在索引里删除，也是可以的。

    $ bin/nutch solrclean TestCrawl/crawldb/ http://localhost:8983/solr

##8 使用crawl脚本一键抓取
刚才我们是手工敲入多个命令，一个一个步骤，来完成抓取的，其实Nutch自带了一个脚本，`./bin/crawl`，把抓取的各个步骤合并成一个命令，看一下它的用法

    $ bin/crawl 
    Missing seedDir : crawl <seedDir> <crawlDir> <solrURL> <numberOfRounds>

注意，是使用`bin/crawl`，不是`bin/nutch crawl`，后者已经是deprecated的了。

先删除第7节产生的数据，

    $ rm -rf TestCrawl/

###8.1 抓取网页

    $ ./bin/crawl ~/urls/ ./TestCrawl http://localhost:8983/solr/ 2

* `～/urls` 是存放了种子url的目录
* TestCrawl 是存放数据的根目录（在Nutch 2.x中，则表示crawlId，这会在HBase中创建一张以crawlId为前缀的表，例如TestCrawl_Webpage）
* http://localhost:8983/solr/ , 这是Solr服务器
* 2，numberOfRounds，迭代的次数

过了一会儿，屏幕上出现了一大堆url，可以看到爬虫正在抓取！

    fetching http://music.douban.com/subject/25811077/ (queue crawl delay=5000ms)
    fetching http://read.douban.com/ebook/1919781 (queue crawl delay=5000ms)
    fetching http://www.douban.com/online/11670861/ (queue crawl delay=5000ms)
    fetching http://book.douban.com/tag/绘本 (queue crawl delay=5000ms)
    fetching http://movie.douban.com/tag/科幻 (queue crawl delay=5000ms)
    49/50 spinwaiting/active, 56 pages, 0 errors, 0.9 1 pages/s, 332 245 kb/s, 131 URLs in 5 queues
    fetching http://music.douban.com/subject/25762454/ (queue crawl delay=5000ms)
    fetching http://read.douban.com/reader/ebook/1951242/ (queue crawl delay=5000ms)
    fetching http://www.douban.com/mobile/read-notes (queue crawl delay=5000ms)
    fetching http://book.douban.com/tag/诗歌 (queue crawl delay=5000ms)
    50/50 spinwaiting/active, 61 pages, 0 errors, 0.9 1 pages/s, 334 366 kb/s, 127 URLs in 5 queues

###8.2 查看结果

    $ bin/nutch readdb TestCrawl/crawldb/ -stats
    14/02/14 16:35:47 INFO crawl.CrawlDbReader: Statistics for CrawlDb: TestCrawl/crawldb/
    14/02/14 16:35:47 INFO crawl.CrawlDbReader: TOTAL urls:	70
    14/02/14 16:35:47 INFO crawl.CrawlDbReader: retry 0:	70
    14/02/14 16:35:47 INFO crawl.CrawlDbReader: min score:	0.005
    14/02/14 16:35:47 INFO crawl.CrawlDbReader: avg score:	0.03877143
    14/02/14 16:35:47 INFO crawl.CrawlDbReader: max score:	1.23
    14/02/14 16:35:47 INFO crawl.CrawlDbReader: status 1 (db_unfetched):	59
    14/02/14 16:35:47 INFO crawl.CrawlDbReader: status 2 (db_fetched):	11
    14/02/14 16:35:47 INFO crawl.CrawlDbReader: CrawlDb statistics: done


##相关文章
[Nutch 快速入门(Nutch 2.2.1)](http://www.yanjiuyanjiu.com/blog/20140201/)

