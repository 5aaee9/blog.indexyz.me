---
title: 使用 Trix 编辑器的时候上传文件至 sm.ms
date: '2018-11-05T15:31:00+08:00'
categories:
- Web
---
最近再写一个地方的时候用到了 Trix 这个富文本编辑器, 这个编辑器支持直接拖入图片到编辑器窗口来上传图片. 花了点时间写了个 sm.ms 的 兼容接口. 顺路复习了下如何在浏览器发送简单请求

<!--more-->


<script src="https://gist.github.com/Indexyz/4f2ee64db1955d5b23f3016bc44dd141.js"></script>

## 踩的坑
> 请求中的任意XMLHttpRequestUpload 对象均没有注册任何事件监听器；XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问。 [MDN][1]

因为想在上传的时候更新进度 然而 smms 并没有 OPTIONS 允许 CORS 所以在简单请求的时候估计是实现不了进度了

  [1]: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS
