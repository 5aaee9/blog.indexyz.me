---
title: 将 Soyoustart 的 Arm 机器转换为 Arch Linux
date: '2018-10-01T11:39:00+08:00'
categories:
- Linux
---
Arch Linux 大法好!


<!--more-->

之前入了 Soyoustart 的 ARM 独立服务器, 官方只提供了 Ubuntu 和 Debian 的系统镜像, 但是这些系统下的软件包有很多缺失的(比如 `fish`), 这时候就想将系统装成 archlinuxarm 来快速滚包 (雾)

把安装过程写成了一个脚本, 公布在 gist

<script src="https://gist.github.com/Indexyz/c95d1b08df7cffa68b98a5e758c89cde.js"></script>

安装完成之后使用 账号 `alarm` 密码 `alarm`, 进行登陆并使用 `su` + root 密码 `root` 切换到 root 权限

> 请在安装完成之后即使修改 alarm 和 root 的密码
