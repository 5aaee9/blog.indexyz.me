---
title: 小米MiFlash出现remote partition table doesn't exist的解决方法
tags: 
    - Android
    - Phone
categories:
    - Android
date: 2017-08-09 18:25:41
updated: 2017-09-01 18:13:01
---
这几天在闲鱼收了个小米2S, 可以进充电界面但是不能进系统 

然后果断收了啊 收过来一看可以进Fastboot 那就应该可以刷好了

架好 MiFlash 然后刷MIUI5的时候却发现 

`Failed: remote: partition table doesn't exist`'


<!--more-->

这应该是分区的关系 在某度搜索了一下发现貌似是分区表炸了 

然而我并没有一个好的分区表 这时候看了下刷机包文件中有一个`gpt_both0.bin` 

于是加入

`fastboot %* flash partition "%~dp0images\gpt_both0.bin" || @echo "Flash 
partition" && exit /B 1`

在刷机的 `flash_all.bat` 打开 MiFlash 重新刷机 搞定
