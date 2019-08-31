---
title: 网络重装 CentOS
tags: 
    - Linux
    - CentOS
categories:
    - Linux
date: 2018-03-24 23:21:11
updated: 2018-03-24 23:21:11
thumbnail: https://publish.indexyz.me/images/2018/03/24/centos-logo.png
---
`Debian` 系的系统的网络重装脚本已经很多了 比如 [这个脚本](https://moeclub.org/2017/03/25/82/)

但是 `RedHat` 系列的系统却机没有网络安装脚本, 当需要调整分区或者是安装一个纯净的系统的时候就遇到了困难

但是 `RedHat` 系的系统也是可以从网络重装的 下面就介绍一种网络重装的方法

<!-- more -->

本文主体部分主要是重装 CentOS 6, 会在最后列出重装 `CentOS 7` 和 `CentOS 6` 的区别 

# CentOS 6 (GRUB 1)

首先 先获取机器的 IP地址，网关地址以及子网掩码

```bash
ip route | grep default # 获取网关地址
ip a s eth0             # 获取 IP 地址 和 子网大小
```

然后下载 `vmlinuz` 和 `initrd.img` 到 `/boot`

```bash
cd /boot
wget http://mirror.centos.org/centos/6/os/x86_64/isolinux/vmlinuz
wget http://mirror.centos.org/centos/6/os/x86_64/isolinux/initrd.img
```

编辑 `/etc/grub.conf` 在第一个 `title`标签 前面加入

```
title net
    root (hd0,0)
    kernel /boot/vmlinuz vnc vncpassword=12345678 headless ip=IPADDRES netmask=NETMASK gateway=GATEWAY dns=8.8.8.8 ksdevice=eth0 method=http://mirror.centos.org/centos/6/os/x86_64/ lang=en_US keymap=us
    initrd /boot/initrd.img
```

> 注意 `root (hd0,0)` 的内容可能有所不同 比如在 OVH 独服 上是 `root (hd0,1)`具体按照下面那个 `title` 中的 `root`位置

> 请将上面的 `IPADDRES`, `GATEWAY`, `NETMASK` 分别换成你机器的 IP 地址, 网关地址, 子网掩码

然后重启就可以通过 `IP地址:1` 的端口来连接到 VNC 进行全新安装模式了

# CentOS 7 (GRUB2)

首先 先获取机器的 IP地址，网关地址以及子网掩码

```bash
ip route | grep default # 获取网关地址
ip a s eth0             # 获取 IP 地址 和 子网大小
```

然后下载 `vmlinuz` 和 `initrd.img` 到 `/boot`

```bash
cd /boot
wget http://mirror.centos.org/centos/6/os/x86_64/isolinux/vmlinuz
wget http://mirror.centos.org/centos/6/os/x86_64/isolinux/initrd.img
```

然后使用如下指令来获取硬盘分区的 UUID

```bash
cat /etc/grub2.cfg | grep search | grep set=root
```

将会有以下的输出

```bash
search --no-floppy --fs-uuid --set=root 8c1015ef-fed5-4e77-bca0-478070c89df7
```

这里的 `8c1015ef-fed5-4e77-bca0-478070c89df7` 就是磁盘分区的 UUID

接下来修改 `/etc/grub.d/40_custom` 在文件末尾加入

```
menuentry 'net' {
  load_video
  set gfxpayload=keep
  insmod gzio
  insmod part_msdos
  insmod ext2
  set root='hd0,msdos1'
  if [ x$feature_platform_search_hint = xy ]; then
    search --no-floppy --fs-uuid --set=root --hint='hd0,msdos1' PARTUUID
  else
    search --no-floppy --fs-uuid --set=root PARTUUID
  fi
  linux16 /vmlinuz inst.vnc inst.vncpassword=12345678 inst.headless ip=IPADDRES::GATEWAY:NETMASK::eth0:none nameserver=8.8.8.8 inst.repo=http://mirror.centos.org/centos/6/os/x86_64/ inst.lang=en_US inst.keymap=us
  initrd16 /initrd.img
}
```

> 请将上面的 `PARTUUID`, `IPADDRES`, `GATEWAY`, `NETMASK` 分别换成你机器的 分区 UUID, IP 地址, 网关地址, 子网掩码

接下来使用以下命令来重新生成 grub 引导文件以及在下次启动的时候进入网络安装模式

```
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-reboot net
```

然后重启就可以通过 `IP地址:1` 的端口来连接到 VNC 进行全新安装模式了

# 常见问题

## 我在修改 grub 的时候该怎么填 `initrd.img` 和 `vmlinuz` 的地址

首先 查看是否有挂载在 `/boot` 下的分区

```bash
df -h | grep /boot
```

 如果有返回则代表应该直接以 `/initrd.img` 开头 如果没有的话以 `/boot/initrd.img` 开头

## CentOS 7 和 CentOS 6 在写参数时的不同

CentOS 7 的 格式如下

```
/boot/vmlinuz inst.vnc inst.vncpassword=12345678 inst.headless ip=IPADDRES::GATEWAY:NATMASK::eth0:none nameserver=8.8.8.8 inst.repo=http://mirror.centos.org/centos/7.4.1708/os/x86_64/ inst.lang=en_US
```

而 CentOS 6 的格式如下

```
/boot/vmlinuz vnc vncpassword=12345678 headless ip=IPADDRES netmask=NETMASK gateway=GATEWAY dns=8.8.8.8 ksdevice=eth0 method=http://mirror.centos.org/centos/6/os/x86_64/ lang=en_US keymap=us
```

## VNC 的默认密码是啥 如何修改

`vnc` 的密码是在 `vmlinuz` 那段定义的 在上面的几段配置文件中都为 `12345678`

可以通过 修改 `vncpassword`和 `inst.vncpassword` 的方式来修改密码 

> 密码必须长于 8 位
