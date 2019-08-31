---
title: MultiMC 的简单使用教程
tags: 
    - Usage
    - Game
    - Minecraft
categories:
    - Minecraft
date: 2017-08-09 18:24:07
updated: 2017-09-01 18:13:16
thumbnail: https://publish.indexyz.me/images/2017/12/10/multimc.png
---
> MultiMC 是一个免费, 开源的[Minecraft](https://minecraft.net/zh-hans/)启动器. 它最大的特点便是允许用户定义自己的整合包(在MMC中这个概念为 `instances`) 并且可以方便的管理他们

<!--more-->

## 基本介绍
首先来看看官网的features
```bash
Manage multiple instances of Minecraft at once
Start Minecraft with a custom resolution
Change Java's runtime options (including memory options)
Shows Minecraft's console output in a color-coded window
Kill Minecraft easily if it crashes/freezes
Custom icons and groups for instances
Forge integration (automatic installation, version downloads, mod management)
Minecraft world management
Import and export Minecraft instances to share them with anyone
Supports every version of Minecraft that the vanilla launcher does
```
*简单的说就是个大号的启动器*
### 下载使用
在 [官网](https://multimc.org/) 中可以找到各个系统的最新下载地址
当然你觉得 无聊和没事找事 什么的 你可以在 
[GitHub](https://github.com/MultiMC/MultiMC5) 上找到最新的源代码并且自己编译安装
> Note: MultiMC 的官方版是强制要求正版验证的 盗版的同学请去下修改版的MMC

当你下载完成之后解压缩后
Windows 用户 请直接打开 MultiMC.exe
OS X 用户请直接安装MultiMC.app
Linux 用户如果是选择包安装可以在系统的Dock什么的位置找到它 
不是包安装请直接./MulitMC
> 下文都在Windows平台上完成
## 第一次见面
> 部分用户在第一次启动的时候会没有中文 这种情况出现在MultiMC的旧版本 
解决方法也很简单 重启一次启动器就好了

现在第一次启动的时候有个安装界面 一路下一步就好了 请记得在第一步选择简体中文
![QQ截图20170121115953.png][1]
## 在开始直接我们还要...
### 添加账号
在右上角有个配置文件 添加自己的Minecraft账号 (`配置文件 -> 管理账号 -> 添加`)

![`$60H1EXDEDMA(IVM_@3DSH.png][2]

上图是我添加完成之后的效果

### 设置代理服务器(选做)
![QQ截图20170121120524.png][3]

因为Minecraft的源 `亚马逊S3` 在国内速度简直爆炸 
所以我们最好通过代理服务器来下载游戏资源
`设置 -> 代理` 修改代理为你的服务器然后确定 (上图是纸飞机的默认端口)
> 设置完代理请重启启动器来使代理生效


## 让我们开始吧
### 导入一个整合包
`新建 -> 导入Mod整合包`选择你的整合包地址或者选择一个本地的整合包 
按下OK之后便直接导入到了MMC当中了

![Z~`IYT{14UW)4M~~I4ZYA4.png][4]

> 在第一次启动的时候获取相关文件需要较长时间 请等待 如果速度较慢建议设置代理

### 创建自己的第一个实例
在 `新建 -> 纯净Minecraft` 中选择一个版本 OK之后就会 
添加这个版本的Minecraft到你的库中

### 复制实例
在实例上右键 选择`复制实例` 可以获得当前这个实例的子实例(几乎一样)

### 创建Forge实例
在新建了的 纯净Minecraft 中(你也可以选择复制一份)
右击 实例 选择 `修改实例`
然后按下 `版本 -> 安装Forge` 选择一个你需要的Forge版本点击OK就可以安装上Forge了
![QQ截图20170121123000.png][5]


  [1]: https://publish.indexyz.me/images/2017/01/974912626.png
  [2]: https://publish.indexyz.me/images/2017/01/1111769254.png
  [3]: https://publish.indexyz.me/images/2017/01/4198598707.png
  [4]: https://publish.indexyz.me/images/2017/01/4269960964.png
  [5]: https://publish.indexyz.me/images/2017/01/4219265299.png

