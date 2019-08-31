---
title: 自动更新 Rancher 上的应用
tags: 
    - Software
categories:
    - Software
date: 2018-02-03 23:49:00
updated: 2018-02-03 23:49:00
---
之前有介绍过 `rancher` 的使用方式 现在有个新的问题 如果我们需要在每次 `CI` 构建结束之后都自动更新 rancher 上的 service 该怎么办

<!--more-->

Rancher 官方有提供一个 [API](https://rancher.com/docs/rancher/v1.6/en/api/v2-beta/) 通过他我们可以方便的管理 rancher 的各个功能

当然 我也写了一个小工具来更新 service

{% github Indexyz rancher-webhook-update %}

## 如何使用
现在我就适配了在 `gitlab` 的 CI 构建完成之后来更新 service

为什么选择 gitlab ? 

原因有两个
1. GitLab 提供免费的 Registry 可以在每次构建成功之后直接从他的 Registery 中拉下映像
2. GitLab 支持免费私有 Repo

~~简单来说就是我穷~~

### 安装
修改 `config.py` 中的内容为你的配置文件 

当然这些配置项都是可以通过环境变量来设置的

这样可以方便我们直接在 `Docker` 中跑起来项目 或者是在 `Heroku` 上托管

### 运行
直接 `gunicorn main:app --log-file=-` 然后就运行了一个在 `:8080` 上的服务器

### 配置
在 `GitLab` 的 `Settings > Integrations` 添加 `Webhook`

![gitlab integations][gitlab integations]

> 通过 [random.org](https://www.random.org) 来生成高强度的 `Secret Token`

我们在下面的多选框中选中 `Pipeline events` 并且输入我们的 updater 的地址


地址 的格式为
```
http(s)://url-of-updater/webhook/gitlab/{prject-id}/{service-id}
```

然后 在每次构建成功的时候就会自带推送了!


[gitlab integations]: https://publish.indexyz.me/images/2018/02/04/gitlab-integations.png