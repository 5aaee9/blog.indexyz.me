---
title: Python实现清屏
tags: 
    - Python
categories:
    - Python
date: 2017-08-09 18:03:29
updated: 2017-09-01 18:16:21
---
刚刚看busybox实现的时候看到了一个有意思的东西, 实现了清屏, 
有时候Python又要写一些控制台应用程序的时候可能要用到, 于是放在了这里


<!--more-->

```python
import sys
sys.stdout.write("\033[H\033[J")
```

解释:
这是ANSI的控制字符
\033[H - 光标复位
\033[J - 清除屏幕
