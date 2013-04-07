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

##1 贝叶斯原理

###1.1 贝叶斯公式

设A、B是两个事件，且P(A)>0，称 $$P(Y \vert X)=\dfrac {P(XY)}{P(X)}$$ 为事件A发生的条件下事件B发生的**条件概率**。

**乘法公式** $$P(XYZ)=P(Z \vert XY)P(Y \vert X)P(X)$$  
**全概率公式**  $$P(X)=P(X \vert Y_1)+ P(X \vert Y_2)+…+ P(X \vert Y_n)$$  
**贝叶斯公式**  $$P(Y_i \vert X)=\dfrac{P(XY_i)}{P(X)}=\dfrac{P(X \vert Y_i)P(Y_i)}{P(X)}=\dfrac{P(X \vert Y_i)P(Y_i)}{\sum\limits _{j=1} ^{n} P(X \vert Y_j)}$$  

在此处，贝叶斯公式，我们要用到的是 $$P(Y_i \vert X)=\dfrac{P(X \vert Y_i)P(Y_i)}{P(X)}$$

以上公式，请读者参考[《概率论与数理统计（第五版）》](http://book.douban.com/subject/1231189/)的1.4节“条件概率”（这里将原书中的A换成了X，B换成了Y），获得更深的理解。

<!-- more -->

###1.2 贝叶斯定理在分类中的应用
在分类（classification）问题中，常常需要把一个事物分到某个类别。一个事物具有很多属性，把它的众多属性看做一个向量，即$$x=(x_1,x_2,x_3,…,x_n)$$，用x这个向量来代表这个事物。类别也是有很多种，用集合$$Y={y_1,y_2,…y_m}$$表示。如果x属于$$y_1$$类别，就可以给x打上$$y_1$$标签，意思是说x属于$$y_1$$类别。这就是所谓的**分类(Classification)**。

x的集合记为X，称为属性集。一般X和Y的关系是不确定的，你只能在某种程度上说x有多大可能性属于类$$y_1$$，比如说x有80%的可能性属于类$$y_1$$，这时可以把X和Y看做是随机变量，$$P(Y \vert X)$$称为Y的**后验概率**（posterior probability），与之相对的，P(Y)称为Y的**先验概率**（prior probability）[^2]。

在训练阶段，我们要根据从训练数据中收集的信息，**对X和Y的每一种组合学习后验概率$$P(Y \vert X)$$。**分类时，来了一个实例x，在刚才训练得到的一堆后验概率中找出所有的$$P(Y \vert x)$$， 其中最大的那个y，即为x所属分类。根据贝叶斯公式，后验概率为$$P(Y \vert X)=\dfrac{P(X \vert Y)P(Y)}{P(X)}$$
 
在比较不同Y值的后验概率时，分母P(X)总是常数，**因此可以忽略**。先验概率P(Y)可以通过计算训练集中属于每一个类的训练样本所占的比例容易地估计。

我们来举个简单的例子，让读者对上述思路有个形象的认识[^3]。  
考虑一个医疗诊断问题，有两种可能的假设：（1）病人有癌症。（2）病人无癌症。样本数据来自某化验测试，它也有两种可能的结果：阳性和阴性。假设我们已经有先验知识：在所有人口中只有0.008的人患病。此外，化验测试对有病的患者有98%的可能返回阳性结果，对无病患者有97%的可能返回阴性结果。

上面的数据可以用以下概率式子表示：  
P(cancer)=0.008,P(无cancer)=0.992  
P(阳性|cancer)=0.98,P(阴性|cancer)=0.02  
P(阳性|无cancer)=0.03，P(阴性|无cancer)=0.97  
假设现在有一个新病人，化验测试返回阳性，是否将病人断定为有癌症呢？

在这里，Y={cancer，无cancer}，共两个类别，这个新病人是一个样本，他有一个属性阳性，可以令x=(阳性)。我们可以来计算各个类别的后验概率：  
P(cancer | 阳性) = P(阳性 | cancer)p(cancer)=0.98\* 0.008 = 0.0078  
P(无cancer | 阳性) =P(阳性 | 无cancer)\* p(无cancer)=0.03\* 0.992 = 0.0298   
因此，应该判断为无癌症。

在这个例子中，类条件概率，P(cancer|阳性)和P(无cancer|阳性)直接告诉了我们。

一般地，对**类条件概率$$P(X \vert Y)$$**的估计，有朴素贝叶斯分类器和贝叶斯信念网络两种方法，这里介绍朴素贝叶斯分类器。

###1.3 朴素贝叶斯分类器
**1、条件独立性**  
给定类标号y，朴素贝叶斯分类器在估计类条件概率时假设属性之间条件独立。条件独立假设可以形式化的表达如下：  
$$
\prod\limits_{i=1}^{n} P(x_i  \vert Y=y)
$$
其中每个训练样本可用一个属性向量$$X=(x_1,x_2,x_3,…,x_n)$$表示，各个属性之间条件独立。

比如，对于一篇文章，
 
> Good good study,Day day up.

可以用一个文本特征向量来表示，`x=(Good, good, study, Day, day , up)`。一般各个词语之间肯定不是相互独立的，有一定的上下文联系。但在朴素贝叶斯文本分类时，我们假设个单词之间没有联系，可以用一个文本特征向量来表示这篇文章，这就是“朴素”的来历。

**2、朴素贝叶斯如何工作**  
**有了条件独立假设，就不必计算X和Y的每一种组合的类条件概率**，只需对给定的Y，计算每个$$x_i$$的条件概率。后一种方法更实用，因为它不需要很大的训练集就能获得较好的概率估计。

**3、估计分类属性的条件概率**  
$$P(x_i \vert Y=y)$$怎么计算呢？它一般根据类别y下包含属性$$x_i$$的实例的比例来估计。以文本分类为例，xi表示一个单词，$$P(x_i \vert Y=y)=$$包含该类别下包含单词的xi的文章总数/ 该类别下的文章总数。

**4、贝叶斯分类器举例**
假设给定了如下训练样本数据，我们学习的目标是根据给定的天气状况判断你对PlayTennis这个请求的回答是Yes还是No。

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td valign="top" width="95">Day</td>
<td valign="top" width="95">Outlook</td>
<td valign="top" width="95">Temperature</td>
<td valign="top" width="95">Humidity</td>
<td valign="top" width="95">Wind</td>
<td valign="top" width="95">PlayTennis</td>
</tr>
<tr>
<td valign="top" width="95">D1</td>
<td valign="top" width="95">Sunny</td>
<td valign="top" width="95">Hot</td>
<td valign="top" width="95">High</td>
<td valign="top" width="95">Weak</td>
<td valign="top" width="95">No</td>
</tr>
<tr>
<td valign="top" width="95">D2</td>
<td valign="top" width="95">Sunny</td>
<td valign="top" width="95">Hot</td>
<td valign="top" width="95">High</td>
<td valign="top" width="95">Strong</td>
<td valign="top" width="95">No</td>
</tr>
<tr>
<td valign="top" width="95">D3</td>
<td valign="top" width="95">Overcast</td>
<td valign="top" width="95">Hot</td>
<td valign="top" width="95">High</td>
<td valign="top" width="95">Weak</td>
<td valign="top" width="95">Yes</td>
</tr>
<tr>
<td valign="top" width="95">D4</td>
<td valign="top" width="95">Rain</td>
<td valign="top" width="95">Mild</td>
<td valign="top" width="95">High</td>
<td valign="top" width="95">Weak</td>
<td valign="top" width="95">Yes</td>
</tr>
<tr>
<td valign="top" width="95">D5</td>
<td valign="top" width="95">Rain</td>
<td valign="top" width="95">Cool</td>
<td valign="top" width="95">Normal</td>
<td valign="top" width="95">Weak</td>
<td valign="top" width="95">Yes</td>
</tr>
<tr>
<td valign="top" width="95">D6</td>
<td valign="top" width="95">Rain</td>
<td valign="top" width="95">Cool</td>
<td valign="top" width="95">Normal</td>
<td valign="top" width="95">Strong</td>
<td valign="top" width="95">No</td>
</tr>
<tr>
<td valign="top" width="95">D7</td>
<td valign="top" width="95">Overcast</td>
<td valign="top" width="95">Cool</td>
<td valign="top" width="95">Normal</td>
<td valign="top" width="95">Strong</td>
<td valign="top" width="95">Yes</td>
</tr>
<tr>
<td valign="top" width="95">D8</td>
<td valign="top" width="95">Sunny</td>
<td valign="top" width="95">Mild</td>
<td valign="top" width="95">High</td>
<td valign="top" width="95">Weak</td>
<td valign="top" width="95">No</td>
</tr>
<tr>
<td valign="top" width="95">D9</td>
<td valign="top" width="95">Sunny</td>
<td valign="top" width="95">Cool</td>
<td valign="top" width="95">Normal</td>
<td valign="top" width="95">Weak</td>
<td valign="top" width="95">Yes</td>
</tr>
<tr>
<td valign="top" width="95">D10</td>
<td valign="top" width="95">Rain</td>
<td valign="top" width="95">Mild</td>
<td valign="top" width="95">Normal</td>
<td valign="top" width="95">Weak</td>
<td valign="top" width="95">Yes</td>
</tr>
<tr>
<td valign="top" width="95">D11</td>
<td valign="top" width="95">Sunny</td>
<td valign="top" width="95">Mild</td>
<td valign="top" width="95">Normal</td>
<td valign="top" width="95">Strong</td>
<td valign="top" width="95">Yes</td>
</tr>
<tr>
<td valign="top" width="95">D12</td>
<td valign="top" width="95">Overcast</td>
<td valign="top" width="95">Mild</td>
<td valign="top" width="95">High</td>
<td valign="top" width="95">Strong</td>
<td valign="top" width="95">Yes</td>
</tr>
<tr>
<td valign="top" width="95">D13</td>
<td valign="top" width="95">Overcast</td>
<td valign="top" width="95">Hot</td>
<td valign="top" width="95">Normal</td>
<td valign="top" width="95">Weak</td>
<td valign="top" width="95">Yes</td>
</tr>
<tr>
<td valign="top" width="95">D14</td>
<td valign="top" width="95">Rain</td>
<td valign="top" width="95">Mild</td>
<td valign="top" width="95">High</td>
<td valign="top" width="95">Strong</td>
<td valign="top" width="95">No</td>
</tr>
</tbody>
</table> 
可以看到这里样本数据集提供了14个训练样本，我们将使用此表的数据，并结合朴素贝叶斯分类器来分类下面的新实例：  
x = (Outlook = Sunny,Temprature = Cool,Humidity = High,Wind = Strong)

在这个例子中，属性向量X=(Outlook, Temperature, Humidity, Wind)，类集合Y={Yes, No}。我们需要利用训练数据计算后验概率$$P(Yes \vert x)$$和$$P(No \vert x)$$，如果$$P(Yes \vert x)>P(No \vert x)$$，那么新实例分类为Yes，否则为No。

为了计算后验概率，我们需要计算先验概率P(Yes)和P(No)和类条件概率$$P(x_i \vert Y)$$。

因为有9个样本属于Yes，5个样本属于No，所以$$P(Yes)=\dfrac{9}{14}$$, $$P(No)=\dfrac{5}{14}$$。类条件概率计算如下：  
$$P(Outlook = Sunny \vert Yes)=\dfrac{2}{9}　　　P(Outlook = Sunny \vert No)=\dfrac{3}{5}$$  
$$P(Temprature = Cool  \vert Yes) =\dfrac{3}{9}　　　P(Temprature = Cool  \vert No) =\dfrac{1}{5}$$  
$$P(Humidity = High  \vert Yes) =\dfrac{3}{9}　　　P(Humidity = High  \vert No) =\dfrac{4}{5}$$
$$P(Wind = Strong  \vert Yes) =\dfrac{3}{9}　　　P(Wind = Strong  \vert No) =\dfrac{3}{5}$$    

后验概率计算如下：
$$
\begin{aligned}
P(Yes  \vert  x) & = P(Outlook = Sunny \vert Yes) \times P(Temprature = Cool  \vert Yes) \newline
& \times P(Humidity = High  \vert Yes) \times P(Wind = Strong  \vert Yes) \times P(Yes) \newline
& =\dfrac{2}{9} \times \dfrac{3}{9} \times \dfrac{3}{9} \times \dfrac{3}{9} \times \dfrac{3}{9} \times \dfrac{9}{14}=\dfrac{2}{243}=\dfrac{9}{1701} \approx 0.00529
\end{aligned}
$$
$$
\begin{aligned}
P(No  \vert  x)&= P(Outlook = Sunny \vert No) \times P(Temprature = Cool  \vert No) \newline
& \times P(Humidity = High  \vert No) \times P(Wind = Strong  \vert No) \times P(No) \newline
& =\dfrac{3}{5}\times \dfrac{1}{5} \times \dfrac{4}{5} \times \dfrac{3}{5} \times  \dfrac{5}{14}=\dfrac{18}{875} \approx 0.02057
\end{aligned}
$$
通过计算得出$$P(No  \vert  x)> P(Yes  \vert  x)$$，所以该样本分类为No[^3]。

**5、条件概率的m估计**  
假设有来了一个新样本 $$x_1= (Outlook = Cloudy,Temprature = Cool,Humidity = High,Wind = Strong)$$，要求对其分类。我们来开始计算，  
$$P(Outlook = Cloudy \vert Yes)=\dfrac{0}{9}=0  P(Outlook = Cloudy  \vert No)=\dfrac{0}{5}=0$$  
计算到这里，大家就会意识到，这里出现了一个新的属性值，在训练样本中所没有的。如果有一个属性的类条件概率为0，则整个类的后验概率就等于0，我们可以直接得到后验概率$$P(Yes  \vert  x_1)= P(No  \vert  x_1)=0$$，这时二者相等，无法分类。

当训练样本不能覆盖那么多的属性值时，都会出现上述的窘境。简单的使用样本比例来估计类条件概率的方法太脆弱了，尤其是当训练样本少而属性数目又很大时。

解决方法是使用m估计(m-estimate)方法来估计条件概率：
$$
P(x_i \vert y_i)=\dfrac{n_c+mp}{n+m}
$$
n是类$$y_j$$中的样本总数，$$n_c$$是类$$y_j$$中取值$$x_i$$的样本数，m是称为等价样本大小的参数，而p是用户指定的参数。如果没有训练集（即n=0），则$$P(x_i \vert y_i)=p$$, 因此p可以看作是在类$$y_j$$的样本中观察属性值$$x_i$$的先验概率。等价样本大小决定先验概率和观测概率$$\dfrac{n_c}{n}$$之间的平衡[^2]。

##2 朴素贝叶斯文本分类算法
现在开始进入本文的主旨部分：如何将贝叶斯分类器应用到文本分类上来。

###2.1文本分类问题
在文本分类中，假设我们有一个文档d∈X，X是文档向量空间(document space)，和一个固定的类集合C={c1,c2,…,cj}，类别又称为标签。显然，文档向量空间是一个高维度空间。我们把一堆打了标签的文档集合<d,c>作为训练样本，<d,c>∈X×C。例如：  
<d,c>={Beijing joins the World Trade Organization, China}  
对于这个只有一句话的文档，我们把它归类到 China，即打上china标签。

我们期望用某种训练算法，训练出一个函数γ，能够将文档映射到某一个类别：
γ:X→C

这种类型的学习方法叫做有监督学习，因为事先有一个监督者（我们事先给出了一堆打好标签的文档）像个老师一样监督着整个学习过程。

朴素贝叶斯分类器是一种有监督学习，常见有两种模型，多项式模型(multinomial model)和伯努利模型(Bernoulli model)[^4]。

###2.2 多项式模型

####2.2.1 基本原理
在多项式模型中， 设某文档$$d=(t_1,t_2,…,t_k)$$，tk是该文档中出现过的单词，允许重复，则  
先验概率$$P(c)=$$ 类c下单词总数/整个训练样本的单词总数  
类条件概率$$P(t_k \vert c)=$$(类c下单词tk在各个文档中出现过的次数之和+1)/(类c下单词总数+|V|)

V是训练样本的单词表（即抽取单词，单词出现多次，只算一个），`|V|`则表示训练样本包含多少种单词。在这里，`m=|V|, p=1/|V|`。

$$P(t_k \vert c)=$$可以看作是单词tk在证明d属于类c上提供了多大的证据，而P(c)则可以认为是类别c在整体上占多大比例(有多大可能性)。

####2.2.2 伪代码[^1]
``` java
//C，类别集合，D，用于训练的文本文件集合
TrainMultiNomialNB(C,D) {
    // 单词出现多次，只算一个
    V←ExtractVocabulary(D)
    // 单词可重复计算
    N←CountTokens(D)
    for each c∈C
        // 计算类别c下的单词总数
        // N和Nc的计算方法和Introduction to Information Retrieval上的不同，个人认为
        //该书是错误的，先验概率和类条件概率的计算方法应当保持一致
        Nc←CountTokensInClass(D,c)
        prior[c]←Nc/N
        // 将类别c下的文档连接成一个大字符串
        textc←ConcatenateTextOfAllDocsInClass(D,c)
        for each t∈V
            // 计算类c下单词t的出现次数
            Tct←CountTokensOfTerm(textc,t)
        for each t∈V
            //计算P(t|c)
            condprob[t][c]← 
    return V,prior,condprob
}

ApplyMultiNomialNB(C,V,prior,condprob,d) {
    // 将文档d中的单词抽取出来，允许重复，如果单词是全新的，在全局单词表V中都
    // 没出现过，则忽略掉
    W←ExtractTokensFromDoc(V,d)
    for each c∈C
        score[c]←prior[c]
        for each t∈W
            if t∈Vd
                score[c] *= condprob[t][c]
    return max(score[c])
}
```

####2.2.3 举例
给定一组分类好了的文本训练数据，如下：  

<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td valign="top" width="64">docId</td>
<td valign="top" width="236">doc</td>
<td valign="top" width="126">类别In c=China?</td>
</tr>
<tr>
<td valign="top" width="64">1</td>
<td valign="top" width="236">Chinese Beijing Chinese</td>
<td valign="top" width="126">yes</td>
</tr>
<tr>
<td valign="top" width="64">2</td>
<td valign="top" width="236">Chinese Chinese Shanghai</td>
<td valign="top" width="126">yes</td>
</tr>
<tr>
<td valign="top" width="64">3</td>
<td valign="top" width="236">Chinese Macao</td>
<td valign="top" width="126">yes</td>
</tr>
<tr>
<td valign="top" width="64">4</td>
<td valign="top" width="236">Tokyo Japan Chinese</td>
<td valign="top" width="126">no</td>
</tr>
</tbody>
</table>

给定一个新样本
> Chinese Chinese Chinese Tokyo Japan

对其进行分类。该文本用属性向量表示为`d=(Chinese, Chinese, Chinese, Tokyo, Japan)`，类别集合为`Y={yes, no}`。

类yes下总共有8个单词，类no下总共有3个单词，训练样本单词总数为11，因此$$P(yes)=\dfrac{8}{11}, P(no)=\dfrac{3}{11}$$。类条件概率计算如下：  
$$P(Chinese  \vert  yes)=\dfrac{5+1}{8+6}=\dfrac{6}{14}=\dfrac{3}{7}$$  
$$P(Japan  \vert  yes)=P(Tokyo  \vert  yes)= \dfrac{0+1}{8+6}=\dfrac{1}{14}$$  
$$P(Chinese \vert no)=\dfrac{1+1}{3+6}=\dfrac{2}{9}$$  
$$P(Japan \vert no)=P(Tokyo \vert  no) =\dfrac{1+1}{3+6}=\dfrac{2}{9}$$  
分母中的8，是指yes类别下textc的长度，也即训练样本的单词总数，6是指训练样本有Chinese,Beijing,Shanghai, Macao, Tokyo, Japan 共6个单词，3是指no类下共有3个单词。

有了以上类条件概率，开始计算后验概率，  
$$P(yes  \vert  d)=\left(\dfrac{3}{7}\right)^3 \times \dfrac{1}{14} \times \dfrac{1}{14} \times \dfrac{8}{11}=\dfrac{108}{184877} \approx 0.00058417$$  
$$P(no  \vert  d)= \left(\dfrac{2}{9}\right)^3 \times \dfrac{2}{9} \times \dfrac{2}{9} \times \dfrac{3}{11}=\dfrac{32}{216513} \approx 0.00014780$$  
因此，这个文档属于类别china。

###2.3 伯努利模型

####2.3.1 基本原理
$$P(c)=$$ 类c下文件总数/整个训练样本的文件总数  
$$P(t_k \vert c)=$$(类c下包含单词tk的文件数+1)/(类c下单词总数+2)  
在这里，$$m=2, p=\dfrac{1}{2}$$。

后验概率的计算，也有点变化，见下面的伪代码。

####2.3.2 伪代码
``` java
//C，类别集合，D，用于训练的文本文件集合
TrainBernoulliNB(C, D) {
    // 单词出现多次，只算一个
V←ExtractVocabulary(D)
    // 计算文件总数
    N←CountDocs(D)
    for each c∈C
        // 计算类别c下的文件总数
        Nc←CountDocsInClass(D,c)
        prior[c]←Nc/N
        for each t∈V
            // 计算类c下包含单词t的文件数
            Nct←CountDocsInClassContainingTerm(D,c,t)
            //计算P(t|c)
            condprob[t][c]←(Nct+1)/(Nct+2)
    return V,prior,condprob
}

ApplyBernoulliNB(C,V,prior,condprob,d) {
    // 将文档d中单词表抽取出来，如果单词是全新的，在全局单词表V中都没出现过，
    // 则舍弃
    Vd←ExtractTermsFromDoc(V,d)
    for each c∈C
        score[c]←prior[c]
        for each t∈V
            if t∈Vd
                score[c] *= condprob[t][c]
            else
                score[c] *= (1-condprob[t][c])
    return max(score[c])
}
```

####2.3.3 举例
还是使用前面例子中的数据，不过模型换成了使用伯努利模型。

类yes下总共有3个文件，类no下有1个文件，训练样本文件总数为11，因此$$P(yes)=\dfrac{3}{4}, P(Chinese  \vert  yes)=\dfrac{3+1}{3+2}=\dfrac{4}{5}$$  
$$P(Japan  \vert  yes)=P(Tokyo  \vert  yes)=\dfrac{0+1}{3+2}=\dfrac{1}{5}$$  
$$P(Beijing  \vert  yes)= P(Macao \vert yes)= P(Shanghai  \vert yes)=\dfrac{1+1}{3+2}=\dfrac{2}{5}$$  
$$P(Chinese \vert no)=\dfrac{1+1}{1+2}=\dfrac{2}{3}$$  
$$P(Japan \vert no)=P(Tokyo \vert  no) =\dfrac{1+1}{1+2}=\dfrac{2}{3}$$  
$$P(Beijing \vert  no)= P(Macao \vert  no)= P(Shanghai  \vert  no)=\dfrac{0+1}{1+2}=\dfrac{1}{3}$$  

有了以上类条件概率，开始计算后验概率，  
$$
\begin{aligned}
P(yes  \vert  d)&=P(yes) \times P(Chinese \vert yes) \times P(Japan \vert yes) \times P(Tokyo \vert yes) \newline
&\times (1-P(Beijing \vert yes)) \times (1-P(Shanghai \vert yes))\newline
&\times (1-P(Macao \vert yes)) \newline
&=\dfrac{3}{4} \times \dfrac{4}{5} \times \dfrac{1}{5} \times \dfrac{1}{5} \times (1-\dfrac{2}{5} \times (1-\dfrac{2}{5}) \times (1-\dfrac{2}{5})=\dfrac{81}{15625} \approx 0.005
\end{aligned}
$$
$$P(no   \vert   d)= \dfrac{1}{4} \times \dfrac{2}{3} \times \dfrac{2}{5} \times \dfrac{2}{5} \times (1-\dfrac{1}{3}) \times (1-\dfrac{1}{3}) \times (1-\dfrac{1}{3})=\dfrac{16}{729} \approx 0.022$$  
因此，这个文档不属于类别china。

###2.4 两个模型的区别
二者的计算粒度不一样，多项式模型以单词为粒度，伯努利模型以文件为粒度，因此二者的先验概率和类条件概率的计算方法都不同。

计算后验概率时，对于一个文档d，多项式模型中，只有在d中出现过的单词，才会参与后验概率计算，伯努利模型中，没有在d中出现，但是在全局单词表中出现的单词，也会参与计算，不过是作为“反方”参与的。

##3 代码详解
本文附带了一个eclipse工程，有完整的源代码，以及一个微型文本训练库。

ChineseSpliter用于中文分词，StopWordsHandler用于判断一个单词是否是停止词，ClassifyResult用于保存结果，IntermediateData用于预处理文本语料库，TrainnedModel用于保存训练后得到的数据，NaiveBayesClassifier是基础类，包含了贝叶斯分类器的主要代码，MultiNomialNB是多项式模型，类似的，BernoulliNB是伯努利模型，二者都继承自NaiveBayesClassifier，都只重写了父类的计算先验概率，类条件概率和后验概率这3个函数。

###3.1 中文分词
中文分词不是本文的重点，这里我们直接使用第三方工具，本源码使用的是[极易中文分词组件](http://www.jesoft.cn/)，你还可以使用[MMSEG](http://chtsai.org/)，中科院的[ICTCLAS](http://ictclas.org/)等等。

``` java
/**
     * 对给定的文本进行中文分词.
     * 
     * @param text
     *            给定的文本
     * @param splitToken
     *            用于分割的标记,如"|"
     * @return 分词完毕的文本
     */
    public String split(final String text, final String splitToken) {
        String result = null;
        
        try {
            result = analyzer.segment(text, splitToken);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
}
```

###3.2 停止词处理
停止词(Stop Word)是指那些无意义的字或词，如“的”、“在”等。去掉文档中的停止词也是必须的一项工作,这里简单的定义了一些常见的停止词，并根据这些常用停止词在分词时进行判断。

``` java
/** 常用停用词. */
    private static String[] stopWordsList = {
            // 来自 c:\Windows\System32\NOISE.CHS
            "的", "一", "不", "在", "人", "有", "是", "为", "以", "于", "上", "他", "而",
            "后", "之", "来", "及", "了", "因", "下", "可", "到", "由", "这", "与", "也",
            "此", "但", "并", "个", "其", "已", "无", "小", "我", "们", "起", "最", "再",
            "今", "去", "好", "只", "又", "或", "很", "亦", "某", "把", "那", "你", "乃",
            "它",
            // 来自网络
            "要", "将", "应", "位", "新", "两", "中", "更", "我们", "自己", "没有", "“", "”",
            "，", "（", "）", "" };

    /**
     * 判断一个词是否是停止词.
     * 
     * @param word
     *            要判断的词
     * @return 是停止词，返回true，否则返回false
     */
    public static boolean isStopWord(final String word) {
        for (int i = 0; i < stopWordsList.length; ++i) {
            if (word.equalsIgnoreCase(stopWordsList[i])) {
                return true;
            }
        }
        return false;
    }
```

###3.3 预处理数据
我们这里使用[搜狗的文本分类语料库](http://www.sogou.com/labs/dl/c.html)作为训练样本，把SogouC.reduced.20061102.tar.gz解压到D盘，目录结构为

``` bash
D:\Reduced
         |-- C000008
         |-- C000010
         |-- C000013
         |-- C000014
         |-- C000016
         |-- C000020
         |-- C000022
         |-- C000023
         |-- C000024
```
IntermediateData.java主要用于处理文本数据，将所需要的信息计算好，存放到数据库文件中。

中间数据文件主要保存了如下信息，

``` java
/** 单词X在类别C下出现的总数. */
	public HashMap[] filesOfXC;
	/** 给定分类下的文件数目. */
    public int[] filesOfC;
    /** 根目录下的文件总数. */
    public int files;
    
	/** 单词X在类别C下出现的总数 */
	public HashMap[] tokensOfXC;
    /** 类别C下所有单词的总数. */
    public int[] tokensOfC;
    /** 整个语料库中单词的总数. */
    public int tokens;
    /** 整个训练语料所出现的单词. */
    public HashSet<String> vocabulary;
```
我们使用命令

``` bash
IntermediateData d:\Reduced\ gbk d:\reduced.db
```
将文本训练库的信息计算好，保存到中间文件中。以后的阶段，我们都不再需要文本语料库了，只需要reduced.db。


###3.3 训练
基本的框架代码都在NaiveBayesClassifier中，MultiNomialNB和BernoulliNB都只是重新实现(override)了这三个函数。

``` java
/** 计算先验概率P(c). */
    protected void calculatePc() {
    }
    
    /** 计算类条件概率P(x|c). */
    protected void calculatePxc() {
    }
    
    
    /**
     * 计算文本属性向量X在类Cj下的后验概率P(Cj|X).
     * 
     * @param x
     *            文本属性向量
     * @param cj
     *            给定的类别
     * @return 后验概率
     */
    protected double calcProd(final String[] x, final int cj) {
        return 0;
    }
```

训练函数如下：

```
public final void train(String intermediateData, String modelFile) {
    	// 加载中间数据文件
    	loadData(intermediateData);
    	
    	model = new TrainnedModel(db.classifications.length);
    	
    	model.classifications = db.classifications;
    	model.vocabulary = db.vocabulary;
    	// 开始训练
    	calculatePc();
    	calculatePxc();
    	db = null;
    	
    	try {
    		// 用序列化，将训练得到的结果存放到模型文件中
            ObjectOutputStream out = new ObjectOutputStream(
                    new FileOutputStream(modelFile));
            out.writeObject(model);
            out.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
}
```
我们使用命令：

``` bash
MultiNomialNB –t d:\reduced.db d:\reduced.mdl
```
开始训练，得到的模型文件保存在reduced.mdl中。

###3.4 分类
有了模型文件，就可以用它来进行分类了。

可以使用命令

``` bash
MultiNomialNB d:\reduced.mdl d:\temp.txt gbk
```
对文本文件temp.txt进行分类。

还可以将当初训练出这个模型文件的文本库，进行分类，看看正确率有多少，即“吃自己的狗食”，命令行如下

``` bash
MultiNomialNB -r d:\reduced\ gbk d:\reduced.mdl
```

分类函数如下：

``` java
/**
 * 对给定的文本进行分类.
 * 
 * @param text
 *            给定的文本
 * @return 分类结果
 */
public final String classify(final String text) {
    String[] terms = null;
    // 中文分词处理(分词后结果可能还包含有停用词）
terms = textSpliter.split(text, " ").split(" ");
    // 去掉停用词，以免影响分类
    terms = ChineseSpliter.dropStopWords(terms); 
        
double probility = 0.0;
    // 分类结果
    List<ClassifyResult> crs = new ArrayList<ClassifyResult>(); 
    for (int i = 0; i < model.classifications.length; i++) {
        // 计算给定的文本属性向量terms在给定的分类Ci中的分类条件概率
        probility = calcProd(terms, i);
        // 保存分类结果
        ClassifyResult cr = new ClassifyResult();
         cr.classification = model.classifications[i]; // 分类
        cr.probility = probility; // 关键字在分类的条件概率
        System.out.println("In process....");
        System.out.println(model.classifications[i] + "：" + probility);
        crs.add(cr);
    }
        
    // 找出最大的元素
    ClassifyResult maxElem = (ClassifyResult) java.util.Collections.max(
            crs, new Comparator() {
                public int compare(final Object o1, final Object o2) {
                    final ClassifyResult m1 = (ClassifyResult) o1;
                    final ClassifyResult m2 = (ClassifyResult) o2;
                    final double ret = m1.probility - m2.probility;
                    if (ret < 0) {
                        return -1;
                    } else {
                        return 1;
                    }
                }
            });

    return maxElem.classification;
}
```
测试正确率的函数getCorrectRate()，核心代码就是对每个文本文件调用classify()，将得到的类别和原始的类别比较，经过统计后就可以得到百分比。

更多细节请读者阅读[源代码](http://yanjiuyanjiu-wordpress.stor.sinaapp.com/uploads/2010/05/NaiveBayesClassifier.zip)。

##参考文献

[^1]: Christopher D. Manning, Prabhakar Raghavan, Hinrich Schütze, [Introduction to Information Retrieval](http://nlp.stanford.edu/IR-book/), Cambridge University Press, 2008, chapter 13, Text classification and Naive Bayes.

[^2]: Pang-Ning Tan, Michael Steinbach, Vipin Kumar, 《[数据挖掘导论](http://book.douban.com/subject/1786120/)》，北京：人民邮电出版社，2007，第140~145页。

[^3]: 石志伟, 吴功宜, “[基于朴素贝叶斯分类器的文本分类算法](http://d.wanfangdata.com.cn/Conference_5615512.aspx)”, 第一届全国信息检索与内容安全学术会议，2004

[^4]: 洞庭散人，“[基于朴素贝叶斯分类器的文本分类算法（上）](http://www.cnblogs.com/phinecos/archive/2008/10/21/1315948.html)”，“[基于朴素贝叶斯分类器的文本分类算法（下）](http://www.cnblogs.com/phinecos/archive/2008/10/21/1316044.html)”，2008

[^5]: DL88250, “[朴素贝叶斯中文文本分类器的研究与实现（1）](http://blog.csdn.net/DL88250/archive/2008/02/20/2108164.aspx)”，“[朴素贝叶斯中文文本分类器的研究与实现（2）](http://blog.csdn.net/DL88250/archive/2008/03/27/2224126.aspx)”，2008
