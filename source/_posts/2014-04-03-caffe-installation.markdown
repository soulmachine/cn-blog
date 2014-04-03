---
layout: post
title: "Caffe 安装配置(CentOS + 无GPU)"
date: 2014-04-03 16:05
comments: true
categories: Deep-Learning
---

**环境**: CentOS 6.4

由于我的CentOS服务器上没有Nvidia的显卡，不过 caffe 是可以在CPU模式下进行train和predict的，因此我尝试了在没有GPU的情况下把caffe跑起来。

主要参考官网的文档，[Installation](http://caffe.berkeleyvision.org/installation.html)。

安装 Caffe 前需要安装以下库：

**Prerequisites**

* CUDA (5.0 or 5.5)
* Boost
* MKL (but see the boost-eigen branch for a boost/Eigen3 port)
* OpenCV
* glog, gflags, protobuf, leveldb, snappy, hdf5
* For the Python wrapper: python, numpy (>= 1.7 preferred), and boost_python
* For the Matlab wrapper: Matlab with mex

##1. 安装CUDA

	wget http://developer.download.nvidia.com/compute/cuda/repos/rhel6/x86_64/cuda-repo-rhel6-5.5-0.x86_64.rpm
	sudo rpm -Uvh libgcc-4.4.7-4.el6.x86_64.rpm
	yum search cuda
	sudo yum install cuda

或者

    wget http://developer.download.nvidia.com/compute/cuda/5_5/rel/installers/cuda_5.5.22_linux_64.run
    sudo ./cuda_5.5.22_linux_64.run

##2. 安装Boost

	sudo yum install boost-devel

<-- more -->

##3. 安装MKL
MKL是Intel的商业软件，性能很高，也卖的很贵。还好可以申请非商业版，去这里 <https://registrationcenter.intel.com/RegCenter/NComForm.aspx?ProductID=1461&pass=yes> 申请，申请成功之后你会得到一个序列号以及下载地址，下载完并解压， 执行`sudo ./install.sh` ， 之后按提示安装就好了，这个安装特别简单。

##4. 安装OpenCV

    sudo yum install opencv-devel

##5. 安装其他库

    wget https://google-glog.googlecode.com/files/glog-0.3.3.tar.gz
    tar zxf glog-0.3.3.tar.gz
    cd glog-0.3.3
    ./configure
    make
    sudo make install

    sudo yum install gflags-devel protobuf-devel leveldb-devel snappy-devel hdf5-devel

##6. 配置OpenCV环境
Caffe作者默认你已经配置好了OpenCV环境，文档里没有说这一步。好在有人已经写好了配置OpenCV的脚本，<https://github.com/jayrambhia/Install-OpenCV> ，直接拿来用。

    git clone https://github.com/jayrambhia/Install-OpenCV
    cd Install-OpenCV/RedHat
    sudo ./opencv_latest.sh

如果脚本运行失败，则详细阅读`RetHat/opencv_install.sh`的代码，然后手工敲入命令进行安装。

    mkdir OpenCV
    cd OpenCV
    wget -O opencv-2.4.7.tar.gz http://sourceforge.net/projects/opencvlibrary/files/opencv-unix/2.4.7/opencv-2.4.7.tar.gz/download
    tar -zxf opencv-2.4.7.tar.gz
    cd opencv-2.4.7
    sed  -i '/string(MD5/d' cmake/cl2cpp.cmake
    mkdir build
    cd build
    cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
    make -j 4
    sudo make install
    sudo sh -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
    sudo ldconfig

##7. 编译

    cp Makefile.config.example Makefile.config
    make all

##8. 运行MNIST例子
主要参考官网的 [Training MNIST with Caffe](http://caffe.berkeleyvision.org/mnist.html)

###8.1 下载数据集

    cd $CAFFE_ROOT/data/mnist
    ./get_mnist.sh
    cd $CAFFE_ROOT/examples/lenet
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64
    ./create_mnist.sh
    
运行完上述命令后，应该会得到两个数据集，`mnist-train-leveldb/`, 和 `mnist-test-leveldb/`.

最终的model，

###8.2 切换到CPU模式
由于服务器没有安装显卡，只能使用CPU训练。切换到CPU模式非常简单，只需要在`lenet_solver.prototxt`中修改一行：

> \# solver mode: 0 for CPU and 1 for GP   
> solver_mode: 0

###8.3 开始训练

    ./train_lenet.sh

经过一段时间运行，训练完成！最终的model，会存为一个二进制的protobuf文件，`lenet_iter_10000`. 


##参考资料

1. [CNN之Caffe配置](http://www.cnblogs.com/alfredtofu/p/3577241.html)


注意， CUDA 5.5 不支持 Visual Studio 2013，参考 [CUDA 5.5 release notes](http://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html#windows-5-5)