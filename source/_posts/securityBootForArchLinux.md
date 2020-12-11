---
title: "为 Arch Linux 配置安全启动"
date: 2020-12-09 00:39:19
---

最近为自己的Arch Linux配置了安全启动( security boot )，在此记录一下流程以及遇到的问题。

## 实现安全启动

关于安全启动的实现方法，[Arch Wiki](https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface/Secure_Boot#Implementing_Secure_Boot)内有完整的描述，但由于其采用wiki方式写作，对流程描述较为复杂。本节旨在描述实现安全启动的最简方法，跳过更加详细深入的概念性内容。

**注：本文仅描述了为Arch Linux设置安全启动，关于进一步的实现与Windows安全启动共存，请自行阅读Arch Wiki[相应条目](https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface/Secure_Boot#Microsoft_Windows)。**

<!--more-->
### 生成Key

安全启动的实现包括四种Key：

+ Platform Key(PK)
+ Key Exchange Key(KEK)
+ Signature Database(db)
+ Forbidden Signatures Database(dbx)

为了正常使用安全启动，我们至少需要前三种Key。为了方便，我们可以使用脚本自动生成：

```bash
# 后文自动处理内核更新等操作的 sbupdate 工具默认使用位于/etc/efi-keys的Key。

# 为了方便，这里直接创建此目录并在该目录生成Key

mkdir /etc/efi-keys
cd !$
curl -L -O https://www.rodsbooks.com/efi-bootloaders/mkkeys.sh
chmod +x mkkeys.sh
./mkkeys.sh
<随意输入一个嵌入在key中的名字，例如"Secure Boot">
```

### 自动签名

为了在内核、引导更新时自动签名，我们可以使用sbupdate工具。

#### 安装

通过如下命令安装：

```bash
yay -S sbupdate-git
```

#### 配置

安装后，我们需要为其写入配置，指明需要自动签名的内容。

打开/etc/sbupdate.conf，修改配置文件，以下给出我的设置样例以及选项说明：

```bash
# Key所在的目录
KEY_DIR="/etc/efi-keys"
# EFI引导分区位置
ESP_DIR="/boot"
# 生成的单独EFI引导文件位置
OUT_DIR="EFI/Linux"
# 生成的单独EFI引导文件所携带的默认内核参数
CMDLINE_DEFAULT="loglevel=3 quiet nomce rw"
# 需要额外签名的文件，一般包括自己的启动加载器文件
# 理论上内核是自动签名的，但不知道为什么我的vmlinuz-linux-zen不行，因此我也把内核文件写在了这里
EXTRA_SIGN=('/boot/EFI/BOOT/BOOTX64.EFI' '/boot/EFI/systemd/systemd-bootx64.efi' '/boot/vmlinuz-linux-zen')

# 内核单独配置
INITRD["linux-zen"]="/boot/initramfs-linux-zen.img"
# 如果不进行内核参数的配置，默认使用上面的CMDLINE_DEFAULT
CMDLINE["linux-zen"]="root=PARTUUID=db9157b1-bed4-fa44-a715-5c1bfc8bdfb4 quiet nomce loglevel=3 rw"
```

以下对内核单独配置做特别说明：

sbupdate会为每个内核生成一个直接的引导文件（位于${ESP_DIR}/${OUT_DIR}，文件名为${kernel_name}-signed.efi），其中包含了内核参数和临时文件系统的配置（分别对应配置文件中的CMDLINE[${kernel_name}]与INITRD[${kernel_name}]）。

用户可以使用的引导方法有三种：

1. 使用boot loader（例如grub/systemd-boot等）的既有配置进行引导
2. 直接使用该文件进行引导
3. 使用boot loader调用该文件完成间接引导

用户当前使用的往往是第一种方法。在这种方法没有问题的情况下，其实是没有使用这个直接引导文件的必要的。即：

***可以忽略上面的OUT_DIR、CMDLINE_DEFAULT和内核单独配置。***

#### 签名

第一次配置完毕后，需要自己手动执行一下签名指令：

```bash
sudo sbupdate
```

在将来内核更新时，该工具会自动为其签名。

#### 对于systemd-boot自动签名的特殊处理

对于其他引导方法，可能上面配置方法已经足够，但如果你使用systemd-boot作为boot loader，可能还需要进一步的配置。

##### hooks执行顺序的处理

关于这种情况的解释：

一般情况下，我们只需要在更新内核、systemd时使用pacman hook更新签名，但在使用systemd-boot的情况下，我们同样需要在更新systemd时更新引导（即`sudo bootctl update`，该命令会替换/boot/EFI/BOOT/BOOTX64.EFI和/boot/EFI/systemd/systemd-bootx64.efi到最新版本），因此这时hooks的执行顺序成为了一个很大的问题——我们需要确保引导的更新先于签名过程。

想必使用systemd-boot的用户应该了解使用pacman hook进行自动更新的方式（参见[systemd boot - 自动更新](https://wiki.archlinux.org/index.php/Systemd-boot_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0)）。在文档中给出的方法有两种：

1. 安装systemd-boot-pacman-hook（本质是下载了systemd-boot.hook）
2. 自己手动在/etc/pacman.d/hooks/100-systemd-boot.hook写入所给内容。

我们可以看到，sbupdate自带的用于更新签名的hook名称为95-sbupdate.hook，而根据[alpm-hooks文档](https://www.archlinux.org/pacman/alpm-hooks.5.html#_description)所言：

> Hooks are run in alphabetical order of their file name, where the ordering ignores the suffix.
> 钩子按其文件名的字母顺序运行，该顺序忽略后缀。

按照该规则，无论是方法1得到的system-boot.hook还是方法2写入的100-systemd-boot.hook，都无法满足**先于签名hook执行**的需求。我的解决方法是使用方法2，但将文件名修改为90-systemd-boot.hook，使其满足顺序需求。

##### 额外签名boot loader的处理

我们已经解决了第一个障碍，不过不要急，这里还有一个问题。

我们把需要额外签名的boot loader文件写在了sbupdate的EXTRA_SIGN里，但注意到在95-sbupdate.hook中，更新签名使用的命令是`/usr/bin/sbupdate -k`，通过查看sbupdate源码[66-72行](https://github.com/andreyv/sbupdate/blob/master/sbupdate#L66-L72)、[195-205行](https://github.com/andreyv/sbupdate/blob/master/sbupdate#L195-L205)可以得知，-k参数表明sbupdate是被hook调用的，而为了安全原因，使用hook调用sbupdate会跳过EXTRA_SIGN中的内容，这显然不是我们期望的。

解决方法也很简单，打开/usr/share/libalpm/hooks/95-sbupdate.hook，将-k参数去除即可。

### 写入Key到Bios

完成上面的步骤后，已经万事具备，只欠东风了，最后我们只需要将Key写入到Bios。

#### 删除既有Key

Bios默认会自带微软Key，为了写入我们自己的Key，需要将原有的Key删除。

在这一步骤中，需要进入Bios，找到安全启动的证书管理选项，在里面选择删除即可。

#### 写入新的Key

利用`sbsigntools`简化Key的写入过程。

首先我们需要一个这样的目录：

```bash
/etc/secureboot/keys
├── db
├── dbx
├── KEK
└── PK
```

执行如下命令进行创建：

```bash
mkdir -p /etc/secureboot/keys/{db,dbx,KEK,PK}
```

接着将之前位于`/etc/efi-keys`的`*.auth`文件拷贝到对应的目录（没有dbx是正常的）。

最后需要执行如下命令：

```bash
sbkeysync --verbose
sbkeysync --verbose --pk
```

如果有幸没有报错，恭喜你，已经完成了Arch Linux的安全启动配置，这就到Bios中开启安全启动选项并启动吧！

## 结语

在本文中，我罗列出了自己为本机配置安全启动做的所有操作。尽管如此，我并不能保证读者们能够复现我的成功步骤。

如果你在执行文章中的某些步骤时出现差错，推荐阅读Arch Wiki的原文。原文中包括了同一步骤的多种实现方式，且给出了详细的错误处理，比该文更具有代表性。

## 参考

1. [security boot - Arch Wiki](https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface/Secure_Boot)
2. [systemd boot - Arch Wiki](https://wiki.archlinux.org/index.php/Systemd-boot)
3. [sbupdate Readme.md - GitHub](https://github.com/andreyv/sbupdate/blob/master/README.md)
