---
title: Arch Linux 的 Aria2 食用指南
date: 2021-02-07 22:05:28
tags: ['archlinux']
---

最近有些受够了 Chrome 自带的低速下载，尝试配置了一下 Aria2 ，感觉十分良好。将过程记录如下。

<!--more-->

## 后端

### 安装
Aria2 可以在 Arch Linux 源内找到，我们只需要：

```bash
sudo pacman -S aria2
```

### 配置

安装后本来需要进行复杂的配置，但好在已经有人整理出了[较为通用的配置方案](https://github.com/P3TERX/aria2.conf)，我们在这个配置的基础上参照注释进行少量修改即可。

Aria2 默认的配置文件为 ~/.aria2/aria2.conf ，为了方便，我们直接将配置文件放在此处：

```bash
cd ~
git clone https://github.com/P3TERX/aria2.conf
mv aria2.conf .aria2
```

接着打开 ~/.aria2/aria2.conf ，参照配置进行修改（一般只需要修改各个路径和 rpc-secret ）。

### 自动启动

通过以上配置文件，我们为 aria2 开启了 rpc 。

> 在这里解释一下，Aria2 默认的模式是每次下载都需要手动运行一次 aria2 ，下载完成后自动关闭。而开启 rpc 后，aria2 将作为后台应用持续运行，我们可以随时请求后台的 aria2 进行下载。
> 
> 一般情况下推荐使用 rpc ，aria2 的众多前端也基于 rpc 运作。

通过上面的解释，我们知道如今的 aria2 需要后台启动，而我们总不能每次开机都手动执行它一次，于是，我们要为其配置自动启动。 [Arch Wiki](https://wiki.archlinux.org/index.php/aria2) 推荐我们使用 systemd ：

> To use aria2 as a daemon, you can write a systemd user unit.
>
> 要将aria2用作守护程序，您可以编写一个systemd用户单元。

具体是要在 ~/.config/systemd/user 目录下放置一个 aria2.service 服务，内容如下：

```bash
[Unit]
Description=Aria2 Daemon

[Service]
ExecStart=/usr/bin/aria2c

[Install]
WantedBy=default.target
```

接着执行以下命令启动之：

```bash
systemctl --user enable aria2.service
systemctl --user start aria2.service
```

至此完成后端配置。

## 前端

目前有多款流行的 Aria2 前端，如 AriaNg 、 webui-aria2 、 yaaw 等等，使用方法都大同小异。

我的目标是使用它取代 Chrome 的内置下载，因此最终我选择了 Aria2 for Chrome 插件。这个插件可以拦截 Chrome 的下载请求转发到 Aria2 ，并内嵌了 AriaNg 前端界面便于用户管理。在此我主要介绍该拓展。

首先安装拓展进行配置：

![20210207225704](https://rmt.dogedoge.com/fetch/allwens-work/storage/20210207225704.png)

虽然理论上在拓展里进行配置已经足够，但为了保险，我们接着打开拓展内嵌的 AriaNg ，在内部进行同样的配置：

![20210207225930](https://rmt.dogedoge.com/fetch/allwens-work/storage/20210207225930.png)

其他的 Aria2 选项都是不用动的，因为它们使用的是我们之前做的后端配置。

配置完成后打开侧栏，如果侧栏内显示“Aria2状态 已连接”则说明配置成功，大功告成啦！

![20210207230538](https://rmt.dogedoge.com/fetch/allwens-work/storage/20210207230538.png)