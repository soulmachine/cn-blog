---
layout: post
title: "运行mahout的朴素贝叶斯分类器"
date: 2013-12-23 17:20
comments: true
categories: machine-learning
tags: [mahout, naive bayes]
---
##1.准备数据

###1.1 下载数据集，并解压

	wget http://people.csail.mit.edu/jrennie/20Newsgroups/20news-bydate.tar.gz
	tar -xf 20news-bydate.tar.gz
	#上传到hdfs
	hadoop fs -put 20news-bydate-test .
	hadoop fs -put 20news-bydate-train .

###1.2 转换格式

	#转换为序列文件(sequence files)
	mahout seqdirectory -i 20news-bydate-train -o 20news-bydate-train-seq
	mahout seqdirectory -i 20news-bydate-test -o 20news-bydate-test-seq
	#转换为tf-idf向量
	mahout seq2sparse -i 20news-bydate-train-seq -o 20news-bydate-train-vector -lnorm -nv -wt tfidf
	mahout seq2sparse -i 20news-bydate-test-seq -o 20news-bydate-test-vector -lnorm -nv -wt tfidf

##2. 训练朴素贝叶斯模型

	mahout trainnb -i 20news-bydate-train-vectors/tfidf-vectors -el -o model -li labelindex -ow

##3. 测试朴素贝叶斯模型

	mahout testnb -i 20news-bydate-train-vectors/tfidf-vectors -m model -l labelindex -ow -o test-result

##4. 查看训练后的结构

	mahout seqdumper -i labelindex 

    Input Path: labelindex
    Key class: class org.apache.hadoop.io.Text Value Class: class org.apache.hadoop.io.IntWritable
    Key: alt.atheism: Value: 0
    Key: comp.graphics: Value: 1
    Key: comp.os.ms-windows.misc: Value: 2
    Key: comp.sys.ibm.pc.hardware: Value: 3
    Key: comp.sys.mac.hardware: Value: 4
    Key: comp.windows.x: Value: 5
    Key: misc.forsale: Value: 6
    Key: rec.autos: Value: 7
    Key: rec.motorcycles: Value: 8
    Key: rec.sport.baseball: Value: 9
    Key: rec.sport.hockey: Value: 10
    Key: sci.crypt: Value: 11
    Key: sci.electronics: Value: 12
    Key: sci.med: Value: 13
    Key: sci.space: Value: 14
    Key: soc.religion.christian: Value: 15
    Key: talk.politics.guns: Value: 16
    Key: talk.politics.mideast: Value: 17
    Key: talk.politics.misc: Value: 18
    Key: talk.religion.misc: Value: 19
    Count: 20





