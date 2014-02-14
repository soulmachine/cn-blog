---
layout: post
title: "数值计算库与科学计算库"
date: 2013-02-26 23:15
comments: true
categories: Machine-Learning
---
##BLAS 接口
[BLAS](http://www.netlib.org/blas/), [LAPACK](http://www.netlib.org/lapack/), [ATLAS](http://math-atlas.sourceforge.net/) 这些数值计算库的名字很类似，他们之间有什么关系呢？BLAS是一组线性代数运算接口，目前是事实上的标准，很多数值计算/科学计算都实现了这套接口。

BLAS定义了那些函数呢？可以查看[官方文档](http://www.netlib.org/blas/)。

LAPACK是BLAS的第一个实现，是最老牌的数值计算库，用FORTRAN 77语言写的。LAPACK实现了BLAS接口，并扩充了一些功能。很多数值计算库/科学计算库底层调用了LAPACK。

很多硬件厂商都实现BLAS接口，例如[Intel MKL](http://software.intel.com/en-us/intel-mkl)(Math Kernel Library), [AMCL](http://developer.amd.com/tools/cpu-development/amd-core-math-library-acml/)(AMD Math Core Library)等。很多开源库也支持，例如ATLAS。

还有非常多的库实现了BLAS接口，见[Wikipedia BLAS](http://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms) 的Implementations小节。

下面介绍一些各种语言常用的数值计算/科学计算库。

<!-- more -->

##C/C++
首先是Intel 的MKL 和 AMD 的AMCL，性能一流，不过是商业软件，价格昂贵。

[GSL - GNU Scientific Library](http://www.gnu.org/software/gsl/)，GNU实现的库，质量很高，不过是用纯C写的，用起来比较繁琐。

[Armadillo](http://arma.sourceforge.net/)，最新版 2013-02-20 Version 3.6.3

[IT++](http://itpp.sourceforge.net/)，最后版本是4.2,2010-09-21。

##Java
这个页面[JavaNumerics page](http://math.nist.gov/javanumerics/)专门收集了关于Java数值计算的库。

[java-matrix-benchmark](https://code.google.com/p/java-matrix-benchmark/)这个开源项目，比较了各类Java线性代数库的性能。

Java的数值计算库主要分为两类：Pure Java和Natie Wrapper。Pure Java是指用纯Java编写的，Native Wrapper是指该库底层调用了C++或Fortan编写的第三方库，上面封装了一层，提供了更有好的接口。

Pure Java的有：[Colt](http://dsd.lbl.gov/~hoschek/colt/), [Commons Math](http://commons.apache.org/proper/commons-math/), [EJML](https://code.google.com/p/efficient-java-matrix-library/), [JAMA](http://math.nist.gov/javanumerics/jama/), [Trove](http://trove.starlight-systems.com/)

Native Wrapper有：[jblas](http://jblas.org)，[Matrix Toolkit Java](https://github.com/fommil/matrix-toolkits-java)

下面介绍一些影响力较大的java数值计算/科学计算库。

[Commons Math](http://commons.apache.org/proper/commons-math/), 最新版本是3.1.1,2013年1月9号发布。这个库提供一些基本的数学运算，没有high-level的东西，例如矩阵，向量等，用起来会比较繁琐。

[JAMA](http://math.nist.gov/javanumerics/jama/), 最新版是Version 1.0.3 (November 9, 2012)。

[Colt](http://acs.lbl.gov/software/colt/)，已经不更新了，最后版本是1.2.0，2004年9月发布的。

Apache Mahout使用了Colt作为high performance collections，见官方[这个页面](https://cwiki.apache.org/MAHOUT/mahout-collections.html)，说“The implementation of Mahout Collections is derived from Cern Colt”，以及quora 这个帖子[What are the best resources for distributed numerical analysis/matrix algorithms](http://www.quora.com/Distributed-Algorithms/What-are-the-best-resources-for-distributed-numerical-analysis-matrix-algorithms)。

##Python
目前最有影响力的莫过于[NumPy](http://www.numpy.org/)和[SciPy](http://www.scipy.org/)。Amazon.com上可以搜到专门讲它们的书。

SciPy依赖NumPy，主要是在数值计算方面调用了NumPy。

##Ruby
[SciRuby](http://sciruby.com/), 是SciPy和NumPy的克隆，目前还在开发中。

##R
R刚开始时是统计学家开发的语言，专门用于数理统计，现在功能不断增强，内置了很多数值计算和科学计算的功能。R在数据分析领域比较火。

##Scala
目前用google搜索 “scala numerical computing”，能找得到的就是[ScalaLab](http://code.google.com/p/scalalab/)了。

##Matlab
最后，别忘了Matlab是支持多语言调用的。

可以用Matlab生成DLL，给C/C++语言调用。其实，凡是能调用DLL的语言，都可以使用这个DLL，例如Python, Ruby等。

可以用[Matlab JavaBuilder](http://www.mathworks.cn/products/javabuilder/)将m文件转换为jar文件，然后在java代码中就可以调用了。

##如何选择
本文的重点在于选择一个高性能，同时又比较易用的库，即被让我们调用，用来写程序的库，不是一个集成环境或REPL环境。因此R和Matlab不在讨论范围内。R和Matlab用来做原型或前期Data Exploration比较适合。

选择一个工具（语言，框架，库等），要看其是否成熟。我个人的一些判断指标，主要有

1. 有没有大厂商的支持（作为vendor之类的）；
2. amazon.com上能否搜到书。

从厂商的支持来看，几个主要的大厂商如 Intel，AMD和Apple都开发了自己的数学库。Python则有很成熟的NumPy，在Amazon上能搜到书，例如“SciPy and NumPy”， “NumPy Cookbook”。 因此，目前来看，C++和Python是比较成熟的方案。

##参考资料
[Wikipedia BLAS](http://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms)  
[Wikipedia LAPACK](http://en.wikipedia.org/wiki/LAPACK)  
[用 BLAS/LAPACK 编写矩阵运算程序](http://blog.henix.info/blog/blas-lapack-do-matrix-operation.html)  
[BLAS, LAPACK, ATLAS](https://wikis.utexas.edu/display/~cdupree/BLAS,+LAPACK,+ATLAS)  
[BLAS 和 LAPACK ，以及其他常用数值计算库](http://hi.baidu.com/luckykele2012/item/6a3b25423018c40d6dc2f090)  
[Any numerical computing environment on Java platform](http://fdatamining.blogspot.com/2011/10/any-numerical-computing-environment-on.html)  
[C++ Libraries for Scientific Computing](http://www.myoutsourcedbrain.com/2009/04/c-libraries-for-numerical-processing.html)  
[Scientific Library Options for C or C++](http://stackoverflow.com/questions/3121139/scientific-library-options-for-c-or-c)  
[Why is Python used for high-performance/scientific computing (but Ruby isn't)?](http://programmers.stackexchange.com/questions/138643/why-is-python-used-for-high-performance-scientific-computing-but-ruby-isnt)  

