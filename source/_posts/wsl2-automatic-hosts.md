---
title: Windows 简单访问 WSL2
date: 2022-11-29 12:12:06
tags: ['wsl']
---

想必很多人在使用 WSL 时都遇到过与 Windows 主机通信的问题。在日常编程中，很常用的场景是在 Windows 中访问部署在 WSL 中的数据库、web 或其它服务等。Windows 本身是会做映射的，比如使用如下命令启动一个 http server：

```python
➜ source git:(master) python -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

这时在 Windows 上访问 `localhost:8000`是可以正常打开的。但在某些其它场景（如数据库连接时），该映射又会失效。可能是因为 WSL 默认只映射了 http 服务的端口？（存疑）

为了解决上述问题，我们需要一种手段来确保 Windows 主机能够轻松访问到 WSL。以下列举三种解决方式，并最后详细描述我的解决方案。

<!--more-->

## 手动查找 WSL 的局域网 IP，并使用该 IP 访问

这种方法最为简单直白，手动找到 WSL 的 IP 访问即可。

在 WSL 中执行`ifconfig`命令，可获得大量输出：

```sh
➜ source git:(master) ifconfig
.....
eth0: .......
        inet 172.21.255.188  .....
.....
```

找到上面`inet`后的 IP 地址，即为 WSL 的局域网 IP。

但 WSL 的 IP 在每次启动时都会变化，如果经常需要访问 WSL 的话该方法并不是一个好选择。

## 连接映射

使用 [HobaiRiku/wsl2-auto-portproxy](https://github.com/HobaiRiku/wsl2-auto-portproxy) 将 WSL 中的所有 TCP 端口映射到 Windows localhost 上（暂不支持 UDP）。

该方法在需要访问 WSL 时下载二进制文件运行即可，不需要额外操作。

## 将 WSL IP 添加到 Windows 的 hosts 文件中

刚刚第一种方法的缺点是 IP 在每次启动时都会变化，这让我们想到了域名。

众所周知域名解析会先查找 hosts 文件，如未找到才会使用 DNS，因此该方法是将 WSL 的 IP 和自定义的域名添加到 Windows 的 hosts 文件，访问时只需访问自己设定的域名。

实施思路类似于 DDNS 的“本机 IP 变化，修改域名解析记录”，需要做到“WSL IP 变化，修改 hosts 文件”，而我们知道 WSL IP 变化其实就是指 WSL 启动，所以最终需求变为了：**WSL 启动时将 IP 写入 Windows 的 hosts 文件**。

从这个需求开始，又衍生出了两个思路

1. 在 Windows 上完成该过程，具体方法是为 WSL 启动的事件绑定自定义任务，该任务通过在 WSL 中执行`ifconfig`加以正则匹配获取 IP，将其写入到 hosts 文件中；
2. 在 WSL 上完成该过程，具体方法是添加开机自启脚本，在该脚本中通过任意手段获取本机 IP，并将其写入到 /mnt/c/Windows/System32/drivers/etc/hosts （因为 Windows 的盘符会自动挂载到 WSL 的 /mnt 下）

### 思路 1

思路 1 的核心是自定义任务，具体的工作流程可以通过很多手段实现。比较简单的是 [shayne/go-wsl2-host](https://github.com/shayne/go-wsl2-host)，该项目会自动添加在 WSL 启动时触发的自定义任务，理论上装上即可用（可惜在我这里没有正常运行，不知道是什么原因）。

### 思路 2

思路 2 的实现较少，因为在过去 WSL 中没有一种行之有效的“开机自启”方法。但该障碍已随着 [WSL 对 systemd 的支持](https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/)而消失不见，因此我手撸了一个[采用这种方式的项目](https://github.com/amtoaer/wsl2-automatic-hosts)。

要使用该项目，对于 Arch Linux：

1. 确保 WSL 能够正常读写 /mnt/c/Windows/System32/drivers/etc/hosts 文件；
2. 下载 [release](https://github.com/amtoaer/wsl2-automatic-hosts/releases) 中的 .pkg.tar.zst 文件并使用`sudo pacman -U`安装；
3. 使用`sudo systemctl enable --now wah.service`启用服务。

对于其它发行版，仅需把上述的第二个流程替换为手动安装：

```sh
install -D -m 755 $srcdir/wah $pkgdir/usr/bin/wah
install -D -m 755 $srcdir/wah.service $pkgdir/usr/lib/systemd/system/wah.service
install -D -m 755 $srcdir/domains $pkgdir/etc/wah/domains
```

其它步骤完全相同。

该项目默认会使用的域名是 arch.wsl，如果需要自定义，可编辑 /etc/wah/domains 文件，对于多个自定义域名，请使用**单个空格**分割。

该项目会自动在 hosts 文件底部添加 WSL 的地址，并在每次 WSL 启动时自动更新：

![image-20221129130815511](https://rmt.ladydaily.com/fetch/allwens-work/storage/image-20221129130815511.png)
