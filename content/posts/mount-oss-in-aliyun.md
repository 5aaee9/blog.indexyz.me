---
title: 在阿里云服务器上挂载OSS储存来进行拓展
tags: 
    - Linux
    - Aliyun
categories:
    - Linux
date: 2017-08-09 18:13:59
updated: 2017-09-01 18:14:49
thumbnail: https://publish.indexyz.me/images/2017/12/10/aliyun.png
---
> 最近有一部分文件要用到大容量的储存，然而阿里云的磁盘IO和大小我就不说什么了，这时候我看到了阿里云的OSS，搜索了一下发现可以挂载成虚拟硬盘


<!--more-->

## 安装
```bash
wget 
https://github.com/aliyun/ossfs/releases/download/v1.79.9/ossfs_1.79.9_centos7.0_x86_64.rpm
yum install -y ossfs_1.79.9_centos7.0_x86_64.rpm
```
写入你需要挂载的Bucket，Access Key ID 和 Access Key Secret等信息，格式如下
```bash
echo bucketname:AccesskeyID:Accesskeysecret > /etc/passwd-ossfs
chmod 640 /etc/passwd-ossfs
```
## 挂载
```bash
mkdir /mnt/oss
echo 'ossfs#bucket_name mount_point fuse _netdev,url=url,allow_other 0 0' >> 
/etc/fstab
# 修改bucket_name mount_point url为你自己的
mount -a
```

