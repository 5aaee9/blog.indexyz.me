---
title: 在Linux主机上扩展主机硬盘
tags: 
    - Linux
categories:
    - Linux
date: 2017-08-09 18:06:32
updated: 2017-09-01 18:15:38
thumbnail: https://publish.indexyz.me/images/2017/12/10/redhat.png
---
很多主机商的VPS中, 
在安装完系统之后所给的硬盘大小并不是标注的硬盘大小(而且不同系统有时候大小还不一样). 
在这种情况下就要自己扩充硬盘大小了.


<!--more-->

首先登陆到服务器
```bash
fdisk –l
```
```
[root@localhost ~]# fdisk -l

Disk /dev/sda: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000aea04

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    20971519     9972736   8e  Linux LVM

Disk /dev/sdb: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x789cab85

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb3              63    83875364    41937651   8e  Linux LVM

Disk /dev/mapper/centos-root: 9093 MB, 9093251072 bytes, 17760256 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 1073 MB, 1073741824 bytes, 2097152 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
这里的sdb3就是我们要操作的硬盘了, 我们要把它合并到/dev/centos/root

首先
```bash
fdisk /dev/sdb3
n
p
3
# 默认
# 默认
t
8e
w
```
这时候重启一次系统, 让分区表重新加载
```bash
pvcreate /dev/sdb3
# 如果出现错误则是服务商已经把这个硬盘划给了另外一个Group 加上-ff参数就好了
vgextend centos /dev/sdb3
vgdisplay
```

```
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               49.50 GiB
  PE Size               4.00 MiB
  Total PE              12672
  Alloc PE / Size       2424 / 9.47 GiB
  Free  PE / Size       10248 / 40.03 GiB
  VG UUID               #####
```
这时候
```bash
lvresize -L +40.03G /dev/centos/root
```
最后
```bash
resize2fs /dev/centos/root
```
如果出现以下提示的话
```
resize2fs 1.42.9 (28-Dec-2013)
resize2fs: Bad magic number in super-block while trying to open /dev/centos/root
Couldn't find valid filesystem superblock.
```
使用
```
xfs_growfs /dev/centos/root
```
