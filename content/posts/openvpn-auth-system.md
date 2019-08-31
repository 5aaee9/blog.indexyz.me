---
title: 关于OpenVPN的用户认证的研究
tags: 
    - Proxy
    - Linux
    - Python
    - OpenVPN
categories:
    - Linux
date: 2017-08-09 17:58:46
updated: 2017-09-01 18:17:11
thumbnail: https://publish.indexyz.me/images/2017/12/10/openvpn.png
---
> 最近在研究OpenVPN 
 然后想把自己的Evplex和ss-panel接入OpenVPN认证，于是就开始研究了

<!--more-->

## 1. /etc/server.conf
```bash
# 添加下列行 会调用verify.sh进行认证并且不要求CA证书
auth-user-pass-verify ./verify.sh via-env
client-cert-not-required
username-as-common-name

# 添加下列行会调用connect和disconnect进行对链接的传入和传出进行控制
script-security 3 system
client-connect ./connect.sh
client-disconnect ./disconnect.sh
```
### 对disconnect.sh和connect.sh 的参数解释
| Value Name        | Mean           |
| ------------------| --------------:|
| $bytes_received   | 收到的流量      |
| $bytes_sent       | 发送的流量      |
| $trusted_ip       | 链接的IP        |
| $trusted_port     | 链接的端口      |
| $ifconfig_pool_remote_ip|链接者的IP|
|$common_name| 用户名|

## 对面板的用户获取
大家知道 ss-panel的V3有webapi, Evplex当然也有webapi
Evplex的WebApi是为了接入更多类型的节点和方便不使用mysql作为数据库的用户


```python
Main.py:
---
import Config
import urllib
import json
import time

def main():
    API_obj  = urllib.urlopen(Config.API_URL + "users?key=" + Config.API_PASS)

    API_json = json.load(API_obj)
    user_list = []
    if API_json["ret"]:
        for item in API_json["data"]:
            if item["enable"]:
                user_list.append(item["user_name"] + " " + item["passwd"])
    with open(Config.STOAGE_PATH, "w") as file:
        file.write("\n".join(user_list))
        
if __name__ == "__main__":
    while True:
        main()
        time.sleep(Config.RSYNC_TIME * 60)
---

Config.py
---
API_URL     = "http://127.0.0.1:8000/api/"
API_PASS    = "Pleasechangeit"

STOAGE_PATH = "users"
RSYNC_TIME  = 0.1                       # min
---
```
