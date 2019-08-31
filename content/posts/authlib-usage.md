---
title: Authlib 使用教程
tags: 
    - Minecraft
    - Game
    - Usage
categories:
    - Minecraft
date: 2017-08-09 18:35:11
updated: 2017-09-01 09:12:29
thumbnail: https://publish.indexyz.me/images/2017/12/10/minecraft.png
---
总所周知 Minecraft 使用了被称为 `Yggdrasil`  的正版认证系统 实现了在对等情况下的身份认证

<!-- more -->

但是在开启了离线模式的服务器中 也有着许多的身份认证方式 例如使用 `AuthMe` 或者 `CrazyLogin` 之类的登陆插件

但是这些插件对与Mod开发者并不友好 在某些使用了 `FakePlayer` 的环境下会把假人也当为玩家 这时候这个假人的各种操作就炸了

有没有解决这些问题的登陆方式呢 那当然是有的啦 这就是 [authlib](https://github.com/to2mbn/authlib-agent)

## 原理
通过对 `Java字节码` 的修改 将 `Mojang` 的请求重定向到自己的认证服务器

### Identity 和 Authlib 的 Agent 的区别
大体构建相同 都是使用自己的API Server重定向请求
Identity最主要的特点是开箱即用 这点比官方的Authlib要好
事实上我和Authlib作者聊过 在未来可能会规范API的路由
有可能 ~~(这个Flag立的)~~ 我会在未来的某个时候完全重写Identity也不一定 咳咳扯远了

## 使用
### 安装 
前提环境是 `Nodejs` 我们可以使用 [NVM](https://github.com/creationix/nvm) 来方便的进行Nodejs的安装
数据库环境需要 `MongoDB` 和 `Redis` 这两个环境的安装在互联网上已经很多了 这里不在讲述 
> 注意 
> MongoDB 请不要在没有设置账户验证的情况下将端口开放到公网 就算是设定了账户认证最好也不要开放到公网 
> Redis在不开放到公网的情况下建议设置强密码加密 

这时候就可以通过Git获取最新的源代码安装了
```bash
git clone -b reload https://github.com/Indexyz/Identity.git
npm install
// Configure your settings in config.js
npm run start
```
这时候访问你的localhost:3000 
你就成功了
### 使用Docker安装
Docker运行应该是最方便的运行方式了 他可以方便快捷的管理每个服务
```
docker run -d -v /data/identity/db:/data/db --name MongoDB-Identity mongo
docker run -d --name Redis-Identity redis
docker run --link MongoDB-Identity:mongo --link Redis-Identiy:redis \
    # -e ENV_KEY=ENV_VALUE
    -p 80:3000 indexyz/identity
```
其中的 `ENV_KEY` 和 `ENV_VALUE` 可以在 [Identity Wiki-Environment](https://github.com/Indexyz/Identity/wiki/Environment) 看到
然后访问本地的80端口 你就可以看到Identity的界面了
## 和Authlib-agent进行配合
使用我修改过的 [authlib-agent](https://github.com/Indexyz/authlib-agent) 修改其中的链接然后进行打包
```bash
git clone https://github.com/Indexyz/authlib-agent
nano configure.sh            # 修改 AGENT_API_ROOT 和 AGENT_SKIN_DOMAINS
sh build.sh
```
运行之后你会在根目录下见到 `authlibagent.jar` 这就是打包出来的jar了
如果你想不接入自己的 Identity 可以选择使用 [官方服务器](http://authentication.mcdev.se/) 
这是他的[AuthlibAgent.jar](https://public.hyperworld.xyz/Gamer/Minecraft/AuthLib/authlibagent.jar)
