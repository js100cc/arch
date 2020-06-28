# 制作系统安装 U 盘

## 需求：

1. U 盘，1G 以上
2. https://www.archlinux.org/download/ 下载系统 iso, 这里假设下载到 `~/arch.iso`
3. 一台 Linux 或具有相关命令的电脑

## 准备工作

1. 备份 U 盘数据
2. 电脑插入 U 盘
3. CLI: `lsblk` 查看 U 盘设备名，这里假设是 `/dev/sdb`

## 方法一，`dd`

```shell
dd if=~/arch.iso of=/dev/sdb bs=1M status=progress
```

自动格式化分区以及复制文件，且基本无需担心能否启动。简单粗暴且兼容性好。

每次使用 `dd` 都会重新格式化和分区，且分区格式和方式是先确定的了。因此对于大 U 盘来说并不一定合适，比如需要备份和保存其它数据。

## 方法二，手动分区

手动分区，再将系统文件散列出来复制到分区中。

1. `fdisk /dev/sdb`，也可以使用其它分区工具
2. `mkfs.fat -F32 /dev/sdb1`, FAT32 格式支持性好
3. `mkdir -p /mnt/{iso,usb}`
4. `mount /dev/sdb1 /mnt/usb`
5. `mount -o loop ~/arch.iso /mnt/iso`
6. `cp -a /mnt/iso/. /mnt/usb`，这一步之后，多数 linux distros 已经 OK 了，Arch 还要下面的步骤
7.  `sync`
8. `blkid -o value -s UUID /dev/sdb1`, 得到分区的 UUID
9. `vim /mnt/usb/loader/entries/archiso-x86_64.conf`
10. 修改最后一行, `archisolabel=...` 改为 `archisodevice=/dev/disk/by-uuid/$UUID`, 将分区实际 UUID 替换掉 $UUID

优点：

1. 不想保存当前系统文件时，可以删除分区内容，写入其它系统文件或直接当作 U 盘使用。
2. 可以分多个区，保存多个系统文件，一个 U 盘实现多系统安装。

缺点：

1. 繁琐，对于只是偶尔安装系统或专门一个 U 盘当系统安装盘或修复盘的情况没有必要。
2. 兼容性不一定能保证，大多数 Linux distros 直接支持或稍加调整即可，其它系统不一定支持。

## reference

更详细或更多的方式参考 [USB flash installation medium](https://wiki.archlinux.org/index.php/USB_flash_installation_medium#Using_manual_formatting)
