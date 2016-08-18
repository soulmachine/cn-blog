---
title: 深度学习开发环境配置：Ubuntu1 6.04+Nvidia GTX 1080+CUDA 8.0
date: 2016-08-17 23:59:11
tags: 深度学习
categories: 深度学习
---

前提条件，已经安装好了 Ubuntu 16.04 操作系统， 见[安装 Windows 10 和 Ubuntu 16.04 双系统](http://cn.soulmachine.me/2016-08-14-dual-install-windows-ubuntu/)



## 1. 安装 Nvidia 驱动


```shell
sudo add-apt-repository -qy ppa:graphics-drivers/ppa
sudo apt-get -qy update
sudo apt-get -qy install nvidia-370
sudo apt-get -qy install mesa-common-dev
sudo apt-get -qy install freeglut3-dev
sudo reboot
```

注意，一般比较新的主板，默认是UEFI BIOS，默认启用了 Secure Boot，否则开机后登陆不进去。老主板没有这个问题。


## 2. 安装 CUDA 8.x

去 [CUDA 8.x 下载页面](https://developer.nvidia.com/cuda-release-candidate-download)，一定要下载 runfile 安装方式的安装包，参考资料里的好几篇都是选择这种方式，貌似 deb包有坑？

<!-- more -->

```shell
chmod u+x ./cuda_8.0.27_linux.run
sudo ./cuda_8.0.27_linux.run --tmpdir=/tmp
```

执行后会有一系列提示让你确认，第一个就是问你是否安装显卡驱动，由于前一步已经安装了显卡驱动，所以这里就不需要了，况且 runfile 自带的驱动版本不是最新的。 因此 `Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 361.77?` 这里选择 no。

```shell
Do you accept the previously read EULA?
accept/decline/quit: accept

Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 361.77?
(y)es/(n)o/(q)uit: n

Install the CUDA 8.0 Toolkit?
(y)es/(n)o/(q)uit: y

Enter Toolkit Location
 [ default is /usr/local/cuda-8.0 ]: 

Do you want to install a symbolic link at /usr/local/cuda?
(y)es/(n)o/(q)uit: y

Install the CUDA 8.0 Samples?
(y)es/(n)o/(q)uit: y

Enter CUDA Samples Location
 [ default is /home/programmer ]:
```

你以为你会成功安装吗？并不是，你一定会碰到一个错误，` Installation Failed. Using unsupported Compiler.` ，这是因为 Ubuntu 16.04 默认的 GCC 5.4 对于 CUDA 8.x来说过于新了，CUDA 安装脚本还不能识别新版本的 GCC。

看了一下安装日志，解决方案也很简单，加一个 `--override` 选项，

```shell
sudo ./cuda_8.0.27_linux.run --tmpdir=/tmp --override
```

这次可以成功了。

```
===========
= Summary =
===========

Driver:   Not Selected
Toolkit:  Installed in /usr/local/cuda-8.0
Samples:  Installed in /home/programmer, but missing recommended libraries

Please make sure that
 -   PATH includes /usr/local/cuda-8.0/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-8.0/lib64, or, add /usr/local/cuda-8.0/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run the uninstall script in /usr/local/cuda-8.0/bin

Please see CUDA_Installation_Guide_Linux.pdf in /usr/local/cuda-8.0/doc/pdf for detailed information on setting up CUDA.

***WARNING: Incomplete installation! This installation did not install the CUDA Driver. A driver of version at least 361.00 is required for CUDA 8.0 functionality to work.

To install the driver using this installer, run the following command, replacing <CudaInstaller> with the name of this run file:

    sudo <CudaInstaller>.run -silent -driver

Logfile is /tmp/cuda_install_6794.log

Signal caught, cleaning up
```

把以下两行加入到 `.bashrc`，

```shell
export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

## 安装补丁

```shell
chmod u+x ./cuda_8.0.27.1_linux.run
sudo ./cuda_8.0.27.1_linux.run
```


## 测试是否安装成功

最后再来测试一下CUDA，运行：

```shell
nvidia-smi
```

结果如下所示：

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 370.23                 Driver Version: 370.23                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 1080    Off  | 0000:05:00.0      On |                  N/A |
| 27%   29C    P8     9W / 180W |    515MiB /  8110MiB |      4%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|    0      4761    G   /usr/lib/xorg/Xorg                             259MiB |
|    0      5224    G   compiz                                         253MiB |
+-----------------------------------------------------------------------------+
```

再来试几个CUDA例子：

```shell
cd ~/NVIDIA_CUDA-8.0_Samples/1_Utilities/deviceQuery
make
```

执行 `./deviceQuery`，得到：

```
CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GTX 1080"
  CUDA Driver Version / Runtime Version          8.0 / 8.0
  CUDA Capability Major/Minor version number:    6.1
  Total amount of global memory:                 8110 MBytes (8504279040 bytes)
  (20) Multiprocessors, (128) CUDA Cores/MP:     2560 CUDA Cores
  GPU Max Clock rate:                            1734 MHz (1.73 GHz)
  Memory Clock rate:                             5005 Mhz
  Memory Bus Width:                              256-bit
  L2 Cache Size:                                 2097152 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
  Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 5 / 0

  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 8.0, CUDA Runtime Version = 8.0, NumDevs = 1, Device0 = GeForce GTX 1080
Result = PASS
```

再测试试一下nobody：

```shell
cd ~/NVIDIA_CUDA-8.0_Samples/5_Simulations/nbody/
make
```

执行：

```shell
./nbody -benchmark -numbodies=256000 -device=0
```

得到：

```
> Windowed mode
> Simulation data stored in video memory
> Single precision floating point simulation
> 1 Devices used for simulation
gpuDeviceInit() CUDA Device [0]: "GeForce GTX 1080
> Compute 6.1 CUDA device: [GeForce GTX 1080]
number of bodies = 256000
256000 bodies, total time for 10 iterations: 2364.286 ms
= 277.192 billion interactions per second
= 5543.830 single-precision GFLOP/s at 20 flops per interaction
```

至此，说明 CUDA 8.x 安装成功了。


## 参考资料

* [深度学习主机环境配置: Ubuntu16.04+Nvidia GTX 1080+CUDA8.0](http://www.52nlp.cn/%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E4%B8%BB%E6%9C%BA%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE-ubuntu-16-04-nvidia-gtx-1080-cuda-8)
* [Nvidia GTX 1080 on Ubuntu 16.04 for Deep Learning - Changjiang](http://yangcha.github.io/GTX-1080/)
* [Build Personal Deep Learning Rig: GTX 1080 + Ubuntu 16.04 + CUDA 8.0RC + CuDnn 7 + Tensorflow/Mxnet/Caffe/Darknet - by Guanghan Ning](http://guanghan.info/blog/en/my-works/building-our-personal-deep-learning-rig-gtx-1080-ubuntu-16-04-cuda-8-0rc-cudnn-7-tensorflowmxnetcaffedarknet/)
* [GeForce GTX 1080, CUDA 8.0, Ubuntu 16.04, Caffe](https://github.com/BVLC/caffe/wiki/GeForce-GTX-1080,---CUDA-8.0,---Ubuntu-16.04,---Caffe)
