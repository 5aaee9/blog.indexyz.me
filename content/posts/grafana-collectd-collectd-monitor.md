---
title: Linux 搭建 grafana + influxdb + collectd
tags: 
    - Linux
    - Monitor
    - Dashboard
categories:
    - Linux
date: 2017-10-28 08:18:21
updated: 2017-10-28 15:29:18
thumbnail: https://publish.indexyz.me/images/2017/12/10/grafana.png
---
最近一直想搞个监控系统来检测下大陆到国外的延时什么的, 但是 `SmokePing` 的图表太玄学了 而且界面像是上个世纪的 所以想坑一下计划了很久的 `Grafana`.

<!-- more -->

> 抽烟ping看得懂的，都是文森特凡高的后代啊（蔡博语）…… 来搞IT可惜了…… —— 某大佬

# 环境
本文接下来都在 `Arch Linux` 上安装 大部分软件的搭建使用 `Docker` 来保证别的系统也可用
数据储存在 `/data`

## 安装 Docker
对于其他系统, 官方已经准备了一键安装脚本 执行就好了
```bash
curl -sSL get.docker.com | bash
```
对于 `Arch Linux` 官方的源中已经有了 `Docker` 的存在
```bash
pacman -S docker
systemctl start docker
systemctl enable docker
```

## Pull 所需的映像
* influxdb:alpine
* grafana/grafana
* busybox:latest

# Collectd
对于 `Collectd` 我们使用从包管理器安装的方法安装 而不是运行 `Docker 映像` 因为 `Collectd` 的数据收集 `CPU 内存 网络使用` 需要收集宿主机的信息
## 安装
默认包管理已经有现成的了
```bash
pacman -S collectd
```
其他系统请参考 [官方教程](https://collectd.org/download.shtml)
## 启动
使用 `systemd` 管理
```bash
systemctl enable collectd
systemctl start collectd
```
# InfluxDB
首先我们需要生成配置文件 
```bash
docker run --rm influxdb:alpine influxd config > influxdb.conf
```
这时候在本目录 `/data` 下可以看见 `influxdb.conf` 这一个配置文件了
## 运行
`Docker` 运行就好了
```bash
docker run -d -p 8086:8086 -p 8083:8083 -p 25826:25826/udp \
  -v /data/influxdb_volume:/var/lib/influxdb \
  -v /data/influxdb.conf:/etc/influxdb/influxdb.conf:ro \
  -v /usr/share/collectd/types.db:/usr/share/collectd/types.db \
  --restart=always --name influxdb \
  -e GOGC=10 \
  -e INFLUXDB_DATA_INDEX_VERSION=tsi1 \
  -e INFLUXDB_ADMIN_ENABLED=true \
  influxdb:alpine -config /etc/influxdb/influxdb.conf
```
这时候就运行了 `influxdb` 了
## 身份验证
通过 `exec` 进入容器创建账户
```bash
docker exec -it {container_id} bash
influx
# 创建账户
> CREATE USER <username> WITH PASSWORD '<password>' WITH ALL PRIVILEGES
# 创建数据库
> CREATE DATABASE collectdb
> quit
# ^P + Q 
```
然后编辑 `influxdb.conf`
```bash
$ nano influxdb.conf
# 找到 
# [http]
#  enabled = true
#  bind-address = ":8086"
#  auth-enabled = false     <---
```
将 `auth-enabled` 改为 `true` 然后保存退出 最后重启容器
```bash
docker restart {container_id}
```
这时候 `InfluxDB` 已经安装成功了
## 和 Collectd 对接
### InfluxDB 部分
打开 `influxdb.conf` 配置文件
找到
```conf
[[collectd]]
  enabled = false             // <-----
  bind-address = ":25826"
  database = "collectd"       // <-----
```
将这里的 `enable` 改为 `true` 打开 `collectd` 收集
同时将 `database` 改为要收集到的数据库 根据上下文应该为 `collectdb`
> 注意 25826/udp 是可以绕过账户保护的 请对这个端口使用防火墙什么的

修改完成后重启 `influxdb` 接下来要修改 `collectd` 配置文件了
### Collectd 部分
`collectd` 的配置文件在 `/etc/collectd.conf`
我使用的配置文件为
```bash
# cat /etc/collectd.conf
BaseDir "/etc/collectd"
PIDFile "/run/collectd.pid"
Hostname "localhost"
Interval 60
<loadplugin df> 
    Interval 120
</loadplugin>
LoadPlugin disk
LoadPlugin interface
LoadPlugin load
LoadPlugin memory
LoadPlugin network
LoadPlugin processes
LoadPlugin users
LoadPlugin ping
<plugin interface>
    Interface "eth0"
    IgnoreSelected false
</plugin>
<plugin network>
    Server "{ InfluxDB Host }" "25826"
</plugin>
<plugin ping>
    Host "{ Host you want to ping }"
    Host "{ Another host you want to ping }"
    Interval 5.0
    Timeout 0.9
    TTL 255
</plugin>
```
你可以参考我的配置文件对 `collectd` 进行配置
配置完成之后
```bash
systemctl restart collectd
```
# Grafana
## 安装
`Grafana` 的安装很简单 只需要
```bash
docker run -d -v /var/lib/grafana --name grafana-storage busybox:latest
docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  --volumes-from grafana-storage \
  grafana/grafana
```
访问 `http://localhost:3000` 就可以看到面板了
默认账号和密码都是 `admin` 请及时修改
![Grafana Index](https://publish.indexyz.me/images/2017/12/12/Grafana-Index.png)

## 数据源
点击 `Getting Started` 中的 `Add data source` 我们来添加 `influxdb` 作为数据源
按照下图设置数据源 然后点击 `Add` 按钮 如果可以链接就代表成功了
![Grafana Data Source](https://publish.indexyz.me/images/2017/12/12/Grafana-Data-Source.png)

## 创建图表
`Create Dashboard` 然后选 `Graph`
然后点击 `Panel Title` 在弹出的选项中点击 `Edit` 然后就可以添加查询了
一个查询可能长这样
![Grafana Add Query](https://publish.indexyz.me/images/2017/12/12/Grafana-Add-Query.png)

# 成果
最后多添加几个查询 然后改下 `Title` 什么的 一个监控面板就出来了
![Grafana Dashboard](https://publish.indexyz.me/images/2017/12/12/Grafana-Dashboard.png)
当然这里没有细讲各个组件的高级应用 自己慢慢玩吧(
