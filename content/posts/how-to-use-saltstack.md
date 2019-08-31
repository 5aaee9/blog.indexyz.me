---
title: 使用 SaltStack 进行集群管理
tags: 
    - Linux
    - Operation
categories:
    - Linux
date: 2017-08-09 18:33:14
updated: 2017-09-01 18:11:15
thumbnail: https://publish.indexyz.me/images/2017/12/10/saltstack.png
---
Saltstack 是一个使用 Python 编写的开源项目 扩展性很好 可以方便快速的管理大量服务器 它使用了 ZeroMQ 消息队列实现服务端之间的秒级通讯

<!-- more -->

> 实在是厌倦了对大量服务器日复一日的重复操作。尤其是在虚拟化的时代，系统的每个组件都有很多个相同的节点在运行，更让重复的次数再乘以N。 当我发现Salt的时候，我的眼前一亮：这正是我所需要的东西。

## 始
我现在在使用 Saltstack (以下简称salt) 进行我的服务管理 因为我的有些服务并不能 Docker 化而是要统一部署和管理 (比如各个Rancher的Node管理什么的) 
### 概念
salt 在部署上可分为 `master` 和 `minion` master 负责下发指令和配置文件之类的消息 minion 负责执行命令并返回消息

salt 提供了文件配置和命令行管理两种主机管理方式

salt 在配置文件的储存方式上使用了便于读写的 [`YAML`](https://zh.wikipedia.org/wiki/YAML) YAML 主要使用了空格和缩进来表达语义 上手也挺快的 看完 Wikipedia 上的介绍就基本上可以使用 YAML 了
#### Grains
`grains` 相当于各个节点独立的配置文件 

储存在每个节点的 `/etc/salt/grains` 当中 使用 `YAML` 格式储存

#### Salt 文件系统
Salt 中的文件以 `salt://` 开头 指 saltstack 主目录(默认为`/srv/salt` 可在 master 的设定中更改)的相对路径 

### 安装
#### Salt Bootstrap
salt官方有提供一个通用的
[安装脚本](https://docs.saltstack.com/en/latest/topics/tutorials/salt_bootstrap.html)
 通常来说 我们只需要
```bash
curl -L http://bootstrap.saltstack.org | sudo sh -s -- -M -N  # 仅安装Master
wget -O - http://bootstrap.saltstack.org | sudo sh            # 安装 Minion
```

#### 包管理安装
salt 在各个 Linux 发行版的软件包管理器中应该都提供了现成的软件包 
```bash
apt-get install salt-master         # Ubuntu
yum install salt-master             # Centos, require epel
```

#### 修改配置文件
在 minion 端执行 
```bash
sed -i 's/#master: salt/master: Master主控IP/g' /etc/salt/minion
systemctl restart salt-minion
```

#### 接受Key
salt 使用AES加密消息来保障信息传输的安全性 因此 master 首先应该接受 minion 的密匙才能完成配对
```bash
salt-key -L
```
你应该可以看到如下提示消息
```
Accepted Keys:
Denied Keys:
Unaccepted Keys:
salt-minion-1
Rejected Keys:
```
使用 以下指令接受 `salt-minion-1` 的密匙
```bash
salt-key -a salt-minion-1
```
当然 你也可以 用以下指令接受全部的密匙
```bash
salt-key -A
```

## 开始使用
### 下发指令
#### 常用指令
##### 测试所有节点的连通性
```bash
salt '*' test.ping
```
##### 删除已离线的 minion
```bash
salt-run manage.down removekeys=True
```
##### 测试网络连通性
```bash
salt '*' cmd.run 'ping google.com -c 4'
```
#### Minion选择器
在上面的几个常用的指令中 使用了 `salt '*'` 来选择所有节点 那么 还有没有别的选择方式么 那当然是有的

##### 根据节点ID选择
```bash
salt 'salt-minion-1' test.ping
```

##### 根据 grains 是否存在选择
```bash
salt -G 'flag:1' test.ping           # Flag 是一个为 1 的 grains属性
```
因为 grains 预处理了一些变量 所以我们可以
```bash
salt -G 'os:CentOS' test.ping        # 在所有 CentOS 系统的 minion 上执行
```
#### 常用Execution
官方其实有个 
[Execution的列表](https://docs.saltstack.com/en/latest/ref/modules/all/index.html)
在这也讲一下常用的几个常用Execution好了
##### 运行命令
```bash
salt '*' cmd.run '{command}'        # 替换 Command 为你的命令
```
##### 服务操作
```bash
salt '*' service.status {service}   # 查看服务状态
salt '*' service.restart {service}  # 重启服务
salt '*' service.start {service}    # 启动服务
salt '*' service.stop {service}     # 停止服务
```
##### 文件分发
```bash
salt '*' cp.get_file salt://{file} /to/path/{file}
```
追加到文件尾
```bash
salt '*' file.append /to/file "{data}"
``` 
