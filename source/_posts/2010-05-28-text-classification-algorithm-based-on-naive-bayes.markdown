---
layout: post
title: "基于朴素贝叶斯的文本分类算法"
date: 2010-05-28 17:15
comments: true
categories: "machine-learning"
---
作者: 灵魂机器  
新浪博客：[www.weibo.com/soulmachine](www.weibo.com/soulmachine)  
作者博客：[www.yanjiuyanjiu.com](www.yanjiuyanjiu.com)

**摘要**：常用的文本分类方法有支持向量机、K-近邻算法和朴素贝叶斯。其中朴素贝叶斯具有容易实现，运行速度快的特点，被广泛使用。本文详细介绍了朴素贝叶斯的基本原理，讨论了两种常见模型：多项式模型（MM）和伯努利模型（BM），实现了可运行的代码，并进行了一些数据测试。

**关键字**：朴素贝叶斯；文本分类

**Text Classification Algorithm Based on Naive Bayes**
**Author**: soulmachine
**Email**：soulmachine@gmail.com
**Blog**：[www.yanjiuyanjiu.com](www.yanjiuyanjiu.com)


**Abstract**:Usually there are three methods for text classification: SVM、KNN and Naïve Bayes. Naïve Bayes is easy to implement and fast, so it is widely used. This article introduced the theory of Naïve Bayes and discussed two popular models: multinomial model(MM) and Bernoulli model(BM) in details, implemented runnable code and performed some data tests.

**Keywords**: naïve bayes; text classification

##1. 贝叶斯原理
###1.1 贝叶斯公式
