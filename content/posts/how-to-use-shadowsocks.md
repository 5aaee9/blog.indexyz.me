---
title: Shadowsocks的正确食用方法
tags: 
    - Shadowsocks
    - Usage
categories:
    - Usage
date: 2017-08-09 17:54:18
updated: 2017-09-01 18:18:07
thumbnail: https://img10.360buyimg.com/img/jfs/t1/63451/4/9609/85882/5d73b65cEcfcb2cc8/fa1c325ee679689b.png
---
> 最近开了一个ShadowSocks的服务器，然而发现很多的童鞋都不会用SS科学上网
  我只能说:"Naive!"

<!--more-->


## ShadowSocks简介
> 
Shadowsocks（中文名称：影梭）是使用Python等语言开发的、基于Apache许可证开源的代理     
软件。Shadowsocks使用socks5代理，用于保护网络流量。在中国大陆被广泛用于突破防火长城（GFW），以浏览被封锁的内容。
From [Wikipedia][1]


那么问题来了，我们要怎么使用SS进行科学上网呢
# 使用ShadowSocks科学上网
## 获取软件
首先，这是一个开源项目，托管在GitHub上,因此我们可以很方便的进行获取源代码和协助开发
> Tips:
  ShadowSocks的作者曾经被请去喝茶，所以我们现在看到的主分支是rm,只要切换到master就好了

Windows的项目: [Widnows][2]
Android的项目: [Android][3]
IOS的项目: [IOS][4]

单击项目上的Release,可以找到最新的构建版本，我们就可以用它来进行我们的科学上网了,当然,你也可以自己下载master分支的源代码进行构建

----------

## Android

可以通过扫描二维码来快速添加服务器

![Android 2code.png][5]

然后返回到主界面 就可以链接服务器了, Android 自身会报警,同意就可以了
![Android warn.png][6]

然后就可以畅享自由的互联网了
![Android done.png][7]

----------

## Windows

### 链接服务器
获取你自己Shadow的账号和密码
如果你的服务器有提供二维码进行输入的话
可以通过状态栏,服务器,扫描二维码来进行扫描![Use2Code.png][8]

这时候,你的SS已经链接到服务器了, 但是我们还需要使用一个客户端, 
链接到SS上，进行科学上网

#### 方法一:使用全局PAC
ShadowSocks自带了从GFW List获取PAC的方法
只需要在状态栏图标右键,PAC,从GFWList更新本地PAC就可以了
![Get PAC.png][9]

这时候默认的IE浏览器就可以访问自由的互联网了

##### 高级篇——自定义PAC
待填坑
#### 方法二:使用浏览器插件
##### Chrome
安装插件 Proxy SwitchyOmega: [谷歌商店地址][10]  [GitHub][11]
![Chrome Proxy SwirchOmega.png][12]

然后打开拓展的配置页
添加一个规则, 这里叫做ShadowSocks
![NewProxy.png][13]

修改它为本地的代理服务器
![RulesDeteil.png][14]

然后点击任务栏图标, 单击刚刚添加的规则就可以科学上网了
![Rules.png][15]

#### FireFox篇
安装附加组件[AutoProxy][16]
![Install AutoProxy.png][17]

然后打开附加组件的控制GUI(重启后)
添加代理服务器
![Edit1.png][18]

端口填入本地端口
![Edit2.png][19]

现在选择全局代理模式就可以跨越防火长城了
![Firefox all proxy.png][20]


  [1]: https://zh.wikipedia.org/wiki/Shadowsocks
  [2]: https://github.com/shadowsocks/shadowsocks-windows/
  [3]: https://github.com/shadowsocks/shadowsocks-android
  [4]: https://github.com/shadowsocks/shadowsocks-iOS
  [5]: https://img10.360buyimg.com/img/jfs/t1/76709/11/9574/232805/5d73c50dE9d01ce99/761f8dd86abe243a.png
  [6]: https://img10.360buyimg.com/img/jfs/t1/78501/15/9574/201343/5d73c50eE7d31bd9c/86e4ebb524064236.png
  [7]: https://img10.360buyimg.com/img/jfs/t1/83747/1/9468/94326/5d73c50fEecdddccd/c0437cea88a88278.png
  [8]: https://img10.360buyimg.com/img/jfs/t1/40925/11/14364/19874/5d73c510E55f89e5d/c876226a9bd6da55.png
  [9]: https://ae01.alicdn.com/kf/Hed9eead8c38c4d1d877ce30e9017bcdch.png
  [10]: https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=zh-CN
  [11]: https://github.com/FelisCatus/SwitchyOmega
  [12]: https://ae01.alicdn.com/kf/U581d2d171c844aef984f87bb16907e08F.png
  [13]: https://img10.360buyimg.com/img/jfs/t1/47533/16/10026/72413/5d73c513E67dc04b2/9ce8e1d4ed816189.png
  [14]: https://img10.360buyimg.com/img/jfs/t1/68736/20/9547/51767/5d73c514Eb7c6d052/6a7ad3ed0b1e9a66.png
  [15]: https://ae01.alicdn.com/kf/U2d302fdb969e4cb0a6aaf5754b78ef3aW.png
  [16]: https://addons.mozilla.org/zh-CN/firefox/addon/autoproxy/
  [17]: https://img10.360buyimg.com/img/jfs/t1/49430/6/10197/96146/5d73c516Edc2d5294/3be393b7ecb19360.png
  [18]: https://img10.360buyimg.com/img/jfs/t1/70057/32/9704/23788/5d73c517Ec69d3d3a/b45596bfe04663e1.png
  [19]: https://img10.360buyimg.com/img/jfs/t1/83944/15/9592/30701/5d73c518Ea12f2e5e/9f8c1148ad4238ee.png
  [20]: https://img10.360buyimg.com/img/jfs/t1/44055/16/14143/14290/5d73c519E5a586d5e/21c68736c5c98689.png
