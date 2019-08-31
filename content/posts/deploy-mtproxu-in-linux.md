---
title: 在 Linux 机器上部署 MTProxy
date: '2018-10-02T16:36:00+08:00'
categories:
- Linux
---
MTProxy 是 Telegram 专属的一种代理格式, 目前大部分平台上的 Telegram 客户端都提供了对此协议的支持


<!--more-->

## 为何要使用 MTProxy

1. 方便, MTProxy 可以方便的在各种 Telegram 客户端上部署, 方便使用
2. 安全, MTProxy 和客户端之间的通信都是经过了 aes256 加密的, 保证了数据安全

## 安装
MTProxy 有其[官方的实现][1]

不过这个实现比较消耗资源, 编译环境比较难以配置, 于是下文使用

[card]FreedomPrevails/JSMTProxy[/card]

### 安装 Node 环境
安装 nodejs 环境可以使用 nvm 来快速搭建 使用以下命令安装 nvm 以及最新的 nodejs 以及 pm2 守护
```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
export NVM_DIR="${XDG_CONFIG_HOME/:-$HOME/.}nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm

nvm install 10
npm i -g pm2
pm2 startup
```
### 安装 JSMTProxy
使用以下命令获取随机的 screct
```bash
head -c 16 /dev/urandom | xxd -ps
```
记录这串字符作为你的 token

使用以下命令来安装后端
```bash
git clone https://github.com/FreedomPrevails/JSMTProxy.git
```
编辑 `config.json` 将端口换为你需要的端口并且修改 screct

```bash
pm2 start -n MTProxy mtproxy.js
pm2 save
```
然后现在在 Telegram 上连入 Proxy 就可以了

  [1]: https://github.com/TelegramMessenger/MTProxy
