---
title: Rancher 使用 NFS 在多个主机之间共享 Volume
date: '2018-12-31T00:45:00+08:00'
categories:
- Rancher
---
~~Rancher Is The BEST!~~

Rancher 大幅度减轻了我多机负载均衡的难度, 但是内建的 Volume 系统比较难配, 踩了不小的坑（


<!--more-->

# 安装 NFS
NFS 这玩意并不能使用用户名密码登陆, 这里我直接添加了一台 Host 然后创建了一个 service 在 apps 这个 stack 下面
```yaml
version: '2'
services:
  nfs:
    privileged: true
    image: itsthenetwork/nfs-server-alpine:latest
    environment:
      SHARED_DIRECTORY: /nfsshare
    stdin_open: true
    volumes:
    - /data/nfsshare:/nfsshare
    tty: true
```

# 安装 NFS Volume Provider
在应用商店中找到并且部署 `Rancher NFS`

![Rancher NFS][1]

点击 Deployed 并且使用 `serviceName.stackName` 作为地址
> 此处使用 `nfs.apps`

然后会在每个 node 上运行 nfs 的 client

## 注意事项
### 容器启动的时候 failre
`Rancher NFS` 的实现依赖各个主机上的内核对 nfs 的实现, 在某些发行版上 (点名 Arch Linux) 会出现 `mount.nfs: No such device` 的情况

例如 Arch Linux, 执行 `pacman -S nfs-utils` 然后重启机器就可以解决这个问题

  [1]: https://i.loli.net/2018/12/31/5c28f56751fa3.png
