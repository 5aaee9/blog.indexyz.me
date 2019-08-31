---
title: 使用 proxychains 加速国外资源访问
tags: 
    - Linux
    - Proxy
categories:
    - Proxy
date: 2017-08-09 18:30:13
updated: 2017-09-01 18:11:38
---
众所周知, 我国访问国外资源的时候网络可能有些问题, 最近我在阿里云的服务器上 `clone` `GitHub` 的项目的时候 速度十分之慢 (10+KB/s) 因此搜索了一下 发现了 `proxychains` 这个神器

<!-- more --> 

> 本文不讨论如何跨越防火长城

## 安装

```bash
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng
./configure
make && make install
cp ./src/proxychains.conf /etc/proxychians.conf
cd .. 
rm -rf proxychains-ng
```

## 配置
> 使用这个配置的话是默认走纸飞机的

配置文件位于 `/etc/proxychains.conf` 使用 `nano` 或者 `vim` 打开
```
strict_chain
proxy_dns 
remote_dns_subnet 224
tcp_read_time_out 15000
tcp_connect_time_out 8000
localnet 127.0.0.0/255.0.0.0
quiet_mode

[ProxyList]
socks5  127.0.0.1 1080
```

## 使用
```bash
proxychains4 -q `Command`
```
请把 `Command` 替换为你的命令

### 已知问题
在某些程序下不生效 我在用 `yaourt` 构建 `gdrive` 的时候遇到了这个问题

解决方法未知 因为这个方法是使用依赖库的方法注入 所以很多时候都有这个问题 
(`steamcmd` 貌似也这样)
