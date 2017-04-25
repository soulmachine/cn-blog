---
title: 使用显卡挖ZCash教程
date: 2017-04-25 00:23:55
tags: ZCash
---
操作系统: Ubuntu 16.04，显卡 GTX 1080

## 1. 安装显卡驱动和CUDA

```bash
apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64 /" | sudo tee /etc/apt/sources.list.d/cuda.list
sudo apt-get update
sudo apt-get -y install cuda-drivers cuda
```

## 2. 安装 ewbf-miner

到这里下载 https://github.com/nanopool/ewbf-miner/releases

<!-- more -->

## 3. 连接矿池开始挖矿

在这里可以看到全球各大矿池的算力大小: <https://explorer.zcha.in/statistics/miners>

连接 f2pool,

    ./miner --server zec.f2pool.com --user t1P4fnXWE3XibaZEEt99y1K7A3qBhg2rVnw.asusgtx1080 --pass z --port 3357

连接flypool,

    ./miner --server us1-zcash.flypool.org --user t1P4fnXWE3XibaZEEt99y1K7A3qBhg2rVnw.asusgtx1080 --pass z --port 3333

我对比了一下第一大的flypool和第二大的f2pool, 分别挖了一天，感觉 flypool一天大概能挖0.08 ZEC, f2pool 一天能得到 0.12 ZEC，相差好几倍，所以我目前选择 f2pool。

