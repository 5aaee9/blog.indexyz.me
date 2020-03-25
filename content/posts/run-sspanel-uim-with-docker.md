---
title: 使用 Docker 快速的运行 SSPanel-UIM
date: '2020-03-26T01:24:39+08:00'
categories:
- Linux
- Docker
---

Docker is awesome!

<!-- more -->

## 安装环境

以下使用 Debian 10 作为基础环境, 通过 Docker 官方的安装方式安装的 Docker

```bash
curl -sSL get.docker.com | bash
```

## 安装数据库

使用了 MariaDB 作为数据库, 可以使用

```bash
export DB_PASS="P@55w0rD" # 数据库密码
docker volume create mariadb

docker run \
    --name database -d \
    -e MYSQL_ROOT_PASSWORD=$DB_PASS \
    -v mariadb:/var/lib/mysql \
    mariadb:10.5.1
```

### 恢复 SQL 文件

下载 `glzlin_all.sql` 然后恢复到数据库中

```bash
wget https://github.com/Anankke/SSPanel-Uim/raw/dev/sql/glzjin_all.sql

docker exec -i database sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' <<EOF
CREATE DATABASE IF NOT EXISTS sspanel CHARACTER SET utf8 COLLATE utf8_general_ci;
EOF

docker exec -i database sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD" sspanel' < glzjin_all.sql
```

这样就完成了恢复

### 安装 PHPMyAdmin (可选)

```bash
docker run --name phpmyadmin -d \
    -d --link database:db \
    -p 8080:80 \
    phpmyadmin/phpmyadmin
```

可以自己将 `-p 8080:80` 中 8080 换为你需要的端口 默认运行在 :8080 上

访问 http://ip:8080 就可以访问到 PMA 面板了

## 安装面板

```bash
docker run --name sspanel -d \
    --link database:db \
    -e UIM_ENV_REPLACE_ENABLE=1 \
    -e UIM_DB_HOST=db \
    -e UIM_DB_DATABASE=sspanel \
    -e UIM_DB_USERNAME=root \
    -e UIM_DB_PASSWORD=$DB_PASS \
    -p 80:80 \
    sspaneluim/panel

docker exec -i sspanel sh -c 'php xcat initQQWry'
docker exec -i sspanel sh -c 'php xcat resetTraffic'
docker exec -i sspanel sh -c 'php xcat initdownload'
```

可以快速的运行一个可以使用的 SSPanel 实例

### 修改 Config

查看 [Example Config](https://github.com/Anankke/SSPanel-Uim/blob/dev/config/.config.example.php), 看到要修改的 key 名称, 例如 `$_ENV['smtp_host']`, 将其大写后加入 `UIM_` 的 Prefix, 变为 `UIM_SMTP_HOST`, 设置这个环境变量为你需要的值

### 创建 Admin 账户

```bash
docker exec -i sspanel sh -c 'php xcat createAdmin'
```

然后根据提示就可以创建完成 Admin 账户了, 访问 http://ip:80 即可看到面板

### 建议修改的 Config

- `baseUrl (UIM_BASEURL)`  - 站点地址

- `muKey (UIM_MUKEY)` - WEBAPI 校验 KEY

- `appName (UIM_APPNAME)` - 应用名称

### 修改其他文件

将文件 mount 到 /var/www 下面, 例如修改 /config/appprofile.php 这个文件, 只需要在启动参数中加入

```text
-v /root/appprofile.php:/var/www/config/appprofile.php
```

此处就使用宿主机器上的 /root/appprofile.php 替换掉了容器当中的 /config/appprofile.php 文件

## 高级操作

### 使用 Nginx 作为前置代理

> TODO
