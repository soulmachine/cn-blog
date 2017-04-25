---
title: 使用显卡挖以太币教程
date: 2017-04-08 01:04:09
tags: Ethereum
---
操作系统: Ubuntu 16.04，显卡 GTX 1080

## 1. 安装显卡驱动和CUDA

```bash
apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64 /" | sudo tee /etc/apt/sources.list.d/cuda.list
sudo apt-get update
sudo apt-get -y install cuda-drivers cuda
```

## 2. 编译安装 Genoil/cpp-ethereum

```bash
sudo apt -y install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt -y update
sudo apt -y install git cmake libcryptopp-dev libleveldb-dev libjsoncpp-dev libjsonrpccpp-dev libboost-all-dev libgmp-dev libreadline-dev libcurl4-gnutls-dev ocl-icd-libopencl1 opencl-headers mesa-common-dev libmicrohttpd-dev build-essential
git clone git@github.com:Genoil/cpp-ethereum.git
cd cpp-ethereum/
mkdir build
cd build
cmake -DBUNDLE=cudaminer ..
make -j8
```

编译完成后会在当前目录的子目录`ethminer`下生成一个 `ethminer` 可执行文件。

<!-- more -->

## 3. 连接矿池开始挖矿

在这里可以看到全球各大矿池的算力大小: <https://etherchain.org/statistics/miners>

选一个矿池，以<https://ethermine.org/> 这个矿池为例，

    export GPU_FORCE_64BIT_PTR=0
    export GPU_MAX_HEAP_SIZE=100
    export GPU_USE_SYNC_OBJECTS=1
    export GPU_MAX_ALLOC_PERCENT=100
    export GPU_SINGLE_ALLOC_PERCENT=100

如果你的显卡只有2G显存，需要设置以上环境变量，我的 GTX 1080 有8G内存，就不需要设置了。

    ./ethminer --farm-recheck 2000 -U -S us2.ethermine.org:4444 -FS us1.ethermine.org:4444 -O 0xba90FF2fA9016B3883799D150fB15DB5b4894f8b.eth01

`--farm-recheck`大致是重新检查某些东西的时间间隔，毫秒为单位，`-U`指的是用GPU，且用的是CUDA而不是OpenCL，`-S`指定stratum服务器，`-FS`指定的备份服务器(这里用us2做主服务器，us1作备份服务器的原因是，us1是在美国东部，us2在美国西部，而我的机器在西部，离us2近一些)，`-O` 指定自己的钱包地址(我是用的CoinBase的在线)，`.`后面是RigName, 随便填。

我搜到的第二大的矿池是 <http://ethpool.org/>，连接这个矿池的命令跟ethermine.org一模一样，只是地址变了，

    ./ethminer --farm-recheck 2000 -U -S us2.ethpool.org:3333 -FS us1.ethpool.org:3333 -O 0xba90FF2fA9016B3883799D150fB15DB5b4894f8b.eth01


## 4. Genoil 和 Claymore 的比较

Claymore 是另一款挖矿软件，经过我亲自测试，二者的速度基本一样，GTX 1080 都在在 20MH/s 左右，不过 Claymore可以在不影响以太币挖矿速度的情况下，还可以同时挖 Decred/Siacoin/Lbry/Pascal 之一，于是我现在用的是 Claymore这个挖矿程序，命令行如下，

举个例子，同时挖ETH和Decred，

    ./ethdcrminer64 -epool us2.ethermine.org:4444 -ewal 0xba90FF2fA9016B3883799D150fB15DB5b4894f8b.eth01 -epsw x -dpool pasc-us-west1.nanopool.org:15555 -dwal Dsab2dnwdTTpibkUr9VREdhLNytdnCv9nGv -dpsw x

或者同时挖ETH和SiaCoin,

    ./ethdcrminer64 -epool us2.ethermine.org:4444 -ewal 0xba90FF2fA9016B3883799D150fB15DB5b4894f8b.eth01 -epsw x -dpool stratum+tcp://siamining.com:7777 -dwal a808cdb0061d81418f6f146775dad4e3590eba207f285ad67b061a2ec01f6402960e02e36e7a.sia01 -dcoin sia

不过要注意，在 Ethereum-only 模式下，会收取 1% 的费用，在 Dual模式下，会收取 2%的费用，当然不会直接向你收费，它每个小时大概会有 36 到 72 秒为作者挖矿，这样间接达到了收费的目的。

不要试图同时运行`ethminer`和`ethdcrminer64`，这样的话它们的速度同时会降为原来的一半，总的速度还是跟单个一样。

Claymore 比较方便的是可以同时挖ETH和另一种币，大家可以同时看到我的ETH挖矿速度<https://ethermine.org/miners/ba90FF2fA9016B3883799D150fB15DB5b4894f8b>和SiaCoin挖矿速度 <https://siamining.com/addresses/a808cdb0061d81418f6f146775dad4e3590eba207f285ad67b061a2ec01f6402960e02e36e7a>
