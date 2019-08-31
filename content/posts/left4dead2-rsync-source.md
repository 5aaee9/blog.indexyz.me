---
title: Left 4 Dead 2 服务器Rsync同步源
tags: 
    - Game
    - Left 4 Dead 2
categories:
    - Game
date: 2017-08-09 18:04:09
updated: 2017-09-01 18:16:06
thumbnail: https://publish.indexyz.me/images/2017/12/10/left4dead2.png
---
> 最近因为L4D2的某些原因(Downtown插件需要GLIBC2.15以上然而CentOS6无法提供), 
重装了服务器, 然而阿里云上的SteamCMD老是拉不下来端, 
于是我在国外的文件服务器上搭建了Rsync源来解决服务端问题


<!--more-->

## 使用方法
```bash
yum install rsync
// 链接密码root
rsync -avzP --delete root@rsync.indexs.lol::l4d2server serverfiles
```

### 另
```
// 阿里云安装时错误代码
Error! App '222860' state is 0x402 after update job.
CWorkThreadPool::~CWorkThreadPool: work complete queue not empty, 36 items 
discarded.
CWorkThreadPool::~CWorkThreadPool: work processing queue not empty: 22 items 
discarded.
// 求大佬分析
```
