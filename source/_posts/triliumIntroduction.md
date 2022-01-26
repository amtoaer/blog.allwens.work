---
title: trilium - 构建你的个人知识库
date: 2022-01-26 16:17:07
tags:
---

最近偶然了解了一款笔记应用：[trilium](https://github.com/zadam/trilium)。正如其README中写的那样：

> Build your personal knowledge base with Trilium Notes.

使用这款软件，可以很方便地构建个人知识库，其独特的设计和极高的拓展性吸引了我。

<!--more-->

## 特点

该软件吸引我的主要有五点：

1. 独特的目录结构设计，单个笔记既能做“文件”又能做“文件夹”，便于组织文档。
2. 笔记支持自定义图标，查看编辑历史，克隆（类似于文件系统的硬链接）等。
3. 所见即所得的富文本编辑器，有着不错的编辑体验。
4. 具有极高拓展性，通过`api`可自定义按钮、页面元素，进行笔记的批量操作等。
5. 前后端分离，轻松实现多端共享、笔记同步。

## 运行

该软件有多种运行模式，如：

1. 本地打开客户端使用，笔记内容仅存储在本地。
2. 部署在服务器后，在本地客户端设置服务器地址，本地和云端笔记内容会定时同步。
3. 部署在服务器后，直接访问服务器地址，通过浏览器前端直接与服务器交互。

个人推荐使用第二种方式，在服务器部署一份用于备份笔记。

## 安装

客户端可前往[trilium - release](https://github.com/zadam/trilium/releases)安装，如对中文翻译有需求，可安装[trilium-translation - release](https://github.com/Nriver/trilium-translation/releases)。

## 部署

一般情况下，我们可以直接使用[压缩包方式](https://github.com/baddate/trilium/wiki/%E5%8E%8B%E7%BC%A9%E5%8C%85%E5%AE%89%E8%A3%85%E6%9C%8D%E5%8A%A1%E5%99%A8)进行服务器部署。流程十分简单，按照步骤来就可以了。

## 展示

以下展示几个典型功能。

### 文档与子文档

右边展示栏显示文档内容与子文档预览。

![image-20220126185121723](https://rmt.ladydaily.com/fetch/allwens-work/storage/image-20220126185121723.png)

### 笔记的克隆

同一笔记可以组织在目录的不同位置。

![image-20220126190133566](https://rmt.ladydaily.com/fetch/allwens-work/storage/image-20220126190133566.png)

### 笔记编辑

成熟的富文本编辑器。

![image-20220126190530905](https://rmt.ladydaily.com/fetch/allwens-work/storage/image-20220126190530905.png)

### 自定义按钮

在侧栏加入自定义按钮，点击后生成今日笔记和其下的“学习”、“娱乐”两个子笔记。

![image-20220126190648895](https://rmt.ladydaily.com/fetch/allwens-work/storage/image-20220126190648895.png)

### 笔记同步

与远程服务器进行笔记内容同步。

![image-20220126190854977](https://rmt.ladydaily.com/fetch/allwens-work/storage/image-20220126190854977.png)

## 高级

如果有更高级的要求，可参考[Trilium：超高自由度的个人知识库（进阶篇）](https://sspai.com/post/59792)。

想要自己编写脚本，可参考官方的 api 文档：

+ 前端：[Class: FrontendScriptApi](https://zadam.github.io/trilium/frontend_api/FrontendScriptApi.html)
+ 后端：[Class: BackendScriptApi](https://zadam.github.io/trilium/backend_api/BackendScriptApi.html)
