---
title: 在Linux系统上使用ASF挂卡
tags: 
    - Linux
    - Usage
    - Steam
categories:
    - Steam
date: 2017-10-04 07:32:11
updated: 2017-10-05 13:30:24
thumbnail: https://publish.indexyz.me/images/2017/12/10/steam.png
---
Steam的卡片可是可以回本的 但是 `Idle Master` 这些挂机需要使用 `Windows` 的虚拟机 但是 Windows 的机器并不好搞 然后我就发现了 `Archi's Steam Farm` 简称 `ASF`, 这位dalao模拟了 Steam 客户端的操作进行挂卡 虽然是 `Dot Net` 写的 但是可以在 Linux 上使用 `Mono` 运行 最新的 ASF 甚至已经可以脱离 Mono 运行了 这篇文章就是记录了怎么在Linux上运行ASF 2, ASF 3 有可能在未来开一篇文章单独讲

<!-- more -->

本文的系统环境是 `CentOS 7` 其他发行版可能需要自己更换部分代码 (比如包管理安装包什么的)
# 准备工作

环境的部署
```bash
yum -y install epel-release
yum -y install yum-utils tmux wget unzip 
rpm --import "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"   
yum-config-manager --add-repo http://download.mono-project.com/repo/centos/
yum -y install mono-complete
```
这样就部署了 `Mono` 
输入以下指令来测试
```bash
mono -V
```

# 获取 ASF
```bash
mkdir steam
cd steam
wget https://github.com/JustArchi/ArchiSteamFarm/releases/download/2.3.2.4/ASF.zip
unzip ASF.zip
```
这样就获得了 ASF2 的最后一个 `Release`

# 配置
## 全局配置文件
可以在 `config/ASF.json` 中配置全局配置 默认配置如下
```json
{
    "AutoRestart": true,
    "AutoUpdates": true,
    "Blacklist": [],
    "ConnectionTimeout": 60,
    "CurrentCulture": null,
    "Debug": false,
    "FarmingDelay": 15,
    "GiftsLimiterDelay": 1,
    "Headless": false,
    "IdleFarmingPeriod": 3,
    "InventoryLimiterDelay": 3,
    "LoginLimiterDelay": 10,
    "MaxFarmingTime": 10,
    "MaxTradeHoldDuration": 15,
    "OptimizationMode": 0,
    "Statistics": true,
    "SteamOwnerID": 0,
    "SteamProtocol": 6,
    "UpdateChannel": 1,
    "WCFBinding": 0,
    "WCFHost": "127.0.0.1",
    "WCFPort": 1242
}
```

配置项的意义可以在 [官方Wiki](https://github.com/JustArchi/ArchiSteamFarm/wiki/Configuration) 中看到
## 个人配置文件
可以直接编辑 `minimal.json` 来快速配置
> 注意! minimal.jsom 和 example.json 这两个文件将不会被读取
```json
{
    "Enabled": true,
    "SteamLogin": "{{ steam 用户名 }}",
    "SteamPassword": "{{ steam 密码 }}"
}
```
可以增加几条实用的选项
- FarmOffline: true  # 在离线模式下挂卡
- DismissInventoryNotifications: false  # 显示挂卡获得卡的邮件


# 开始挂卡
因为 Linux 的运行是断开 SSH 直接杀掉进程的 所以我们需要用 tmux 来进行会话管理
```
tmux
mono ASF.exe
```
这样就在 tmux 中运行了 ASF 下次运行只需要 `tmux a` 就可以进入这个 session 了
