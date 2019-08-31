---
title: CurseForge 整合包格式研究
tags: 
    - Minecraft
    - CurseForge
categories:
    - Minecraft
date: 2017-08-16 04:14:35
updated: 2017-09-01 18:10:47
thumbnail: https://publish.indexyz.me/images/2017/12/10/forge.png
---
最近在写启动器关于curseforge整合包的导入 然后研究了下如何把curseforge的整合包转换为classic的整合包

<!-- more -->

首先来看`manifest.json`中的对于filed和定义
```json
{
    "manifestType": "minecraftModpack",   // manifest类型
    "manifestVersion": 1,  // manifest版本
    "name": "Test Package",  // 整合包名称
    "version": "1.0.0",      // 整合包版本
    "author": "Indexyz",     // 整合包作者
    "overrides": "overrides"，  // 重载文件夹
    "minecraft": {  // Minecraft相关信息
        "version": "1.10.2", // 关于Minecraft的版本
        "modLoaders": [{ // mod加载器 定义类似forge版本的信息
             "id": "forge-12.18.3.2422",    // 定义forge版本
             "primary": true    // 是否主要
        }]
    },
    "files": [{      // 定义mods文件
        "projectID": 228404,    // curseforge上的项目ID
        "fileID": 2392124,      // curseforge上项目的文件ID
        "required": true        // 是否需要
    }]
}
```

## 下载地址
curseforge的mod下载地址是如下格式
```
minecraft.curseforge.com/projects/{项目名称}/files/{文件ID}/download
```

### 关于项目名的获取
目前有个curseforge的API
访问
```
https://curse.mcdev.se/project/id/{项目ID}
```
可以在返回的payload中获取到项目的base url
在现在 你可以通过项目的Id去 
```
https://curse.mcdev.se/files/project/{项目ID}
```
获取到全部的文件列表
## Overrides
整合包的Overrides是一个文件夹 由mainfest的 overrides定义 在相应的文件夹中 主要是将里面内容覆盖(加入) .minecraft 当中

其中也有可能包含不在curseforge控制下的mod