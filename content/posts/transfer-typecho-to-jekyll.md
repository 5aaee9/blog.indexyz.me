---
title: 又将Typecho换到了GitHub上的Jekyll
tags: 
    - Blog
categories:
    - Blog
date: 2017-08-09 18:29:01
updated: 2017-09-01 18:11:58
thumbnail: https://publish.indexyz.me/images/2017/12/10/jekyll.png
---
## 再次更换博客的构架

<!-- more -->

### 原因?
自建 `Typecho` 的环境太过麻烦了 在DigitalOcean上跑一个512MB的机子也带不动数据库加PHP 
(而且 `MySQL` 的吃内存业界皆知 512MB的内存可能都带不动)
### 遇到的困难

- `GitHub Page` 不支持自定义域名的Https证书 
对我这种有Https强迫症的人来说简直是个末日

- 静态页面评论贼尴尬

### 解决方法

1. - 使用自己服务器上的 `Nginx` 反向代理 `GitHub Page`
     
     这种方法是我现在在使用的 谁叫我服务器在吃灰呢

     `Nginx` 配置文件 [GitHub Gist](https://gist.github.com/taddev/8872330)
   - 听说 使用 `CloudFlare` 的服务可以反向代理 `GitHub Page` 实现Https
     
     这种方法的好处还是蛮多的 比如不需要自己搞Https证书 但是需要切换域名的NS到
      CloudFlare所以我没有使用

2. 使用 `Disqus`
   
   在多说倒闭了之后各种静态页面的层出不穷 我使用了Disqus作为评论系统 
虽然他在中国大陆可能被墙

## Jekyll 相关
### 主题 
使用了 `码志` 的 [Jekyll主题](https://github.com/mzlogin/mzlogin.github.io) 
自己进行了一些微小的改动

### 遇见的困难
也不知道 算不算是一个困难
在我的笔记本 ( `Arch Linux` )上从 `Aru` 安装了Ruby之后安装了环境无法启动 

最后使用了 `~/.gem/ruby/2.4.0/gems/` 这种路径手动启动