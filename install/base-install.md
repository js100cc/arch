# 基本安装

**我最初是看了 3 天的 wiki，然后一次性在物理机上安装了 Arch Linux，所以不要指望
这篇文章能有啥用，它只是用来保存我的某些记忆。**

Arch Linux 跟大部分的 Linux 发行版本都不太相同。

下面是安装过程中的一些不同：

1. 没有 GUI/TUI installer/wizard
2. 全命令行操作
3. 核心程序或包只包含最基本和必要的东西
4. 安装什么程序自己决定，包括系统内核

因此，虽然使用 Arch Linux 不要求你是高手，但熟悉一些基本的 bash 操作还是有用的。

## 登录

经过前面的步骤，我们已经使用 U 盘将 `archiso` 临时系统载入了内存，并且默认以
 `root` 的身份登录到 `archiso`。

## 联网

Arch Linux 的安装是需要联网的，如果是台式机直连的网线在默认情况下是直接联网了，
如果是 WIFI 环境，可以使用 `iw` 和 `wpa_supplicant`，当然，一些基本的命令比如
 `ip a`，`dhcpcd`，`ping` 之类的也可能用到。

## 分区、格式化

之前说过，建议主板 UEFI 模式 + 硬盘 GPT 分区，这对于安装多系统以及系统间的
独立性比较有好处。archlinux wiki 也有支持其它组合的方案。

我使用 `fdisk` 进行分区。我习惯分配 1 个 1G 的 EFI 分区，一个 `/` 分区，有时会
额外增加一个 `home` 分区。因为是个人使用，并且随时可以用 U 盘修复问题，况且
熟练使用且良好维护的 Arch Linux 几乎不出问题，通常就是 `EFI + /`。

分区后，要格式化，这里的 sda 只是示例，实际按 `lsblk` 等命令查看设备名:

1. `mkfs.fat -F32 /dev/sda1`
2. `mkfs.ext4 /dev/sda2`
3. 我通常不设置 `swap` 分区，如果后续确有需要，可以用 'swap-file' 的方式

顺便再 `mount`

1. `mount /dev/sda2 /mnt`
2. `mkdir /mnt/efi`
3. `mount /dev/sda1 mnt/efi`
4. 如果有其它分区，比如 `/dev/sda3` 作为 `/home`，重复 2，3 步就好

## 国内源

我使用 Arch Linux 很大一个原因是它的国内源快的飞起。

1. `vim /etc/pacman.d/mirrorlist`
2. 用 vim macro 将这台机器所处地区的所有 server 移到顶部，国内可以选 163、清华
等放第一排，美国建议 kernel.org 放第一排。

## pacstrap

照着 wiki 走就行了，这时可以将后续用到的几乎所有程序列出来，一次安装。

也可以实现一个简单的脚本，将所有程序列在里面。通常，这一步我只安装少量的程序。

```shell
pacstrap /mnt base base-devel linux linux-firmware vim bash-completion
```

它的下载速度取决于你是否配置好国内源以及你的 ISP。这里 base-devel 和 vim 以及
bash-completion 都不是必须的，但后续的 `sudo`、配置文件编辑、包文件名补全等
需要它们。

## genfstab

`genfstab -U /mnt >> /mnt/etc/fstab` 自动生成文件系统分区信息表

## arch-chroot

这是一个重要的命令，它大致上可以让我们从 U 盘启动直接进入到硬盘系统。在修复
问题时很有用。

这里，它的目的在于我们当前安装的系统还没法启动，使用它我们可以临时进入并进行配置。

后面的步骤，除非说明，都是在新安装的系统中进行操作的。

## grub

系统启动引导程序。

不一定用到 grub，但 grub 最常用。

1. `pacman -S grub intel-ucode efibootmgr`，也可以 amd-ucode，看 CPU 具体情况
2. `grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=arch`
3. `grub-mkconfig -o /boot/grub/grub.cfg`

## dhcpcd, iw, wpa

之前 `dhcpcd` 是不需要单独安装的，在 base 里已经有了，现在必须手动安装。

笔记本最好安装 `iw` 和 `wpa_supplicant` 或者其它 wifi 工具。台式机也可以装，
防止网线直连出问题，可以用无线卡替补。

这里就牵扯到 `systemd` 了，可以将这些通常开机都要用到的底层工具自启动，这些
工具通常都设置了现成的 service，直接用就好了。

`systemctl enable`、`systemctl start`。

## useradd and sudo and passwd

```shell
useradd -m username
vim /etc/sudoers
passwd root
passwd username
```

注意，本文中我只是列出我认为重要的一些步骤和操作示例，Linux 是相当灵活的，
请根据实际情况作出调整。

还有一些其它的设置，比如 locale，一些程序需要 UTF-8，hostname 和 hosts 等等，
但由于这些不是必须的，这里我就不列出来了，可以参考 wiki。


## 退出 chroot

`exit` 或 `Ctrl-d`

## umount

`umount -R /mnt`

这里注意，如果有用到 swap file，先 `swapoff`

## reboot

至此，Arch Linux 的基本安装就完成了。注意，这时它仍然只有 CLI 没有 GUI, 并且
几乎是最简状态，很多其它 CLI 工具也没有。这就是 Arch Linux 很大的一个特点，
我们可以自由的在一个极简的基础上搭建起我们想要的系统。

详情参考 [Installation guide](https://wiki.archlinux.org/index.php/Installation_guide)
