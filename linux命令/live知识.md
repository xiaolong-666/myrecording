

# live 系统

## 简介

`live system` 通常是指从一个可移动的设备：U盘、CD-ROM 或者网络上启动操作系统，无需安装，在运行时自动完成配置。

一个 `live` 系统通常由以下几个部分组成：

- Linux kernel image: 通常以 `vmlinuz` 命名
- Initial RAM disk image: 通常以 `initrd*` 命名
- System image: 操作系统的文件系统映像。通常使用 `SquashFS` 来压缩这个文件系统，只读。在 `live system` 引导期间将使用一个 `RAM` 磁盘以及一个 `union` 机制来允许在运行的系统上写文件。但是所有的修改在关机后都会丢失（除非使用persistence)。
- Boot loader（isolinux）: 从选择的媒介引导的一小段代码。它加载 `linux` 内核及其 `initrd`，使其与关联的文件系统(.squashfs)一起运行。



## LIve CD 启动流程

（1）**BIOS**读取CD-ROM中的BootRecord Volume Descriptor，定位boot.cat文件的地址，读取该文件中的Boot Entry，进一步定位***\*isolinux.bin文件\****的地址，接着，将该文件加载到内存并执行。

​    （2）isolinux.bin在*光盘*中查找isolinux.cfg文件，读取该文件，获得***\*启动配置\****，该文件记录了采用何种启动界面，有图形界面和者字符界面两种，其中图形界面下，会进一步加载vesamenu.c32文件和splash.jpg文件，完成图形界面的显示。在用户选定一个启动项之后，isolinux.bin按照isolinux.cfg中的配置，查找相应的***\*kernel文件\****，将其加载到内存中，将相应的参数传递给kernel并执行。

​    （3）kernel进行**内核初始化**，并将initrd.lz解压到/dev/ram中，并将其挂载为rootfs，执行其中的/init脚本。

​    （4）**init脚本**解析isolinux.bin传递进来的内核参数，接着加载核心驱动，挂载运行时文件系统，在参数解析阶段如果识别出boot=casper，init脚本会执行/scripts/casper脚本，主要调用其中mountroot()函数，实现squashfs的解压和挂载rootfs，最后init脚本执行/bin/run-init程序，该程序完成rootfs的切换，并执行/sbin/init程序，init程序根据/etc/rc.d中的配置，进一步初始化用户环境，包括图形界面、网络等。



`live-build`: 构建live系统的脚本集合

`live-boot`:  是为 `initramfs-tools` 提供钩子的脚本集合，用于生成能够启动的 `initramfs` 系统

> 内核传递`boot=live` 参数，就是给它使用的

`live-config`:  在 live-boot 后，启动时运行的脚本，用于自动配置 live 系统。比如设置主机名、语言环境、创建用户和执行自动登陆等任务。



## 参考链接

[live iso](!https://live-team.pages.debian.net/live-manual/html/live-manual/index.en.html)

[vmlinuz vs initrd.img vs system.map](https://blog.csdn.net/Geek_Tank/article/details/69479196)

[live sytem](https://blog.rickylss.site/os/2020/05/09/live-system/)