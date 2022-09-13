---
title: 从根本上跳过 Pixel 开机引导验证
date: 2021-01-30 16:01:54
tags: ['Pixel']
---

在使用自带 GMS 的 ROM （如 `Pixel Experience/Evolution X`）时，我们需要完成开机引导来进行一些初始配置。但有时该引导无法离线进行，需要连接 Google 服务器进行进一步操作。这给中国大陆的用户带来了极大的不便——通过开机引导后安装某些工具才能连接到 Google,但无法连接 Google 又不能通过开机引导。本文记录了该状况的起因以及若干解决方法。

<!--more-->

> 2022.09.13 添加：
> 
> 注：该方法可能受网络环境影响，在作者撰写该篇文章时使用家中宽带在无代理的环境下操作成功，不过后续陆续收到读者反馈无效，特此声明。
> 
> 如果该方法无效，也可尝试拔掉手机卡重试（来自 [@梦里水乡](https://t.me/shaoxingtown) 跳过 MIUI EU 引导的方法）。

## 无法离线通过开机引导的原因

先说结论，无法离线开机引导是因为该设备触发了 Google 的 FRP 锁，而这往往是因为在上次清空系统时没有退出自己的 Google 帐号。

在 Google FRP 的介绍中，我们可以看到这样的描述：

> If your device has been lost or stolen, and has been Factory Data Reset in an untrusted environment, this will trigger the FRP lock.
>
> If you want to reset your device to factory default settings in an untrusted environment, ensure that you know your Google account login credentials as you will need it to log in once you have reset your device.

简单来说，如果你的设备在不受信任的环境下恢复了出厂设置会触发 FRP 锁。这时开机需要登陆 Google 帐号进行验证。

## 跳过方法

大部分文章中记录的方法不外乎三种：

1. 使用能够连接到 Google 的网络进行登陆验证。
2. 修改 /system/build.prop ，在该文件中加入一行 `ro.setupwizard.mode=DISABLED`。
3. 强行删除本机的 FRP 。

但我个人感觉这三种方法都不是很理想。

第一种方法需要为电脑/路由器配置局域网代理，手机在连接热点时手动输入代理端口，通过电脑/路由器的代理配置连接 Google。这是最纯正的方法，但缺点是操作方法过于复杂，成本过高。

第二种方法是在 build.prop 中写入停用开机引导的配置。在安卓手机终端没有文本编辑器的情况下，需要在 recovery 里先将该文件复制到 /sdcard 目录下，通过 USB 传到电脑上，加上这行后传回去，再覆盖掉原有的 build.prop，最后给予 755 权限。

论可操作性，这种方法比第一种方法简单，本人在 Android 10 及以下系统中都用的是这种方法。但最近我发现该方法在 Android 11 上有问题——**虽然成功跳过引导开机，但在安装程序时安装器频繁闪退， log 中给出的信息是** `can't install packages while in secure frp`。这说明我们虽然成功跳过了开机引导的验证，但 frp 锁依然存在，影响正常使用。

第三种方法更暴力了，强行删除本机的 FRP 验证机制。个人没有使用过这个方法，不过我觉得虽然这个机制处理起来有些复杂，但根本上是为了保证设备安全，贸然删掉反而不好。

----

接下来是本文重点介绍的方法，也是标题中提到的**从根本上跳过验证的方法**。

既然该验证出现的原因是没有在设备中退出 Google 帐号就恢复了出厂设置，那么只要退出了 Google 帐号， FRP 锁也就自然消失了。所以我们的方法是：**通过云端在设备中退出 Google 帐号**。

1. 访问 [Google 帐号主页](https://myaccount.google.com/)。
2. 找到安全性->您的设备并点击。
    ![20210130170316](https://rmt.dogedoge.com/fetch/allwens-work/storage/20210130170316.png)
3. 点击忘记退出 Google 帐号的设备并点击“退出帐号”按钮
   ![20210130170450](https://rmt.dogedoge.com/fetch/allwens-work/storage/20210130170450.png)
4. 成功退出后会发现手机不会强制登陆 Google 帐号验证了，离线完成开机引导即可。

## 参考文章

+ [what is google frp](https://www.samsung.com/nz/support/mobile-devices/what-is-google-frp/)
+ [how to disable frp lock](https://android.imyfone.com/remove-frp-lock/how-to-disable-frp-lock/)
+ [pixel experience开机验证/跳过谷歌验证的解决办法](https://www.jianshu.com/p/41077ae58a99)
