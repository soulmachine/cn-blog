---
layout: post
title: "KNN与K-Means的区别"
date: 2013-02-25 23:41
comments: true
categories: machine-learning
---
##KNN(K-Nearest Neighbor)介绍
Wikipedia上的[KNN词条](http://en.wikipedia.org/wiki/K-nearest_neighbor_algorithm)中有一个比较经典的图如下：

{% img http://yanjiuyanjiu-wordpress.stor.sinaapp.com/uploads/2013/02/022513_0955_KNNKMeans1.png %}

KNN的算法过程是是这样的：

从上图中我们可以看到，图中的数据集是良好的数据，即都打好了label，一类是蓝色的正方形，一类是红色的三角形，那个绿色的圆形是我们待分类的数据。

如果K=3，那么离绿色点最近的有2个红色三角形和1个蓝色的正方形，这3个点投票，于是绿色的这个待分类点属于红色的三角形。

如果K=5，那么离绿色点最近的有2个红色三角形和3个蓝色的正方形，这5个点投票，于是绿色的这个待分类点属于蓝色的正方形。（参考 [酷壳的 K Nearest Neighbor 算法](http://coolshell.cn/articles/8052.html)）

我们可以看到，KNN本质是基于一种数据统计的方法！其实很多机器学习算法也是基于数据统计的。

<!-- more -->

KNN是一种memory-based learning，也叫instance-based learning，属于lazy learning。即它没有明显的前期训练过程，而是程序开始运行时，把数据集加载到内存后，不需要进行训练，就可以开始分类了。

具体是每次来一个未知的样本点，就在附近找K个最近的点进行投票。

再举一个例子，Locally weighted regression (LWR)也是一种 memory-based 方法，如下图所示的数据集。

{% img http://yanjiuyanjiu-wordpress.stor.sinaapp.com/uploads/2013/02/022513_0955_KNNKMeans2.gif %}

用任何一条直线来模拟这个数据集都是不行的，因为这个数据集看起来不像是一条直线。但是每个局部范围内的数据点，可以认为在一条直线上。每次来了一个位置样本x，我们在X轴上以该数据样本为中心，左右各找几个点，把这几个样本点进行线性回归，算出一条局部的直线，然后把位置样本x代入这条直线，就算出了对应的y，完成了一次线性回归。

也就是每次来一个数据点，都要训练一条局部直线，也即训练一次，就用一次。

LWR和KNN是不是很像？都是为位置数据量身定制，在局部进行训练。

##K-Means介绍
{% img http://yanjiuyanjiu-wordpress.stor.sinaapp.com/uploads/2013/02/022513_0955_KNNKMeans3.jpg %}

如图所示，数据样本用圆点表示，每个簇的中心点用叉叉表示。(a)刚开始时是原始数据，杂乱无章，没有label，看起来都一样，都是绿色的。(b)假设数据集可以分为两类，令K=2，随机在坐标上选两个点，作为两个类的中心点。(c-f)演示了聚类的两种迭代。先划分，把每个数据样本划分到最近的中心点那一簇；划分完后，更新每个簇的中心，即把该簇的所有数据点的坐标加起来去平均值。这样不断进行”划分—更新—划分—更新”，直到每个簇的中心不在移动为止。(图文来自Andrew ng的机器学习公开课)。

推荐关于K-Means的两篇博文，[K-Means 算法 _ 酷壳](http://coolshell.cn/articles/7779.html)，[漫谈 Clustering (1)_ k-means pluskid](http://blog.pluskid.org/?p=17)。

##KNN和K-Means的区别
<table style="border-collapse: collapse;" border="0">
<colgroup>
<col style="width: 277px;">
<col style="width: 277px;"></colgroup>
<tbody valign="top">
<tr>
<td style="padding-left: 7px; padding-right: 7px; border: solid 0.5pt;">
<p style="text-align: center;"><span style="font-size: 10pt;"><strong>KNN</strong></span></p>
</td>
<td style="padding-left: 7px; padding-right: 7px; border-top: solid 0.5pt; border-left: none; border-bottom: solid 0.5pt; border-right: solid 0.5pt;">
<p style="text-align: center;"><span style="font-size: 10pt;"><strong>K-Means</strong></span></p>
</td>
</tr>
<tr style="height: 85px;">
<td style="padding-left: 7px; padding-right: 7px; border-top: none; border-left: solid 0.5pt; border-bottom: solid 0.5pt; border-right: solid 0.5pt;"><span style="font-size: 10pt;">1.KNN是分类算法<br>
</span><p></p>
<p><span style="font-size: 10pt;">2.监督学习<br>
</span></p>
<p style="text-align: justify;"><span style="font-size: 10pt;">3.喂给它的数据集是带label的数据，已经是完全正确的数据</span></p>
</td>
<td style="padding-left: 7px; padding-right: 7px; border-top: none; border-left: none; border-bottom: solid 0.5pt; border-right: solid 0.5pt;"><span style="font-size: 10pt;">1.K-Means是聚类算法<br>
</span><p></p>
<p><span style="font-size: 10pt;">2.非监督学习<br>
</span></p>
<p style="text-align: justify;"><span style="font-size: 10pt;">3.喂给它的数据集是无label的数据，是杂乱无章的，经过聚类后才变得有点顺序，先无序，后有序</span></p>
</td>
</tr>
<tr>
<td style="padding-left: 7px; padding-right: 7px; border-top: none; border-left: solid 0.5pt; border-bottom: solid 0.5pt; border-right: solid 0.5pt;"><span style="font-size: 10pt;">没有明显的前期训练过程，属于memory-based learning</span></td>
<td style="padding-left: 7px; padding-right: 7px; border-top: none; border-left: none; border-bottom: solid 0.5pt; border-right: solid 0.5pt;"><span style="font-size: 10pt;">有明显的前期训练过程</span></td>
</tr>
<tr>
<td style="padding-left: 7px; padding-right: 7px; border-top: none; border-left: solid 0.5pt; border-bottom: solid 0.5pt; border-right: solid 0.5pt;"><span style="font-size: 10pt;">K的含义：来了一个样本x，要给它分类，即求出它的y，就从数据集中，在x附近找离它最近的K个数据点，这K个数据点，类别c占的个数最多，就把x的label设为c</span></td>
<td style="padding-left: 7px; padding-right: 7px; border-top: none; border-left: none; border-bottom: solid 0.5pt; border-right: solid 0.5pt;"><span style="font-size: 10pt;">K的含义：K是人工固定好的数字，假设数据集合可以分为K个簇，由于是依靠人工定好，需要一点先验知识</span></td>
</tr>
<tr>
<td style="padding-left: 7px; padding-right: 7px; border-top: none; border-left: solid 0.5pt; border-bottom: solid 0.5pt; border-right: solid 0.5pt;"></td>
<td style="padding-left: 7px; padding-right: 7px; border-top: none; border-left: none; border-bottom: solid 0.5pt; border-right: solid 0.5pt;"></td>
</tr>
<tr>
<td style="padding-left: 7px; padding-right: 7px; border-top: none; border-left: solid 0.5pt; border-bottom: solid 0.5pt; border-right: solid 0.5pt;" colspan="2"><span style="font-size: 10pt;">相似点：都包含这样的过程，给定一个点，在数据集中找离它最近的点。即二者都用到了NN(Nears Neighbor)算法，一般用KD树来实现NN。</span></td>
</tr>
</tbody>
</table>