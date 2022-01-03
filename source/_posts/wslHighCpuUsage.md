---
title: 记 wslg 导致的 wsl2 高 CPU 占用的解决方法
date: 2022-01-03 20:16:21
tags: ['wsl','wslg']
---

 自升级 windows 11 以来，我经常在使用 wsl 时遇到电脑风扇狂转，托盘不断闪动的状况，但由于在学校宿舍嘈杂的环境里不是很明显，我一直没有太在意。

最近放假回到家，自己闲来无事刷了刷剑指 offer，惊觉电脑风扇声音竟然如此巨大，遂尝试搜索解决一下。

<!--more-->

## 复现

该情况在我的电脑上很容易复现，只需运行 wsl ，哪怕空载都会有风扇狂转，CPU 占用过高的问题。

## 排查

打开任务管理器查看 CPU 占用，发现两个高 CPU 占用的进程：

1. vmmem
2. windows 音频设备图形隔离

第一个是 windows 的虚拟机进程，优先考虑它。

因为该问题是在升级 windows 11 后才出现的，初步推测是和 windows 11 引入的 [wslg](https://github.com/microsoft/wslg) 有关。

综合以上分析，我使用搜索引擎查询了 `wslg vmmem high cpu usage`，找到了 [microsoft/WSL#6982](https://github.com/microsoft/WSL/issues/6982) 和 [microsoft/wslg#443](https://github.com/microsoft/wslg/issues/443)。

## 结论

根据上述 issue 中的讨论与分析，该问题是由 wslg [weston](https://github.com/microsoft/weston-mirror) 剪切板崩溃导致的，且可能与多显示器的设置变更有关。目前的解决方法有三种：

1. 用于临时解决的魔法快捷键：`Win+Ctrl+Shift+B` （实质是重置了显卡驱动）

2. 如果不依赖 wslg，可创建`<USERPROFILE>\.wslconfig`，写入如下内容以禁用 wslg：

   ```toml
   [wsl2]
   guiApplications=false
   ```

3. 预览版中已经包含[修复](https://github.com/microsoft/weston-mirror/pull/39)，可手动更新至预览版本：[wslg Pre-release 1.0.28](https://github.com/microsoft/wslg/releases/tag/v1.0.28)

