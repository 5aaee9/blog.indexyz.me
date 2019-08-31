---
title: Hyper的使用教程以及VPN的搭建
tags: 
    - Linux
    - Docker
categories:
    - Linux
date: 2017-08-09 18:09:02
updated: 2017-09-01 18:15:21
thumbnail: https://publish.indexyz.me/images/2017/12/10/hypersh.png
---
> Hyper_是世界上第一家 Container-native 的 Docker 云服务. 
它的核心是底层的HyperContainer虚拟化容器技术, 以及Hypernetes多租户的 Kubernetes 
系统. 而使用方式与传统 VPS 也有很大不同.

<!--more-->

## Hyper_
最近在逛V2EX的时候看见了Hyper_的广告
于是就注册了来了一发, 感觉使用体验还不错
如果你不介意的话, 使用我的邀请码注册, 可获得10刀奖励
[Hyper_注册链接](https://console.hyper.sh/register/invite/xMlMNKnr87Hp9pMxrKpQhKo3MJpE33FJ)


本文以在Hyper上安装CentOS容器和安装OpenVPN为例, 简单介绍下如何使用Hyper_
## 安装和配置Hyper客户端
```bash
wget https://hyper-install.s3.amazonaws.com/hyper-linux-x86_64.tar.gz
tar xzf hyper-linux-x86_64.tar.gz
chmod +x hyper
./hyper config
```
## 基础教程
### Float IP
创建一个新的Float IP
> Tips: 
> 如果IP创建完成, 没用满一个月, 也是算你使用了一个月
> 官方回应: It's a feature.

这里的209.xxx.xxx.xxx就是分配到的IP了

```bash
# ./hyper fip allocate 1
209.xxx.xxx.xxx
```
绑定IP到名为centos的容器上
```bash
./hyper fip attach 209.xxx.xxx.xxx centos
```

### Shadowsock安装
```bash
./hyper run --size s1 -d --name shadowsocks -p 8989 
oddrationale/docker-shadowsocks -s 0.0.0.0 -p 8989 -k Indexyz -m aes-256-cfb
```
为容器绑定IP
```bash
./hyper fip attach 209.xxx.xxx.xxx shadowsocks
```
这段命令创建了一个名为`shadowsocks`的容器, 
使用的映像来自于`oddrationale/docker-shadowsocks` 
同时绑定了`209.xxx.xxx.xxx`作为容器IP
密码为 `Indexyz`
端口为 `8989`
加密方式 `aes-256-cfb`

### OpenVPN的安装
```bash
./hyper run --size s2 -p 8080:8080 -p 22:22 --name=centos-ssh -i -t 
centos:centos6 /bin/bash
```
使用完这个会打开一个bash进程
在终端进行以下操作打开一个ssh链接
```bash
yum install passwd openssl openssh-server -y
sed -i "s/UsePAM.*/UsePAM no/g" /etc/ssh/sshd_config
service sshd start
passwd root                                           # 修改Root密码
```
如果一不小心断开了容器, 请使用以下命令重新启动容器
```bash
./hyper start -i {容器ID}
```
使用ssh链接到服务器
然后参考[OpenVPN安装和面板的对接](https://blog.iinde.xyz/index.php/archives/32/)搭建OpenVPN相关服务
> Tips:
> 使用GCC编译Mproxy可能出错, 使用别的CentOS6编译出来的Mproxy即可解决
