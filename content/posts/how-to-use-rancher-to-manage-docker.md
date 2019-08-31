---
title: Rancher 的使用和容器的管理
tags: 
    - Linux
    - Docker
categories:
    - Linux
date: 2017-08-09 18:20:33
updated: 2017-09-01 18:13:30
thumbnail: https://publish.indexyz.me/images/2017/12/10/rancher.png
---
> Rancher is an open source project that provides a complete platform for 
operating Docker in production. It provides infrastructure services such as 
multi-host networking, global and local load balancing, and volume snapshots. It 
integrates native Docker management capabilities such as Docker Machine and 
Docker Swarm. It offers a rich user experience that enables devops admins to 
operate Docker in production at large scale.
> -- GitHub

<!--more-->
## 前言
最近一直在找一个好用的容器管理服务, 也尝试了很多 例如 
[gaoyangxiaozhu/DockerVI](https://github.com/gaoyangxiaozhu/DockerVI) 
但是无一例外有很多的缺点. 例如部署困难、界面不友好、不是自由/开放源代码软件.
今天, 我在一个关于Docker的文章中见到了 Rancher 然后部署并且试用之后就喜欢上了它.

## 安装
[官方网站](http://rancher.com/)
```bash
docker run -d --restart=always\
       -v /data/rancher/db:/var/lib/mysql \
       --name rancher-server \
       -p 8888:8080 \
       rancher/server
```
只需要启动这一个容器 然后访问localhost:8888 就可以看到Rancher的界面了
在界面的右下角可以切换语言为简体中文
### 用户验证
在`系统管理/访问控制`下可以进行用户的验证
![BFBVJQ{DOH}YX)Z\])0M5$XX.png][1]

我选择的是使用GitHub进行身份验证 按照提示生成`Client ID`和`Secret`
![QQ截图20170102011147.png][2]

保存之后进行一次验证 验证完成之后就自动记录了进行验证的GitHub ID, 
以后就可以用这个ID进行OAuth验证登陆了

![QQ截图20170102011615.png][3]

## 添加主机
在`基础构架/主机`下可以见到主机列表 点击`添加主机`进行添加主机的流程
![QQ截图20170102011943.png][4]

使用SSH登陆到主机上 执行
```bash
curl -sSL get.docker.com | bash
systemctl start docker
systemctl enable docker
```
来安装Docker 
然后输入生成了的脚本安装 
这里如果使用运行Rancher的服务器安装的话在第`4`个步骤的输入框内填入服务器的IP
然后你就可以在`基础构架/主机`下看到你添加的服务器了
![QQ截图20170102012808.png][5]

## 运行容器
在每个主机的最后有一个按钮可以添加容器
![QQ截图20170102013007.png][6]

这里以运行`hyalx/ssr-with-net-speeder`为例子 演示一下容器的运行方法
进行一下设置然后点击创建![QQ截图20170102013319.png][7]

Rancher便会自动Pull下来映像并且运行

## 反向代理
当然 如果我们要使用一个服务 有时候那么长长的一串IP和端口不是那么的美观
在这里我使用了Nginx来进行反向代理
安装Nginx (Yum 安装)
```bash
rpm -Uvh 
http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
yum install nginx
systemctl start nginx 
systemctl enable nginx
```
访问IP 出现Nginx默认页面
在`cd /etc/nginx/conf.d/`下可以编辑默认配置文件 这里提供我的配置文件作为参考
```shell
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
server {
    listen       80;
    server_name  localhost;

    location / {
        proxy_pass http://127.0.0.1:端口;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        sub_filter IP地址:端口 IP地址:80;
        sub_filter_once off;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
> 修改了地址请在GitHub修改应用的redict url和homepage 
要不然每次登陆都会跳转会原来的地址

> 因为Rancher使用了Webshock 所以和正常的反代方式有些不同


  [1]: https://publish.indexyz.me/images/2017/01/3269573140.png
  [2]: https://publish.indexyz.me/images/2017/01/3952703091.png
  [3]: https://publish.indexyz.me/images/2017/01/4151018919.png
  [4]: https://publish.indexyz.me/images/2017/01/4219189877.png
  [5]: https://publish.indexyz.me/images/2017/01/3018941036.png
  [6]: https://publish.indexyz.me/images/2017/01/1644447780.png
  [7]: https://publish.indexyz.me/images/2017/01/3126909881.png
