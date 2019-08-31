---
title: 使用树莓派来收发短信
date: '2018-12-24T15:47:00+08:00'
categories:
- Linux
---
之前手机卡多了很多…… 手机不够放了（逃）
然后正好有个树莓派 就打算使用树莓派收短信


<!--more-->


# 前提准备

- 树莓派 3B 一只
- 华为 E173 一只

使用的系统为官方的 `raspbian-stretch` 最新版本

# 开始安装
首先将移动网卡插上树莓派然后 `lsusb` 可以看到识别出来了

`Bus 001 Device 005: ID 12d1:1c05 Huawei Technologies Co., Ltd. Broadband stick (modem on)`

> 如果发现是 modem off 的时候需要使用 `sudo apt-get install usb-modeswitch` 来切换模式

这时候在 `/dev/` 下面可以看到 ttyUSB 的文件了
```bash
# ls /dev/ttyUSB*
/dev/ttyUSB0  /dev/ttyUSB1  /dev/ttyUSB2
```

然后我们使用
```bash
apt-get install gammu -y
```

来安装 `gammu`

## 配置 gammu
使用 `gammu-config` 来进行配置 我们只需要将 Port 改为 `/dev/ttyUSB2` 即可

> 此处的 ttyUSB2 应该根据个机器来确定

这是时候, 我们使用
```bash
# gammu --identify
Device               : /dev/ttyUSB2
Manufacturer         : Huawei
Model                : E173 (E173)
Firmware             : 21.017.10.00.00
IMEI                 : 3xxxxxxxxxxxxx4
SIM IMSI             : 4xxxxxxxxxxxxx5
```
来验证配置是否成功
此时 可以使用
```bash
# echo "test" | sudo gammu sendsms TEXT 手机号
If you want break, press Ctrl+C...
Sending SMS 1/1....waiting for network answer..OK, message reference=1
```
来测试发送短信

## 将 gammu-smsd 收到的短信转发到 Telegram 上去
我使用 njs 写了个脚本 所以首先安装 nodejs 和 yarn
```bash
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
apt-get install gcc g++ make -y
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
apt-get update -y
apt-get install yarn nodejs -y
```

安装 gammu-smsd
```bash
apt-get install gammu-smsd -y
systemctl start gammu-smsd
systemctl enable gammu-smsd
```
编辑 gammu-smsd 配置文件
```
# cat /etc/gammu-smsdrc

[gammu]
port = /dev/ttyUSB2
connection = at

[smsd]
RunOnReceive=/root/gammu-telegram/index.js
service = files
logfile = syslog
debuglevel = 0

inboxpath = /var/spool/gammu/inbox/
outboxpath = /var/spool/gammu/outbox/
sentsmspath = /var/spool/gammu/sent/
errorsmspath = /var/spool/gammu/error/

# systemctl restart gammu-smsd
```
在 `/root/gammu-telegram` 下创建
`package.json`
```json
{
  "name": "gammu-telegram",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "dependencies": {
    "axios": "^0.18.0",
    "socks-proxy-agent": "^4.0.1"
  }
}
```

`index.js`
```javascript
#!/usr/bin/node
const fs = require('fs')
const path = require('path')
const axios = require('axios')
const SocksProxyAgent = require('socks-proxy-agent')

const botToken = "你的 BOT TOKEN"
const chatId = 你的会话ID

const proxyOptions = 'socks5://127.0.0.1:1080'
const httpsAgent = new SocksProxyAgent(proxyOptions)

const client = axios.create({ baseURL: `https://api.telegram.org/bot${botToken}`, httpsAgent})

// fs.writeFileSync(path.resolve(__dirname + '/env'), JSON.stringify(process.env))
const { SMS_1_NUMBER, SMS_1_TEXT, DECODED_0_TEXT } = process.env

let smsData

if (DECODED_0_TEXT) {
  smsData = DECODED_0_TEXT
} else {
  smsData = SMS_1_TEXT
}

async function main() {
  await client.post('/sendMessage', {
    chat_id: chatId,
    text: "Phone Number: `" + SMS_1_NUMBER + "`\n" + "Tag: #sms\nData: `" + smsData + "`",
    parse_mode: 'Markdown'
  })
}

main()
  .catch(err => console.log(err))
```
然后执行
```bash
yarn install
chmod +x index.js
```

然后每次有新消息的时候就会在 Telegram 上提醒了

## 发送短信
（咕咕咕）
