---
title: vscode 粘贴图片自动上传到多吉图床
date: 2020-12-29 11:04:57
tags: ['image']
---

> 该文章的解决方法有些问题，推荐使用[更完美的方法](https://blog.allwens.work/uploadImageToDogedogeViaPicgo/)。

最近有听说一个限时免费的[多吉图床](https://v2ex.com/t/659652)，支持大陆 CDN 和高性能的实时裁剪功能，我也给作者发了一封邮件尝试申请了一下，大概过了两三天通过了。

申请到图床后，当然是配置图片自动上传了。前段时间一直使用 typora 写 md，可以很方便地使用“自定义图片上传命令”对接 picgo-core 完成图片上传，但现在我更多会使用 vscode 来写博客，因此需求变为了 **vscode 粘贴图片自动上传到多吉图床** ，本文记录了我的探索过程和解决方法。

<!--more-->

## 探索

就我自己了解的，picgo-core 基于插件化设计，通过手动编写插件应该很容易完成自定义上传的需求，难点在 vscode 粘贴图片和 picgo 上传命令的对接上。

但很遗憾，经过一圈查找后，我没有找到 vscode 粘贴图片自定义命令的 hook ，当前[ picgo 的 vscode 拓展](https://github.com/PicGo/vs-picgo)也只支持 picgo-core 官方支持的八种图床，并不能支持 picgo-core 的自定义插件。

>  the plugin feature of PicGo-Core is working in progress.

## 解决

所幸，在查找过程中，我发现了另一款 vscode 拓展： [vsc-markdown-image](https://github.com/imlinhanchao/vsc-markdown-image)，这款插件支持自定义上传方法，只需要编写一个异步 JavaScript 函数，实现上传并返回图片链接即可。

官方文档给出的模板如下：

```javascript
const path = require('path');
module.exports = async function(filePath, savePath, markdownPath) {
    // Return a picture access link
    return path.relative(path.dirname(markdownPath), filePath);
}
```

参数说明：

+ filePath => 临时图片文件的路径
+ savePath => 根据设置生成的图片文件名
+ markdownPath => 当前 .md 文件的路径

模板代码是直接返回了图片相对于当前 md 文件的相对路径。

使用自己浅薄的 js 知识，我写了一个简单的上传函数：

``` javascript
const path = require('path');
const got = require('got')
const FormData = require('form-data')
const fs = require('fs')

module.exports = async function (filePath, savePath, markdownPath) {
    // 在此处填写你申请到的token
    const token = 'your token'
    const postURL = `https://www.dogedoge.com/tools/upload/${token}`
    const form = new FormData()
    form.append('file', fs.createReadStream(filePath))
    try {
        const resp = await got.post(postURL, {
            body: form
        })
        let response = JSON.parse(resp.body)
        return response.data.o_url
    } catch (error) {
        console.error(error)
        return path.relative(path.dirname(markdownPath), filePath);
    }
}
```

经过测试，可以正常上传。

## 使用

我已经将代码上传到 [github 仓库](https://github.com/amtoaer/vsc-markdown-image-dogedoge)，使用方法如下：

```bash
# 下载代码
git clone https://github.com/amtoaer/vsc-markdown-image-dogedoge
# 切换到目录
cd vsc-markdown-image-dogedoge
# 安装依赖可选两种方法：
# 1. 使用yarn安装依赖
yarn
# 2. 使用npm安装依赖
npm install
```

接着需要在 vscode 内安装 Markdown Image 拓展，对拓展设置如下：

1. Markdown-image > Base: Upload Method 调整为自定义
2. Markdown-image.DIY: Path 填写 upload.js 文件的绝对路径

![图 1](https://rmt.dogedoge.com/fetch/allwens-work/storage/pic_1609214107230.png)  

至此配置成功，需要上传图片时只需要右键编辑区域并点击粘贴图片选项，或者使用快捷键 `ALT + SHIFT + V` 即可。