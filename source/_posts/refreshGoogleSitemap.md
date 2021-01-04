---
title: 博客更新文章后自动通知 Google 更新站点地图
date: 2021-01-04 21:57:37
tags: ['vercel','blog']
---

博客上一篇文章已经是2020年12月29日的了。从发布到今天，我每天都尝试在 Google 搜一下它，却发现一直没有收录。

> ~~暴露了自己是一个高强度自搜人的事实XD~~

手动到 Google Search Consoles 页面去查看了一下，显示站点地图上次更新还停留在12月19日...由此引出了今天的问题：**如何让 Google 自动更新站点地图**。

<!--more-->

## 探索过程

首先需要找到 Google 更新站点地图的方法，通过查阅[文档](https://developers.google.com/search/docs/advanced/sitemaps/build-sitemap?hl=zh-cn)可以得知：

> 使用“ping”功能请求我们抓取站点地图。发送如下所示的 HTTP GET 请求：
> http://www.google.com/ping?sitemap={complete_url_of_sitemap}
> 例如：
> http://www.google.com/ping?sitemap=https://example.com/sitemap.xml

所以思路十分清晰，只需要在博客部署完成后手动 GET 这个地址即可。

## 解决方法

### Github Actions

对于使用 Github Action 来部署博客的用户，操作流程十分简单，只需要在 yml 文件部署命令的后面填上一行 `curl http://www.google.com/ping?sitemap=<your sitemap>` 便可以解决。

### Vercel

因为 Github Pages访问速度较慢，我使用的是 Vercel ，而它并没有提供一个类似于 After Deployment Command 的功能。这使我没有办法精确 Hook 它的部署过程。

于是，我去请教了 NEU LUG 群的群友，而群友也给出了一个简单粗暴的解决方法：管它 Hook 不 Hook ，定时更新一次就完了！

![图 1](https://rmt.dogedoge.com/fetch/allwens-work/storage/-112b9cb87f4b5b35.png)

本来我也确实打算这样干了，但考虑到我博客的更新频率，以及 Google 文档中写出的：

> 请仅在新建或更新站点地图时向 Google 发送站点地图相关提醒。如果站点地图无任何变更，请勿多次向我们提交或 ping 站点地图。

为了避免搞出事情，我还是抛弃了这种方法。但这给了我一个思路，即：**不需要盲目追求 Hook ，只要达到效果就可。**

思考后，我想到了这样的方法：

在更新博客（仓库 master 分支 Push ）时，Vercel 构建和 Github actions 同时触发。Github actions 检测 commit message的内容，如果包含“更新博文”这四个字便触发流程：
1. 等待 3min（我博客在 Vercel 的构建要 1min~2min ，为了增加容错率写成 3min ）
2. 发送 GET 请求通知 Google 更新站点地图。

具体 yml 流程文件如下：

```yml
name: ping-google
on:
  push:
    branches:
      - master
jobs:
  ping-google:
    if: "contains(github.event.head_commit.message, '更新博文')"
    runs-on: ubuntu-latest
    steps:
      - name: wait for build
        run: sleep 3m
      - name: ping google to update sitemap.xml
        run: curl http://www.google.com/ping?sitemap=https://blog.allwens.work/sitemap.xml
```

经测试，工作良好。

*这样处理缺点也是存在的：Github actions 对免费用户有着每月 3000 分钟的使用限额。虽然我们使用 sleep 命令休眠三分钟时什么都没干，但这段时间仍然会被计入你的 Github actions 使用时间中。（不过反正这些时间也用不完，我博客更新频率又很低，代价是可以接受的233）*

## 参考资料

1. Github Actions 检测 commit message ：

   [Support [skip ci] out of box with github actions](https://github.com/actions/runner/issues/774)