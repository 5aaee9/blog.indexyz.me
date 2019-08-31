---
title: 在Linux上安装TsunamiUDP进行快速传输
tags: 
    - Linux
    - Transfer File
categories:
    - Linux
date: 2017-08-09 17:56:17
updated: 2017-09-01 18:17:52
---
> TsunamiUDP 是一个专为网络加速诞生的小工具。 
思路很简单，使用TCP进行传输控制、UDP进行数据传输。


<!--more-->

# 安装
```bash
# 安装依赖
yum install autoconf automake gcc unzip
# 下载源代码编译
wget https://github.com/cheetahmobile/tsunami-udp/archive/1.8.1.zip
unzip 1.8.1.zip
cd tsunami-udp-1.8.1
./recompile.sh
make install
```
# 开启输出服务器
```bash
cd /data
tsunamid *
```

# 链接服务器获取数据
```bash
cd / data
tsunami
> connect {serverip}
Connected.
> get *
```
晒一张德国服务器和美国服务器互传的速度截图
![速度截图][1]


  [1]: https://publish.indexyz.me/images/2016/05/2418556794.png
