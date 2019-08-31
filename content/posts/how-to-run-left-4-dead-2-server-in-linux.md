---
title: 在Linux上安装并且运行求生之路2服务器
tags: 
    - Game
    - Left 4 Dead 2
categories:
    - Game
date: 2017-08-09 18:31:55
updated: 2017-09-01 18:12:13
thumbnail: https://publish.indexyz.me/images/2017/12/10/left4dead2.png
---
~~话说求生火了这么久 怎么还不出3啊233~~

<!-- more -->

> 求生之路是一个多人联机的第一人称射击游戏 求生的社区服务器比较多 但是国内的 某些服务器 有的时候会被别人恶意攻击 这时候自建一个社区服务器是一个不错的选择

## 手动安装
### 依赖安装
在安装开始前 你需要

- 一个含有 `Left 4 Dead 2` 的 `Steam` 账号 (可选)
- 一台服务器 要求有 `10GB` 以上的硬盘 推荐 `2GB` 以上的内存

#### Debian / Ubuntu
```bash
sudo dpkg --add-architecture i386
sudo apt-get update && sudo apt-get upgrade
# 64 位
sudo apt-get install mailutils postfix curl lib32gcc1 libstdc++6 
libstdc++6:i386
# 32位
sudo apt-get install mailutils postfix curl libstdc++6
```
#### CentOS / RedHat
```bash
yum install epel-release -y
# 64 位
yum install mailx postfix curl glibc.i686 libstdc++ libstdc++.i686
# 32位
yum install mailx postfix curl libstdc++ 
```

### 安装SteamCMD
```bash
mkdir Steam
mkdir left4dead2
cd Steam
wget http://media.steampowered.com/installer/steamcmd_linux.tar.gz

tar -xvzf steamcmd_linux.tar.gz
rm steamcmd_linux.tar.gz
```

### 通过SteamCMD安装服务器
```
./steamcmd.sh
```
第一次运行会自带下载安装升级
安装完出现 `Steam>` 的时候输入
```bash
login anonymous
force_install_dir ~/left4dead2
app_update 222860 validate
```

当你看见

```
Success! App '222860' fully installed.
```
的时候 服务器便已经安装在 `~/left4dead2` 了
输入 `quit` 来退出Steam CMD (当然你可以直接 `^C` )

### 配置服务器
```bash
cd ~/left4dead2/left4dead2/cfg
```
这时候
```bash
nano server.cfg
```

我使用的配置文件为 
```
hostname "Indexyz Veritas"     // 服务器名
sv_voiceenable 1               // 开启语音
sv_steamgroup "Group"          // 设置为 ID 为 Group 的组服务器 
Steam组管理员可以查询到组ID
sv_steamgroup_exclusive 1      //将服务器设为Steam组私有
```

### 服务器启动
创建一个 `run` 文件并且给它可执行权限
```bash
cd ~/left4dead2
touch run
chmod +x run
```
现在可以编辑启动文件
```bash
nano run
# 输入
./srcds_run -game left4dead2 +exec server.cfg
```
启动器服务器只需要 `./run`

推荐使用 `tmux` 来管理会话 否则关闭终端之后服务器将退出 关于 `tmux` 
请自行搜索引擎

## 自动安装
```bash
adduser l4d2server
su - l4d2server
wget http://gameservermanagers.com/dl/l4d2server
chmod +x l4d2server
```
现在
```bash
./l4d2server install
# 会询问服务器信息
```
安装完成之后
```bash
./l4d2server start
```
以后每次需要操作服务器的时候只需要
```bash
su - l4d2server
./l4d2server command
```

## 其他的坑
### 无法链接 或者 无法找到服务器
尝试允许l4d2服务器数据包
```bash
iptables -A INPUT -p udp- m udp --sport 4380 --dport 1025:65355 -j ACCEPT
iptables -A INPUT -p udp -m udp --sport 10999 --dport 1025:65355 -j ACCEPT
iptables -A INPUT -p udp -m udp --sport 7777 --dport 1025:65355 -j ACCEPT
iptables -A INPUT -p udp -m udp --sport 27015 --dport 1025:65355 -j ACCEPT
service iptables save
```

