---
title: 在 Linux 上使用 redsocks 进行全局 socks5
tags: 
    - Linux
    - Proxy
    - iptables    
categories:
    - Linux
date: 2017-10-22 15:40:15
updated: 2017-10-28 07:45:51
thumbnail: https://publish.indexyz.me/images/2017/12/10/paperplane.png
---
在中国大陆由于一些众所周知以及网络环境有时候会比较差的原因, 我们可能会需要使用代理服务器来进行转发流量. 但是有些软件不支持代理, 如何让它走代理呢? 那就要介绍下 `redsocks` 了.

<!-- more -->

# 安装
redsocks 的安装比较方便 从 GitHub 检出代码然后编译就好了
```bash
git clone https://github.com/darkk/redsocks
cd redsocks
make
```
## 配置
默认我们认为本地 `socks` 代理在 `127.0.0.1:1080`
将配置文件配为下面这样
```conf
// $ cat redsocks.conf 
base {
    log_debug = off;
    log_info = off;
    log = "syslog:daemon";
    // 改成 on 使用 daemon 运行
    daemon = on;
    user = redsocks;
    group = redsocks;
    redirector = iptables;
}

redsocks {
    local_ip = 127.0.0.1;
    local_port = 31338;
    
    ip = 127.0.0.1;
    port = 1080;

    type = socks5;
}
```

## 启动
```bash
./redsocks
```
就可以运行了

# 配置 iptables
```bash
# Transparent SOCKS proxy
# See: http://darkk.net.ru/redsocks/

*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:REDSOCKS - [0:0]

# Redirect all output through redsocks
-A OUTPUT -p tcp -j REDSOCKS

# Whitelist LANs and some other reserved addresses.
# https://en.wikipedia.org/wiki/Reserved_IP_addresses#Reserved_IPv4_addresses
-A REDSOCKS -d 0.0.0.0/8 -j RETURN
-A REDSOCKS -d 10.0.0.0/8 -j RETURN
-A REDSOCKS -d 127.0.0.0/8 -j RETURN
-A REDSOCKS -d 169.254.0.0/16 -j RETURN
-A REDSOCKS -d 172.16.0.0/12 -j RETURN
-A REDSOCKS -d 192.168.0.0/16 -j RETURN
-A REDSOCKS -d 224.0.0.0/4 -j RETURN
-A REDSOCKS -d 240.0.0.0/4 -j RETURN
#-A REDSOCKS -d { 代理服务端的地址 } -j RETURN
# shadowsocks server port
-A REDSOCKS -p tcp --dport { 代理服务端的端口 } -j RETURN

# Redirect everything else to redsocks port
-A REDSOCKS -p tcp -j REDIRECT --to-ports 31338

COMMIT
```

然后全部的 `TCP` 链接就会走代理了