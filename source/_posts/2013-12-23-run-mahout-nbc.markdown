---
layout: post
title: "运行mahout的朴素贝叶斯分类器"
date: 2013-12-23 17:20
comments: true
categories: mahout, machine-learning
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





