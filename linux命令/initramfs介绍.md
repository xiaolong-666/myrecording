# initramfs 介绍

 ## 初始根磁盘

`initrd` 是一个内存中的磁盘结构（ramdisk），其中包含必要的工具和脚本，用于在将控制权交给根文件系统上的 `init` 程序之前挂载所需的文件系统。内核在此根磁盘上触发安装脚本，此脚本的工作是准备系统、切换到真正的根文件系统，然后调用 `init`。

## initial ram 文件系统

`initramfs` 初始 `ram` 文件系统基于 `tmpfs` （大小灵活、内存中的轻量级文件系统）。`initramfs` 的内容是通过创建一个 `cpio` （cpio 是一个文件归档器解决方案）存档来制作的。

所有文件、工具、库、配置设置等都放入 `cpio` 存档。然后使用 `gzip` 压缩此存档并与 `linux` 内核一起存储。引导加载程序在引导时将其提供给 `linux` 内核。

一旦检测到，`linux` 内核将创建一个 `tmpfs` 文件系统，提取其中的存档内容，然后启动位于 `tmpfs` 文件系统根目录中的 `init` 脚本。然后这个脚本挂载真正的根文件系统以及重要的其它文件系统（`/usr` `/var`）。

一旦安装了根文件系统和其它重要文件系统，来自 `initramfs` 的初始化脚本会将根切换到真正的根文件系统，最后调用该系统上的 `/sbin/init` 二进制文件以继续引导过程。

## linux 的 initramrd img

`initramfs` 文件存放于 `/boot` 目录下，一般以 `initrd.img*` 格式命名。

可以使用 `unmkinitramfs` 命令解压该文件系统查看内容

```bash
amd@amd-PC:~$ sudo unmkinitramfs /boot/initrd.img-4.19.0-amd64-desktop temp
amd@amd-PC:~$ ls temp/main/
bin  conf  cryptroot  etc  init  lib  lib32  lib64  libx32  run  sbin  scripts  usr  var
```

可以看到 `initramfs` 跟分区文件系统的雏形很像，只是它的大小不大，少了很多工具和库。有些内核的模块就存放在其中，比如 `/usr/lib/modules/4.19.0-amd64-desktop/kernel/` 。
