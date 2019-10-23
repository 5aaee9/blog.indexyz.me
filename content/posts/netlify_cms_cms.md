---
title: 使用 Netlify CMS 作为博客的 CMS
date: 2019-10-23T07:45:52.452Z
updated: 2019-10-23T07:45:52.458Z
tags:
  - Blog
categories:
  - Blog
---
之前从大佬那里知道了 `netlify cms` 正好换成了 Hugo 之后缺一个方便的方法来编辑页面, 所以今天有空配置了一下.

<!--more-->

## 安装

安装十分的便捷, 在想要的路径下面丢进去一个 `index.html` 和一个配置文件就好了

我才用了 GitHub 作为数据源, 所以还需要在 Netlify 中的 site 加上 GitHub 的 app

具体怎么添加可以看官方的 [doc](https://www.netlifycms.org/docs/authentication-backends/#github-backend) 

下面是我站点的 config

```yaml
backend:
  name: github
  repo: Indexyz/blog.indexyz.me
  accept_roles:
    - admin

media_folder: static/img
public_folder: /img

slug:
  encoding: "ascii"
  clean_accents: true
  sanitize_replacement: "_"

collections:
  - name: "posts"
    label: "Post"
    folder: "content/posts"
    create: true
    fields:
      - {label: "Title", name: "title", widget: "string"}
      - {label: "Publish Date", name: "date", widget: "datetime"}
      - {label: "Updated Date", name: "updated", widget: "datetime"}
      - {label: "Tags", name: "tags", widget: "list"}
      - {label: "Categories", name: "categories", widget: "list"}
      - {label: "Body", name: "body", widget: "markdown"}
  - name: pages
    label: Pages
    files:
      - label: Friend Links
        name: friends
        file: content/friends.md
        fields:
          - {label: "Title", name: "title", widget: "string"}
          - {label: "Layout", name: "layout", widget: "hidden", default: "card"}
          - {label: "Type", name: "type", widget: "hidden", default: "links"}
          - label: Links
            name: links
            widget: list
            fields:
              - {label: Name, name: name, widget: string}
              - {label: Description, name: descp, widget: string}
              - {label: URL, name: url, widget: text}
              - {label: Image, name: image, widget: image}
```

可以非常简单的管理生成前的页面

![image.png](https://i.loli.net/2019/10/23/qSOo913CJk5YavB.png)

安装完成之后打开进行 GitHub OAuth 就可以进入到后台了

## 目前不足

在 Post 列表不能对 Post 进行排序, 默认貌似是按文章的名称字典序排序, 这就导致和找一个 Post 会比较难找

不过官方也说 [在做了](https://github.com/netlify/netlify-cms/issues/54)（但是这个 issue 是 2016 年的
