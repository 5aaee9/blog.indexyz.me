---
title: CentOS编译安装Python
tags: 
    - Linux
    - CentOS
    - Python
categories:
    - Linux
date: 2017-08-09 17:50:31
updated: 2017-09-01 18:18:45
thumbnail: https://publish.indexyz.me/images/2017/12/10/python.png
---
在CentOS6上，默认安装的Python版本是2.6，然而有些软件需要使用Python2.7

<!--more-->

于是只能手动编译升级Python版本

 1. 安装依赖

    ```shell
    yum install -y gcc openssl-devel
    ```

 2. 下载源码包并且编译安装
    ```shell
    wget https://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz
    tar xvf Python-2.7.11.tgz
    cd Python-2.7.11
    ./configure --prefix=/usr/local/python27
    make
    make install
    # 这时候旧的Python还存在，需要移动或者删除掉
    rm /usr/bin/python
    ln -s /usr/local/python27/bin/python /usr/bin/python
    ```

 3. 解决yum等软件的问题
    ```shell
    # CentOS6的yum是不支持Python2.7的，我们需要手动切换到Python2.6
    # 编辑/usr/bin/yum
    vim /usr/bin/yum
    # 将头部的
    #!/usr/bin/python
    # 改为
    #!/usr/bin/python2.6
    ```

 4. 安装pip和easy_install
    ```shell
    wget https://bootstrap.pypa.io/get-pip.py
    python get-pip.py
    ```