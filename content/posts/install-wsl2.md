---
title: WSL2 踩坑日记
categories:
    - Windows
date: '2020-07-12T02:51:08+08:00'
---

618 搞了个 NUC10 做上网机用, 想了一下还是装 Windows 好了. 然后就装了 2004, 直接在 WSL2 中运行一些东西.

<!--more-->

## 安装 WSL2

### 安装 Linux Kernel

你需要下载 Linux Kernel 并安装, 可以从 [微软官方的 WSL2 Kernel 地址](https://aka.ms/wsl2kernel) 下载到. 然后我们就得到了 `wsl_update_x64.msi`  ~~这一步真的是非常的 Windows~~

### 设定 WSL 为 NoLSP

我 Windows 中使用了 Proxifier 把流量引导到 Clash 进行全局策略代理, 但是 WSL 这玩意在用 Proxifier 的情况下会有问题, 在运行 WSL 的时候会提示 `The attempted operation is not supported for the type of object referenced.`

参考 [microsoft/WSL#4177 (comment)](https://github.com/microsoft/WSL/issues/4177#issuecomment-597736482) 

> The easiest solution is to use WSCSetApplicationCategory WinAPI call for wsl.exe to prevent this. Under the hood the call creates an entry for wsl.exe at `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WinSock2\Parameters\AppId_Catalog` This tells Windows not to load LSP DLLs into wsl.exe process.

同时 Proxifier 释出了一个 [工具](www.proxifier.com/tmp/Test20200228/NoLsp.exe), 可以帮助设定 NoLSP 的设定

创建一个新的管理员 Powershell, 进入 NoLSP 所在的位置

```powershell
./NoLsp.exe c:\windows\system32\wsl.exe
```

然后再次运行 WSL 就可以正常运行了

### 安装 ArchWSL

作为一个 Arch 粉, WSL 当然要用 ~~Ubuntu~~ Arch 啦, 最新的 ArchWSL 已经支持了 WSL2, 所以我们只需要进入 [ArchWSL Release](https://github.com/yuk7/ArchWSL/releases) 下载最新的 `ArchWSL.appx` 进行安装就好了

在安装前我们需要安装 cer 文件, 选择安装到 Local Machine 的 `Trusted Root Certification Authorities` 中, 在安装完成之后就可以去删除掉这个证书.

![安装证书](https://i.loli.net/2020/06/22/P4oT5MckQJi1AnL.png)

然后在 Windows Terminal 中就可以看到 Arch 的 Profile 了

![查看 Profile](https://i.loli.net/2020/06/22/AO8IfsVHkq9GK2X.png)

或者我们可以通过 `arch` 命令进入 arch 的 shell 中去

![成果](https://i.loli.net/2020/06/22/DyT1XIEtimBdHoh.png)

## 配置

### DNS Server 有问题

WSL 默认的 DNS 服务器是宿主机, 并且每次运行会覆盖 `/etc/resolv.conf`, 这个在某些情况下会有些问题.

在 `/etc/wsl.conf` 中取消 ResolvConf 的生成

```
cat > /etc/wsl.conf <<EOF
[network]
generateResolvConf = false
EOF
```

然后打开一个新的 Powershell 窗口执行

```powershell
wsl --shutdown
```

然后在 `/etc/resolv.conf` 中添加需要的 nameserver 就好了

### 在 Arch 内的 Docker 不正常

如果直接通过 pacman 安装 docker 并且通过 `dockerd` 运行, 在运行容器的时候会提示

```
docker: Error response from daemon: cgroups: cannot find cgroup mount destination: unknown.
```

这个问题出现的原因是因为因为 WSL 的实现原因, 整个系统并不是以 systemd 作为 init 的, 所以 docker 在运行的时候找不到 cgroup 相关文件系统.

可以在 shell 中运行 [cgroup 的 workaround](https://github.com/microsoft/WSL/issues/4189) 来创建 cgroup 分区

```bash
mkdir /sys/fs/cgroup/systemd
mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
```

