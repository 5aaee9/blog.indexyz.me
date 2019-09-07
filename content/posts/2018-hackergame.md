---
title: 2018 LUG HackerGame 解题笔记
date: '2018-10-10T00:16:00+08:00'
categories:
- CTF
---
最近玩了一下 LUG 的 2018 HackerGame, 下面是解题笔记

推荐阅读:

[Coxxs 大佬的 Blog](https://coxxs.me/879)
[官方 writeups](https://github.com/ustclug/hackergame2018-writeups)
[Soha 大佬的](https://soha.moe/post/ustc-ctf-2018-writeup.html)

<!--more-->

## 签到题
这题没啥好讲的, 只要把 maxlength 改成相应的长度就好了

## 猫咪问答
拿着题目去问 Google / 百度

## 游园会的集章卡片
拼图游戏, （我用画图拼的）

## Word 文档
将附件下载之后后缀改为 zip 然后解压出 flag.txt
(因为 docx 就是 zip + XML 之前 LTT 还是哪里都有做过一期科普)

## 黑曜石浏览器
搜索得到官网 https://heicore.com/ (看 whois 是八月份注册的)
然后用 chrome 打开 dev tools 会直接 crash 掉浏览器
直接用 Firefox 打开就好了 (或者 curl 一下)
崩掉浏览器应该用的是
```
setInterval(() => {
    var r = /./;
    r.toString = function () {
        eval("console.clear();");
        document.documentElement.innerHTML = '';
    };
    if(navigator.userAgent.includes('Safari') && !navigator.userAgent.includes('Edge'))
        console.log('%c', r);
}, 50);
```

然后搜索找到 `isLatestHEICORE` 这个函数 用其中的 UA 请求 flag.txt 就拿到了 flag

## 回到过去
下载附件 去掉最上面的 `q` 和 `ed`
然后根据 `ed` 的 man page, 在倒数第三行加入 `w flag.txt`, 然后将 input pipe 进 ed, 然后生成的 flag.txt 就是 flag 了

## 我是谁
### 第一关
请求页面 会发现 status code 是 `418 I'm a teapot`, 回答你是 teapot 就可以拿到 flag 了

### 第二关
提示换种请求方式, 使用 POST 得到
```
<p>The method "POST" is deprecated.</p>
<p>See RFC-7168 for more information.</p>
```
然后根据 RFC-7168 使用 BREW 请求服务器 拿到 flag
> 这里据大佬将可以使用 curl 进行请求, 我直接使用 nc 发了请求 233

## 猫咪克星
就是一个自动应答, 我用 nodejs 写了一个脚本来完成
```javascript
const process = require('child_process')

const nc = process.spawn('nc', ['202.38.95.46', '12009'])

const calc = (payload) => {
    const t = process.spawnSync('/usr/bin/python', [], {
        input: `print(${payload}); exit()`
    })

    return t.stdout.toString().trim()
        .replace(/\r/g, '')
        .replace(/\n/g, '')
}

nc.stdout.on('data', chunk => {
    let data = chunk.toString().trim()

    if (data === 'You have only 30 seconds') {
        return
    }

    console.log(`<== ${data}`)

    data = data.replace(/exit\(\)/g, "0")
        .replace(/sleep\(100\)/g, "sleep(0)")
        .replace(/__import__\('os'\).system\('find ~'\)/g, '0')
        .replace(/\\x1b\\x5b\\x33\\x3b\\x4a\\x1b\\x5b\\x48\\x1b\\x5b\\x32\\x4a/g, '')
        .replace(/print\(''\)/g, '0')

    console.log(data)
    const result = calc(data)

    console.log(`===> ${result}`)
    nc.stdin.write(result)
    nc.stdin.write('\r\n')
})
```
> 期间会运行 sleep 和 find 啥的指令, 有大佬直接用 py 实现的时候 eval 限定了 import 内容可以绕过这部分

## 猫咪电路
就是土球写的一个 Minecraft 地图, 照着做就好了

## 猫咪银行
这题按正常流程是无法拿到 20 CTB 的, 但是在 理财产品 A1 中有 integer overflow 的问题, 在时间处输入一个较大的数字 (我使用的是 `154748364800000000`) 可以看到时间溢出了, 但是金额还是正数. 这时候取回然后换到 20 CTB 购买 flag 就解决了

## 猫咪遥控器
根据移动轨迹做出点图, 然后使用 `Desmos` 进行画图得到 flag
下面是我使用的代码

```javascript
const origin = require('fs').readFileSync('seq.in').toString()
let x = 0, y = 0
let points = [
    [0, 0]
]
let last = 'D'

for (let char of origin) {
    points.push([x, y])

    switch (char) {
        case 'D' : {
            y -= 1
            break
        }
        case 'R' :{
            x += 1
            break
        }
        case 'L': {
            x -= 1
            break
        }
        case 'U': {
            y += 1
            break
        }
    }
}

let result = ''

for (pair of points) {
    result += `(${pair[0]}, ${pair[1]}), `
}

require('fs').writeFileSync('seq_result', result)
```

## 猫咪和键盘
打开 typed_printf.cpp 可以看到每列都被打乱了. 根据 main 函数等地方整理之后得到原始代码
使用 linux 机器上的 gcc 进行编译, 编译方式在开头有写 (`g++ -std=c++17 typed_printf.cpp`) 然后运行得到 flag
