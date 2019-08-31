---
title: 利用 Socat 单端口转发加速TCP链接
tags: 
    - Linux
    - Proxy
    - Network
categories:
    - Linux
date: 2017-08-09 18:17:07
updated: 2017-09-01 18:14:21
---
> 有童鞋说我的Minecraft服务器链接速度很慢 我是托管在国内的Azure上的 
既然这么说了 那我就拿 `Socat` 转发到阿里云上


<!--more-->


## 安装Socat 
### CentOS
```bash
yum install -y socat
```
### Ubuntu
```bash
apt-get update
apt-get install -y socat
```
## 使用
```bash
nohup socat TCP4-LISTEN:本地端口,reuseaddr,fork TCP4:远程地址:远程端口 >> 
/root/socat.log 2>&1 &
iptables -I INPUT -p tcp --dport 本地端口 -j ACCEPT
service iptables save
service iptables restart
```
