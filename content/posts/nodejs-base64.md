---
title: Node.js的Base64解码和编码
tags: 
    - Node
categories:
    - Node
date: 2017-08-09 18:01:33
updated: 2017-09-01 18:16:40
thumbnail: https://publish.indexyz.me/images/2017/12/10/nodejs.png
---
最近在写一个~~开车软件~~小程序, 用上了Base64, 然而发现 Node.js 
中并没有相关的内置包. 看了下文档发现其实已经包含在Buffer里了
 


<!--more-->
## 编码
```javascript
const str = new Buffer('Strings');
console.log(str.toString('base64'));
// -> U3RyaW5ncw==
```
## 解码
```javascript
const str = new Buffer('U3RyaW5ncw==', 'base64');
console.log(str.toString());
// -> Strings
```
