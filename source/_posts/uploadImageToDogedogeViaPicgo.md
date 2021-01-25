---
title: vscode 粘贴图片自动上传到多吉图床（新）
date: 2021-01-18 14:37:47
tags: ['image']
---

大概三周前，我写了一篇文章，记录了[ vscode 粘贴图片自动上传到多吉图床的方法](https://blog.allwens.work/uploadImageToDogedoge/)。但在这段时间的使用中，我逐渐发现了它的局限性：似乎由于访问权限的限制，该拓展只能上传剪切板中的临时照片，对于复制的本地照片无法正常上传，具体表现是上传的图片为空。

这促使我尝试其它的方法，并最终决定换用 vs-picgo 。

<!--more-->

## vs-picgo 使用拓展

在第一篇文章中我也有提到， vs-picgo 的 README.md 文件中写它只支持 picgo-core 官方支持的八种图床，插件系统还未完工：

> vs-picgo supports 8 kinds of image hosting services: weibo, qiniu, tcyun, upyun, github, aliyun, imgur and SM.MS, which are supported by PicGo-Core. And the plugin feature of PicGo-Core is working in progress.

正因为这个表述，我才放弃了对其的尝试，有了第一篇文章之后的探索过程。然而这次手动安装并试用后，我发现可能是我理解错了开发者的意思。

仔细查看 vs-picgo 的设置，可以发现一个叫做 Config path 的配置项，说明内容为：**The path to your Picgo-Core configuration. Picgo will use Picgo: Pic Bed if this is not specified.** 也就是说，我们已经可以通过指定 Picgo Core 配置文件的位置来覆盖掉该拓展本身的设置，进而使用 Picgo Core 的第三方插件了， vs-picgo 未完工的可能只是插件系统的图形化配置而已...

既然如此，目的就很明确了。只需要安装 Picgo Core 并进行相应配置，最终在拓展中指定配置文件路径即可。

## 安装 Picgo Core

参考[README.md](https://github.com/PicGo/PicGo-Core)，全局安装只需要：
```bash
yarn global add picgo
# or
npm install picgo -g
```

## 配置 Picgo Core

为了上传到多吉图床，我们需要安装 web-uploader 拓展并配置。
```bash
picgo install web-uploader
picgo config uploader

? Choose a(n) uploader # 选择web-uploader
? API地址 # 填写你的上传地址
? POST参数名 # file
? 图片URL JSON路径(eg: data.url) # data.o_url
? 自定义请求头 标准JSON(eg: {"key":"value"}) # 留空
? 自定义Body 标准JSON(eg: {"key":"value"}) # 留空

picgo use uploader

? Use an uploader # 选择web-uploader
```

配置完成后，可以通过`picgo upload /path/to/an/image`测试能否正常上传，如果上传成功则说明配置没有问题。

## 配置 vs-picgo

最后一步，只需要在 vs-picgo 配置里指定 picgo core 的配置文件。参考官方文档：

> picgo 的默认配置文件为 ~/.picgo/config.json 。其中 ~ 为用户目录。不同系统的用户目录不太一样。
>
> linux 和 macOS 均为~/.picgo/config.json。
>
> windows 则为C:\Users\你的用户名\.picgo\config.json。

例如 GNU/linux 下的配置：

![20210118171109](https://rmt.dogedoge.com/fetch/allwens-work/storage/20210118171109.png)

## 使用 vs-picgo

最后就是用法啦。默认的快捷键如下：

| OS           | Uploading an image from clipboard | Uploading images from explorer | Uploading an image from input box |
| ------------ | --------------------------------- | ------------------------------ | --------------------------------- |
| Windows/Unix | Ctrl + Alt + U                    | Ctrl + Alt + E                 | Ctrl + Alt + O                    |
| Os X         | Cmd + Opt + U                     | Cmd + Opt + E                  | Cmd + Opt + O                     |