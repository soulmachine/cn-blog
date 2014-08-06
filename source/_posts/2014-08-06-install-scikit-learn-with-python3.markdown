---
layout: post
title: "安装基于Python3 的NumPy, SciPy和Scikit-Learn"
date: 2014-08-06 09:04
comments: true
categories: Python
---
软件版本：Ubuntun 14.04, Python 3.4, NumPy 1.8.1, SciPy 0.14.0, Scikit-Learn 0.16

Numpy, SciPy 的官网安装文档，安装的是基于Python 2.7的，SciPy-Learn 官网的安装文档，也是Python 2.7的，如果想基于高大上的Python 3，该怎么安装呢？经过一堆的坑之后，我摸索出了方法。

##1. 安装Python 3
首先我们要安装Python 3, 不过，千万别因为有了Python 3, 就卸载系统自带的Python 2.7，很多软件依赖它，所以不能卸载

``` bash
$ sudo apt-get install python3
```

设置Python3为默认Python

``` bash
$ vi ~/.bash_aliases
$ alias python=python3
  wq
```

关闭当前Shell，重新开一个新Shell，输入python就发现进入Python 3.4 的交互环境了。

##2. 安装 NumPy SciPy SymPy 等软件
参考 <http://www.scipy.org/install.html> , 只不过改成了 python3

``` bash
sudo apt-get install python3-numpy python3-scipy python3-matplotlib ipython3 ipython3-notebook python3-pandas python-sympy python3-nose
```

##3. 安装 Scikit-Learn
参考 <http://scikit-learn.org/stable/install.html>, 不过要修改成python3

``` bash
sudo apt-get install build-essential python3-dev python3-setuptools python3-numpy python3-scipy libatlas-dev libatlas3gf-base
sudo update-alternatives --set libblas.so.3 /usr/lib/atlas-base/atlas/libblas.so.3
sudo update-alternatives --set liblapack.so.3 /usr/lib/atlas-base/atlas/liblapack.so.3
sudo apt-get install gfortran

sudo apt-get install git, 并配置好git
mkdir -p ~/local/src
cd ~/local/src
git clone git@github.com:scikit-learn/scikit-learn.git
cd scikit-learn
python setup.py install --user #开始编译
make PYTHON=python3 NOSETESTS=nosetests3 #或者使用make编译
nosetests3 -v sklearn #单元测试，可以在任何位置运行，不一定要在源码目录里
```

这里主要的坑是make, 刚开始我直接用 `make`, 失败，因为它默认是去找Python 2.7的 python.h 来编译，而我没有安装 python-dev, 只是安装了python3-dev，所以会编译失败。

我给 Scikit-Learn 的邮件组发了封邮件，不久得到了回复，要在make 后面加上 `PYTHON=python3`，这次编译成功了，不过到单元测试时说找不到`nosetests`命令，当然找不到了，因为前面安装的是python3-nose而不是python-nose，于是我猜测了一把，用 `make PYTHON=python3 NOSETESTS=nosetests3`试试, 果然可以！
