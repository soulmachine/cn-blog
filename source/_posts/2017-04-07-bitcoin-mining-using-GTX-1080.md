---
title: 使用显卡挖比特币教程
date: 2017-04-08 00:23:55
tags: 比特币
---
操作系统: Ubuntu 16.04，显卡 GTX 1080

## 1. 安装显卡驱动和CUDA

```bash
apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64 /" | sudo tee /etc/apt/sources.list.d/cuda.list
sudo apt-get update
sudo apt-get -y install cuda-drivers cuda
```

## 2. 编译安装 ccMiner

```bash
git clone --recursive git@github.com:tpruvot/ccminer.git
cd ccminer
git checkout --track origin/linux
sudo apt install automake libcurl4-openssl-dev
```

根据显卡修改 `Makefile.am` (<https://github.com/tpruvot/ccminer/wiki/Compatibility>), 比如GTX 1080 则用

    nvcc_ARCH = -gencode=arch=compute_61,code=\"sm_61,compute_61\"

开始编译，

```bash
./autogen.sh
./configure
./build.sh
```

编译完成后会在当前目录生成一个 `ccminer` 可执行文件

<!-- more -->

## 3. 连接矿池开始挖矿

选一个矿池，注册好账号。<https://bitcoinchain.com/pools> 这里可以看到各个矿池的算力，我们选择最大的 AntPool吧。

    ./ccminer -o stratum+tcp://stratum.antpool.com:3333 -u soulmachine.btc01 -p soul123456

`-o` 是矿池服务器地址, `-u`的格式是 `UserId.WorkerId`, `UserId`必须是你注册网站时的用户名，`WorkerId`随便填，`-p`表示密码，随便填即可。还有个参数, `--algo`表示算法，可以不填，不填的时候默认为`auto`，表示自动选择哈希算法。
