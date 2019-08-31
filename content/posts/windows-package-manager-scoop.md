---
title: Windows 下的包管理器 - Scoop
tags: 
    - Windows
    - Package Manager
categories:
    - Windows
date: 2017-09-01 18:19:33
updated: 2017-09-01 18:28:01
thumbnail: https://publish.indexyz.me/images/2017/12/10/window.png
---
提起包管理器肯定是想起了 `Linux` 下的 `yum`, `apt-get`, `pacman` 之流 但是其实Windows下面也是有包管理器的

<!-- more -->

# Scoop
GitHub: [链接](https://github.com/lukesampson/scoop)
官方网站: [链接](https://scoop.sh)

## 安装方法 
安装方法挺简单的 直接打开 PowerShell 然后输入
```powershell
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
```
确保你的PowerShell版本在3.0以上

## 安装软件
```powershell
scoop install nodejs
```
然后scoop就会去搜索nodejs并自动下载安装

### 打开拓展库开关
官方其实也提供了一些额外的包 例如 `Oracle Java` 甚至有提供 `jetbrains-toolbox`
要使用拓展库 你只需要
```powershell
scoop bucket add extras
```
> 确保你已经装了git 如果没有安装请 scoop install git
