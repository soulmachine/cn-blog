---
layout: post
title: "Mhout KMeans 结果分析"
date: 2013-07-07 17:04
comments: true
categories: mahout
published: false
---

# 运行KMeans示例
运行KMeans示例有三种方法，参考前篇一篇博客，[安装mahout]()。

通过阅读`examples\src\main\java\org\apache\mahout\clustering\syntheticcontrol\kmeans\Job.java`和`integration\src\main\java\org\apache\mahout\clustering\conversion\InputDriver.java`的代码，运行KMeans示例还有第4种方法

利用`InputDriver`将文本格式的向量转化为Mahout能识别的向量，

	$ hadoop jar ~/mahout-distribution-0.7/mahout-examples-0.7-job.jar org.apache.mahout.clustering.conversion.InputDriver -i testdata -o vector

利用 mahout 自带的命令行工具，

	$ mahout kmeans -i vector -o output -dm org.apache.mahout.common.distance.EuclideanDistanceMeasure -c centroids -k 6 -cd 0.5 -x 10 -ow 

# 分析聚类结果

## 查看结果

	$ hadoop fs -lsr output
	$ mahout seqdumper -i output/clusteredPoints -o clusteredPoints
	$ vim clusteredPoints
	$ mahout clusterdump -i output/clusters-9-final -o clusters --pointsDir output/clusteredPoints
	$ vim clusters

## 分析结果

# 参考资料
1. [Clustering of synthetic control data](https://cwiki.apache.org/confluence/display/MAHOUT/Clustering+of+synthetic+control+data)
1. [mahout中的kmeans结果分析](http://blog.csdn.net/aidayei/article/details/6665530)