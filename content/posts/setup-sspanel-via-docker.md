---
title: 使用 Docker 快速安装 ss-panel-v3-mod
tags: 
    - Docker
    - Shadowsocks
categories:
    - Docker
date: 2017-12-26 15:43:08
updated: 2017-12-26 23:23:27
thumbnail: https://publish.indexyz.me/images/2017/12/10/paperplane.png
---
`ss-panel-v3-mod` 是一个美观 功能强大的代理管理面板 但是依赖什么的对新手简直是爆炸 装个 `lnmp` 都要一个小时左右 现在使用 `Docker` 可以快速的完成面板的搭建

<!-- more -->

# 始
我们需要首先安装 `Docker`
```bash
curl -sSL get.docker.com | bash
# 在 Arch Linux 当中 官方源已经有了 Docker 我们直接安装就好了
pacman -S docker
systemctl enable docker
systemctl start docker
```

我们需要 `Pull` 以下映像
- mysql
- indexyz/ss-panel-v3-mod-docker
- indexyz/php-crontab
- phpmyadmin/phpmyadmin

全部的文件储存在 `/data` 这个 `Endpoint` 下面
使用 
```bash
mkdir /data
cd /data
```
来开始

# 开始搭建
## 搭建 MySQL
我们需要安装 `MySQL` 作为数据库存储信息
使用 `Docker` 运行数据库只需要
> 我们假定数据库密码为 pa44w@rd

```bash
# 如果想只使用 Web API 进行链接 可以取消掉 -p 那一段
docker run --name panel-database \
    --restart=always -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=pa44w@rd -d \
    mysql:5.7.22
```
## 安装 PHPMyAdmin
`PHPMyAdmin` 是一个方便的 MySQL数据库 管理工具
```bash
docker run --name panel-phpmyadmin \
    -d --link panel-database:db \
    -p 8080:80 \
    phpmyadmin/phpmyadmin
```
然后访问 `http://localhost:8080` 就能看到 `PHPMyAdmin` 的界面了 
使用用户名 `root` 和密码 `pa44w@rd` 就能登录到数据库了
## 创建数据库
在 PHPMyAdmin 创建一个数据库
> 在本教程中 数据库名称为 panel

![Create-Database][CREATE_DATABASE]
导入这个 [数据库文件][SQL_FILE] 到刚刚创建的数据库中
## 安装面板
首先我们先把配置文件下载过来 使用
```bash
wget https://github.com/esdeathlove/ss-panel-v3-mod/raw/new_master/config/.config.php.example -O config.php
```
然后我们就能创建容器了
```bash
docker run --name panel -d \
    -v /data/config.php:/data/www/config/.config.php \
    --link panel-database:db \
    -p 10080:80 \
    indexyz/ss-panel-v3-mod-docker
```
这时候可以访问 `http://localhost:10080` 了 不过我们还没有配置数据库 所以还是会返回 `500` 错误
## 配置 ss-panel
我们使用文本编辑器打开 `config.php`
找到以下内容
```php
$System_Config['db_host'] = 'localhost';
$System_Config['db_database'] = '';
$System_Config['db_username'] = '';
$System_Config['db_password'] = '';
```
修改为
```php
$System_Config['db_host'] = 'db';
$System_Config['db_database'] = 'panel';
$System_Config['db_username'] = 'root';
$System_Config['db_password'] = 'pa44w@rd';
```
别的配置项按自己的需要配置 或者按自己的心情配置 配置完成之后重启下容器
```bash
docker restart panel
```
然后就能显示站点了（
## 配置定时任务
这部分很简单 只需要
```
docker run -d \
    -v /data/config.php:/data/www/config/.config.php \
    --link panel-database:db \
    indexyz/php-crontab
```

# 常见问题
## 我该怎么创建管理员帐号
有两种方法 往数据库的邀请码加入邀请注册后直接改 `is_admin`
还有一种就是进入容器进行添加
```
docker exec -it panel bash
/usr/local/php/bin/php /data/www/xcat createAdmin
# 然后提供邮箱什么的信息 就创建完成了
```

[CREATE_DATABASE]: https://publish.indexyz.me/images/2017/12/12/phpMyAdmin-CreateDatabase.png
[SQL_FILE]: https://github.com/esdeathlove/ss-panel-v3-mod/raw/new_master/sql/glzjin_all.sql
