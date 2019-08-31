---
title: 更新CentOS6上的Git版本
tags: 
    - Linux
    - Git
categories:
    - Linux
date: 2017-08-09 17:53:01
updated: 2017-09-01 18:18:17
thumbnail: https://publish.indexyz.me/images/2017/12/10/git.png
---
最近在玩ss-panel, 在编译Go版本的ss的时候出现编译一直卡死的情况
查找资料后发现是Git的版本太低了(Git要高于1.7.1)

<!--more-->

然而CentOS里最新的Git确实
```bash
#git --version
git version 1.7.1
```

真是史前版本![喷][1]

于是我们需要更新Git到新版本

  [1]: https://publish.indexyz.me/images/2016/05/1334795755.png


<!--more-->

## 方法一(采用第三方的yum源来更新Git)
1. 下载repo， -o 也就是 --output-file， 把下面链接下载的放到某位置上
```bash
wget -O /etc/yum.repos.d/PUIAS_6_computational.repo 
https://gitlab.com/gitlab-org/gitlab- 
recipes/raw/master/install/centos/PUIAS_6_computational.repo
```
2. 下载GPG KEY
```bash
wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-puias 
http://springdale.math.ias.edu/data/puias/6/x86_64/os/RPM-GPG-KEY-puias && rpm 
--import /etc/pki/rpm-gpg/RPM-GPG-KEY-puias
```
3. 验证key是否安装成功
```bash
rpm -qa gpg*
```
4. 验证yum是否正常
 ```bash
yum repolist
```
5. 更新Git
```bash
yum update git
```
## 方法二(构建最新的版本)
最近的版本可以在GitHub上找到 [GitHub-Git](https://github.com/git/git/)
```bash
# 安装依赖
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel 
perl-devel
# 下载源代码
wget https://github.com/git/git/archive/v2.8.2.tar.gz
tar zxvf v2.8.2.tar.gz
cd git-2.8.2/
make prefix=/usr/local install
```
然后就安装了最新的Git了
```bash
# git --version
git version 2.8.2
```
