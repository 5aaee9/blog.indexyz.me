---
title: OpenVPN安装和面板的对接
tags: 
    - Linux
    - Proxy
    - OpenVPN
categories:
    - Linux
date: 2017-08-09 17:57:17
updated: 2017-09-01 18:17:42
thumbnail: https://publish.indexyz.me/images/2017/12/10/openvpn.png
---
> OpenVPN是一个用于创建虚拟专用网络加密通道的软件包，最早由James Yonan编写。OpenVPN允许创建的VPN使用公开密钥、电子证书、或者用户名／密码来进行身份验证。
[Wikipedia](https://zh.wikipedia.org/wiki/OpenVPN)

话说看到了[乌云](http://www.wooyun.org/bugs/wooyun-2015-0165733)上一个关于OpenVPN免流的Bug,同时最近在写[Project 
Evplex](http://www.evplex.xyz),打算给他加上关于面板的对接和用户认证还有流量统计什么的.
<!--more-->

## 安装OpenVPN
修改网络支持转发
```bash
# 关闭SELinux
setenforce 0
vi /etc/sysctl.conf
# 修改参数 net.ipv4.ip_forward = 1 如果默认是空内容，请自行加上
sysctl -p
# 修改防火墙参数
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
iptables -A INPUT -p TCP --dport 443 -j ACCEPT
iptables -A INPUT -p TCP --dport 8080 -j ACCEPT
iptables -A INPUT -p TCP --dport 22 -j ACCEPT
iptables -t nat -A POSTROUTING -j MASQUERADE
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
service iptables save
service iptables restart
```
###安装本体
```bash
yum install epel-release -y
yum update -y
# 安装依赖
yum install -y wget openssl openssl-devel lzo lzo-devel pam pam-devel automake 
pkgconfig
yum install -y openvpn
cp /usr/share/doc/openvpn-*/sample/sample-config-files/server.conf  /etc/openvpn

# 安装EasyRSA
wget --no-check-certificate https://ftp.iinde.xyz/Useful/openvpn/EasyRSA.tar.gz
tar zxvf EasyRSA.tar.gz -C /etc/openvpn/
rm -f EasyRSA.tar.gz
cd /etc/openvpn/EasyRsa/
source vars
./clean-all
./build-ca
./build-key-server server
# 实际上，对于用户名/密码认证机制来说，./build-key user01可以省略掉
./build-key user01
./build-dh
cd ..
openvpn --genkey --secret static.key
```
###启动OpenVPN服务
```bash
service openvpn start
```
###配置文件例子
```cfg
port 443
proto tcp
dev tun
ca /etc/openvpn/EasyRsa/keys/ca.crt
cert /etc/openvpn/EasyRsa/keys/server.crt
key /etc/openvpn/EasyRsa/keys/server.key
dh /etc/openvpn/EasyRsa/keys/dh2048.pem
auth-user-pass-verify /etc/openvpn/verify.sh via-env
client-cert-not-required
username-as-common-name
script-security 3 system
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist /etc/openvpn/ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120 
tls-auth /etc/openvpn/static.key 0  
comp-lzo
max-clients 12
persist-key
persist-tun
log /etc/openvpn/openvpn.log
log-append /etc/openvpn/openvpn.log
verb 3
```


<!--more-->

## 免流部分
### 安装MProxy进行流量转发
```bash
# 自编译安装
yum install git gcc -y
git clone https://github.com/Indexyz/mproxy.git mproxy
cd mproxy
gcc -o mproxy mproxy.c
# mproxy/mroxy 就是生成的可执行文件了

# 当然 你也可以选择我已经编译过的文件
curl -skSL -o mproxy https://ftp.iinde.xyz/Useful/openvpn/mproxy;chmod +x mproxy
```
#### 运行MProxy
```bash
./mproxy -l 8080 -d
```
### 安装Squid进行流量转发
```bash
cd /etc/squid/
rm -f ./squid.conf
vi squid.conf
# 编辑你自己的配置文件
chmod 0755 /etc/squid/squid.conf
squid -z
squid -s
```
配置文件例子
```
acl SSL_ports port 443
acl Safe_ports port 80
acl Safe_ports port 21
acl Safe_ports port 443
acl Safe_ports port 70
acl Safe_ports port 210
acl Safe_ports port 1025-65535
acl Safe_ports port 280
acl Safe_ports port 488
acl Safe_ports port 591
acl Safe_ports port 777
acl CONNECT method CONNECT
via on
request_header_access X-Forwarded-For deny all
request_header_access user-agent  deny all
reply_header_access X-Forwarded-For deny all
reply_header_access user-agent  deny all
http_port 80
http_access allow  all
access_log /var/log/squid/access.log
visible_hostname Indexyz Technology Inc.
cache_mgr Incloud_Everythings
```
## OpenVPN 配置文件
### 使用MProxy的例子
```
setenv IV_GUI_VER "de.blinkt.openvpn 0.6.17" 
machine-readable-output
client
dev tun
proto tcp
connect-retry-max 5
connect-retry 5
resolv-retry 60
remote wap.10086.cn 80
http-proxy-retry
http-proxy-option EXT1 POST http://wap.10086.cn
http-proxy-option EXT1 Host wap.10086.cn
http-proxy-option EXT1 Host: wap.10086.cn / HTTP/1.1
http-proxy-option EXT1 CONNECT
http-proxy {your server ip} 8080

resolv-retry infinite
nobind
persist-key
persist-tun
push route 114.114.114.114 114.114.115.115

<ca>
Enter Your CA here
</ca>
key-direction 1
<tls-auth>
Enter your static key here
</tls-auth>
auth-user-pass
ns-cert-type server
comp-lzo
verb 3
```
### 使用Squid的例子
```
setenv IV_GUI_VER "de.blinkt.openvpn 0.6.17" 
machine-readable-output
client
dev tun
connect-retry-max 5
connect-retry 5
resolv-retry 60
http-proxy-option EXT1 "POST http://rd.go.10086.cn" 
http-proxy-option EXT1 "GET http://rd.go.10086.cn" 
http-proxy-option EXT1 "X-Online-Host: rd.go.10086.cn" 
http-proxy-option EXT1 "POST http://rd.go.10086.cn" 
http-proxy-option EXT1 "X-Online-Host: rd.go.10086.cn" 
http-proxy-option EXT1 "POST http://rd.go.10086.cn" 
http-proxy-option EXT1 "Host: rd.go.10086.cn" 
http-proxy-option EXT1 "GET http://rd.go.10086.cn" 
http-proxy-option EXT1 "Host: rd.go.10086.cn"
http-proxy {your server ip} 80

remote {your server ip} {your server port} tcp-client
resolv-retry infinite
nobind
persist-key
persist-tun
push route 114.114.114.114 114.114.115.115

<ca>
Enter your CA here
</ca>
key-direction 1
<tls-auth>
Enter your static key here
</tls-auth>
auth-user-pass
ns-cert-type server
comp-lzo
verb 3
```
## 用户认证和面板对接
详见[这里](https://blog.iinde.xyz/index.php/archives/33/)

