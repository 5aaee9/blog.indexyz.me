---
title: 在星际蜗牛主机上使用 LVM 安装 Arch Linux 以及常用软件
date: '2019-06-21T15:30:00+08:00'
categories:
- Linux
---
之前星际蜗牛矿难, 流出了一大堆矿难机器, 正好收过来做 NAS (绝对超值)


<!--more-->

# Intro
星际蜗牛的机器差不多是这样的配置
[Intel J1900][1]
4GB 不知名 DDR3 内存
16GB 垃圾板载 SSD（U 盘水平）
四个硬盘位
千兆网口

虽然这个配置不咋的, 但是拿来做 NAS 绰绰有余了, 接下来将讲述如何挂载上多枚硬盘并且作为 NAS 来使用

# 安装
## 安装 Arch Linux
虽然到手之后自带了一个 CentOS 不过我们不知道 root 密码也不知道里面有着什么鬼东西, 直接整个抹盘重装.

此处准备了一个存有 ArchLinux ISO 的 U盘并 Boot 到 LiveCD 模式
首先 fdisk -l 列出磁盘, 我内置的磁盘在 /dev/sda 下面

然后清空磁盘并分区
```bash
dd if=/dev/zero of=/dev/sda
#^C
cfdisk /dev/sda
```
我的分区如下

- 300M EFI System 分区 /dev/sda1
- 剩余空间 EXT4 /dev/sda2

使用以下命令挂载系统分区
```bash
mount /dev/sda2 /mnt
cd /mnt
mkdir -p boot
mount /dev/sda1 boot
```

然后插上网线就可以开始安装 ArchLinux 了
```bash
pacstrap -i /mnt base base-devel
```

> 该过程会联网下载 ArchLinux 相关软件包, 可能会占用较长时间, 泡杯咖啡然后坐和放宽

完成系统安装和安装引导
```bash
genfstab -U /mnt > /mnt/etc/fstab
bootctl --path=/boot install

cd /boot/loader/entires
nano default.conf
```
编写 启动项 这里给上我的来参考
```conf
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=PARTUUID=YOUR_PART_ROOT_PART_UUID rw
```
> YOUR_PART_ROOT_PART_UUID 可以通过 blkid -s PARTUUID -o value /dev/sda2 来获取

接下来执行
```bash
# 添加用户
useradd -m indexyz
# 修改密码
passwd
exit
reboot
# 拔掉 U 盘
```

## 配置 Arch Linux
上一个步骤完成了 Arch Linux 的安装 接下来就该配置 Arch Linux 了

首先先在 /etc/systemd/network 下建立 wired.network 文件

> systemd 全家桶大法好

内容为
```conf
[Match]
Name=enp*

[Network]
DHCP=ipv4
```

然后执行
```bash
systemctl enable --now systemd-networkd
systemctl enable --now systemd-resolved
```

就为全部的有线网口启用了 DHCP 了

> 如果要自己配置静态地址的话 参考 [Arch Linux WIKI - systemd-network][2]

然后可以安装 OpenSSH Server 来远程访问
```bash
pacman -S openssh
systemctl enable --now sshd
```

## 储存
我这里使用了 LVM 来进行储存
> 如果不配置 RAID 就直接 LVM 的话有数据丢失风险!

将磁盘格式化成 Linux LVM 格式
```bash
# fdisk -l
Disk /dev/sdb: 3.65 TiB, 4000787030016 bytes, 7814037168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt

Device     Start        End    Sectors  Size Type
/dev/sdb1   2048 7814037134 7814035087  3.7T Linux LVM


Disk /dev/sdc: 3.65 TiB, 4000787030016 bytes, 7814037168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt

Device     Start        End    Sectors  Size Type
/dev/sdc1   2048 7814037134 7814035087  3.7T Linux LVM
```

然后进行创建
```bash
# pvcreate /dev/sdb1 /dev/sdc1
# vgcreate data /dev/sdb1 /dev/sdc1
# pvs
  PV         VG   Fmt  Attr PSize  PFree
  /dev/sdb1  data lvm2 a--  <3.64t <3.64t
  /dev/sdc1  data lvm2 a--  <3.64t <3.64t
# lvcreate --name storage -l 100%FREE data
# lvdisplay
  --- Logical volume ---
  LV Path                /dev/data/storage
  LV Name                storage
  VG Name                data
  LV Write Access        read/write
  LV Creation host, time nas.internal.indexyz.me, 2019-06-21 07:15:54 +0000
  LV Status              available
  # open                 0
  LV Size                <10.92 TiB
  Current LE             2861583
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:0
# mkfs.ext4 /dev/data/storage
# mkdir /data
# mount /dev/data/storage /data
# blkid | grep data-storage # 获取磁盘的 UUID
```
然后编辑 /etc/fstab 加入
```conf
# LVM Data
UUID=磁盘UUID /data ext4 defaults 0 2
```
然后执行 `mount -a` 就可以看到
```bash
# df -h
/dev/mapper/data-storage   11T   61M   11T   1% /data
```

# 应用
## 安装 Docker
> <del>Docker 大法好</del>

先在 Data 卷下面划分出一个文件夹软链接到 `/var/lib/docker` 下, 因为内置就只有一个 16GB 的垃圾 SSD, 储存 Docker Image 可能会不够用
```bash
cd /data
mkdir docker
ln -s /data/docker /var/lib/docker
```

然后安装 Docker
```bash
pacman -S docker
systemctl enable --now docker
```

## 安装 smb
```bash
pacman -S samba
nano /etc/samba/smb.conf
# 编辑配置文件
smbpasswd -a indexyz
```

我的配置文件为
```conf
[global]
  workgroup =
  server string = Indexyz NAS
  server role = standalone server
  log file = /var/log/samba/%m.log
  max log size = 50
  wins support = yes

[Indexyz]
  comment = Indexyz Shared Files
  browseable = yes
  writable = yes
  path = /data/files
  valid users = indexyz
  public = yes
  printable = no
  guest ok = no
```
可以用于参考


  [1]: https://ark.intel.com/content/www/cn/zh/ark/products/78867/intel-celeron-processor-j1900-2m-cache-up-to-2-42-ghz.html
  [2]: https://wiki.archlinux.org/index.php/Systemd-networkd
