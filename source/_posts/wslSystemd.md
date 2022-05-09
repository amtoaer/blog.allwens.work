---
title: 在 wsl2 中完美启用 systemd
date: 2022-05-09 10:57:53
tags: ['wsl','linux']
---

在 wsl2 中启用 systemd  的方法至少有如下三种：

1. [genie](https://github.com/arkane-systems/genie)
2. [subsystemctl](https://github.com/sorah/subsystemctl)
3. [wsl-distrod](https://github.com/nullpo-head/wsl-distrod)

本文主要介绍第三种方法，并在后文介绍选用第三种方法的优势。

<!--more-->

## 安装方法

安装分为两个选项：

1. 安装一个新的带有 systemd 的发行版
2. 为现有的发行版增加 systemd 支持

以下分别介绍。

### 安装一个新的带有 systemd 的发行版

1. 下载并解压[distrod_wsl_launcher](https://github.com/nullpo-head/wsl-distrod/releases/latest/download/distrod_wsl_launcher-x86_64.zip)，执行解压出的二进制文件

2. 跟随引导安装一个新的发行版

3. [可选项]使用如下命令使你的发行版在 windows 启动时自动运行：

   ```sh
   sudo /opt/distrod/bin/distrod enable --start-on-windows-boot
   ```

### 为现有的发行版增加 systemd 支持

1. 下载并运行最新的安装脚本：

   ```sh
   curl -L -O "https://raw.githubusercontent.com/nullpo-head/wsl-distrod/main/install.sh"
   chmod +x install.sh
   sudo ./install.sh install
   ```

2. 在你的发行版中开启 distrod

   有两个选择：

   1. 在 windows 启动时运行你的发行版：

      ```sh
      /opt/distrod/bin/distrod enable --start-on-windows-boot
      ```

   2. 否则：

      ```sh
      /opt/distrod/bin/distrod enable
      ```

      注：接下来仍然可以通过执行`/opt/distrod/bin/distrod enable --start-on-windows-boot`让发行版跟随 windows 启动。

3. 重新启动你的发行版

   关掉你的 wsl 窗口，在 PowerShell 中执行如下命令：

   ```sh
   wsl --shutdown
   ```

   接下来重新打开一个新的 wsl 窗口，你的 shell 将在 systemd 会话中运行。

## 与其它方法的对比

在[官方文档](https://github.com/nullpo-head/wsl-distrod#how-distrod-works)中，Distrod 描述了其工作原理：

> 简而言之，Distrod 是一个二进制文件，它会创建一个简单的容器，将 systemd 作为 init 进程，并在该容器中启动您的 wsl 会话。为了做到这一点， Distrod 做了以下事情：
>
> 1. 修改具体发行版的 rootfs，以使其与 wsl 和 systemd 兼容。
>    1. 修改 systemd 服务，使其与 wsl 兼容。
>    2. 把`/opt/distrod/bin/distrod`和其它资源放进 rootfs。
>    3. 把 Distrod 二进制文件注册为登录 shell。
> 2. 当 Distrod 作为登录 shell 被 wsl 的 init 进程启动时，Distrod：
>    1. 在简单的容器中启动 systemd
>    2. 在这个容器中启动你实际的 shell
>    3. 在 systemd 会话和 wsl 交互环境间建立桥梁。

事实上 Distrod 和 genie、subsystemctl 的工作原理是类似的：创建一个容器，在容器内以 pid 1 启动 systemd 并使用 shell。在其[官方文档](https://github.com/nullpo-head/wsl-distrod/blob/main/docs/references.md#run-distrod-as-a-standalone-one-shot-command)中也有提过：

> 把 Distrod 作为一个独立的一次性命令：**在这种情况下， distrod 的工作方式就像 genie 和 subsystemctl。**

但是，Distrod 在自启动方面做了更多的工作。通过上述操作，Distrod 做到了：

1. 安装并启用后，启动 wsl 会自动启动 systmed。
2. 添加`--start-on-windows-boot`参数后，Distrod 会注册一个 windows 任务，使得 wsl 在 windows 开机时就会运行。

对比其它方法需要手动编写 shell 脚本实现自动启动，安装更简单、使用更方便的 Distrod 无疑更被用户青睐。

## 结语

在安装、使用过程中遇到任何问题，可以参考[官方资料](https://github.com/nullpo-head/wsl-distrod/blob/main/docs/references.md)。

文章中发现任何错误欢迎在评论区指出！~
