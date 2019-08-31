---
title: 提取 macOS 上的苹方字体并在 Windows 上安装
date: '2019-05-29T17:49:00+08:00'
categories:
- Windows
---
目前的 MBP 配置太差了 每次 `webpack reload` 都要等一会, 所以想使用 Windows 作为开发机. 但是 `Lunacy` 对没有的字体并不会自动 fallback 到目前的字体上, 然后打开 sketch 的文件都是一个个的大 X. 所以想将 macOS 上的字体提取出来在 Windows 上使用


<!--more-->


## 开始
首先我们需要 Adobe 的 `otc2otf`, 可以在 [此处][1] 获取. 然后我们需要 `PingFang.ttc`, 在 Font Book 中找到字体并在 Finder 中打开, 复制出来就好了

接下来我们需要安装 [Font Tools][2]

在确保安装了 Brew 的情况下我们可以直接

```bash
brew install fonttools
```

## 转换
打开 `PingFang.ttc` 的文件夹, 运行 otc2otf
```bash
python2 otc2otf.py -w PingFang.ttc
rm -f PSNameUndefined.otf
```
得到的 `otf` 文件并不能在 Windows 上使用, 会提示不是一个有效的字体文件

我们需要使用 `Font Tools` 进行转换

```bash
#!/bin/bash

for item in `ls *.otf`; do
    echo "Editing: "$item
    ttx -t cmap $item
    FILENAME="$(echo $item | sed 's/\.[^.]*$//')"
    sed -i '' 's/platformID="0" platEncID="3"/platformID="3" platEncID="1"/g' $FILENAME.ttx
    sed -i '' 's/platformID="0" platEncID="4"/platformID="3" platEncID="10"/g' $FILENAME.ttx
    ttx -b -m $item $FILENAME.ttx
done
```

转换完成之后会有 `字体名#1.otf` 的文件, 这个就能被 Windows 所识别了

## 参考文章
- [蘋方移植 – 汚音屋 × UJAM – Medium][3]
- [CMap 表相关修改技术简要指南][4]


  [1]: http://blogs.adobe.com/CCJKType/files/2014/01/otc2otf.py
  [2]: https://github.com/fonttools/fonttools
  [3]: https://medium.com/ujam/trans-pingfang-a-windows-font-d058aebb2550
  [4]: https://darknode.in/font/cmap-modify-tutorial/
