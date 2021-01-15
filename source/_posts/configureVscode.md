---
title: 我是如何配置Vscode的
date: 2021-01-15 13:58:09
tags: [vscode]
categories: 便捷使用
---

从18年接触编程至今，已经有两个多年头了。这些日子里，我换过不少次系统（ Windows -> Ubuntu -> Manjaro -> Arch Linux ），写过不少门语言（ C/C++/VHDL/JavaScript/Java/Golang ），但却从来没有更换过我的编辑器，即这篇文章的主角—— Visual Studio Code 。

<!--more-->

它吸引我的点主要有以下三个：

1. 跨平台
2. 占用低
3. 插件化设计

其中尤其让我惊艳的是二、三点。

作为使用 electron 开发的程序，它摆脱了“网页套壳”的弊端，在低占用的同时甚至达到了和原生程序基本一致的效果。

除此之外，优秀的插件化设计也给了用户更多的自定义空间。通过安装插件，我们可以拓展它的主题、图标、额外功能以及编程语言支持。

> ~~*甭管写啥， vscode 一把梭！*~~

下面贴一张我的 vscode 截图：
![](https://rmt.dogedoge.com/fetch/allwens-work/storage/pic_1610690634182.png)  

## 配置

在使用它的过程中，我尝试了许多配置，试用了许多插件，如今基本有了固定的配置方法和插件列表。写这篇文章除了用作记录，也算是为读者做一个推荐吧。

### 字体

字体是编辑器的灵魂，赏心悦目的字体可以大幅提升打码热情。我个人推荐三款字体：

1. Sarasa Gothic

又被称为更纱黑体，是我个人最喜欢的字体，也是我 vscode 编辑区域使用的字体。比起后面纯粹的编程字体，它的独特之处是使用思源黑体补齐了 CJK 字库，使字体整体更加协调。（它的纯英文版字体叫做 Iosevka ）

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/preview-all.png)

2. JetBrains Mono

JetBrains 出品，必属精品！（大雾）

能够作为 JetBrains 家所有 IDE 的默认字体，它的优秀毋庸置疑。我将其用作 vscode 内置终端的字体。

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/JetBrainsMonoTypeface.gif)

3. Fira Code

是我第一个长久使用的字体，似乎也是 vscode 官方推荐的字体，字型和连字符都很有特点。

这款字体陪伴了我半年左右，用久了可能会觉得有些花哨，但不失为一个优秀的编程字体。

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/stylistic_sets.png)

### 主题

主题方面，主观上只推荐 atom 的两款配色主题：

1. Atom One Light

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/preview.png)

2. Atom One Dark

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/preview%20(1).png)

### 插件

拓展语言支持的插件没啥好说的，在这里只记录一些实用的小插件。

1. Error Lens

默认 vscode 的语法检测会将代码提示显示在下栏的“问题”版块，不够清晰明了。使用该插件可以将问题详情显示在编辑区域的对应行后，方便查看。

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/pic_1610714848157.png)  


2. GitLens

该拓展用于拓展 vscode 的 Git 支持，功能十分齐全，我个人用到的功能主要有：
  
  +  commit 间的代码对比

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/pic_1610714962427.png)  

  + 某行代码来自哪次 commit 的提示

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/pic_1610715086159.png)  


3. SQLTools

数据库拓展，使用其和对应驱动可以在 vscode 内部连接数据库，支持查看数据库、表和执行 SQL 语句等功能。

4. Todo Tree

可以对特定的注释进行高亮，同时提供全局的注释索引。支持自定义设置注释内容和高亮颜色。

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/pic_1610715678658.png)  

5. WakaTime

用于统计 vscode 编辑文件的类型、时间等信息，并将其与 wakatime 云端同步，时间会显示在 vscode 的底栏，如图所示：

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/pic_1610715808840.png)  

该拓展可以将用户的编程时间统计下来，与 WakaTime API 配合使用可以实现一些很有趣的效果，比如通过 [github readme stats](https://github.com/anuraghazra/github-readme-stats) 将 wakatime 编程时间同步到 Github Profile 上。

![](https://rmt.dogedoge.com/fetch/allwens-work/storage/pic_1610716778436.png)  
