# Plymouth一探

Linux 系统采用开机动画去覆盖启动时的打印日志，目的是提供无闪烁的系统启动体验。要使用开机动画，必须在启动时传递 `Splash` 参数，通过内核中*内核模式设置（Kernel Mode-Settings)*。

## Plymouth 的相关原理

整体分为两个主要部分：服务端和客户端，典型的C/S模型。服务端和客户端直接通过 `socket` 通信。

服务端：是一个后台守护进程 `plymouthd` ，用于处理请求，请求种类很多，比如典型的 `update`、`quit` 等。服务端通过 `epoll` 监控相关 `socket` ，监听来自客户端的消息。

客户端：客户端可以多种多样，典型的客户端有：`plymouth` 程序、`systemd` 。客户端通过 `socket` 与服务端建立连接，并通过 `socket` 发送具体的请求。



开机动画中的进度条（动画）更新动作，是通过 `update` 请求调用，该请求是 `systemd` 发送过来的，`systemd` 在每启动完一个服务时，都会向 `plymouthd` 发送相应的 `update` 请求。

我们知道，`systemd` 中会启动很多的服务，当 `systemd` 完成一个服务的启动时，会在其子进程的 `SIGCHLD` 信号处理过程中，向 `plymouthd` 发送`update` 请求，也就表示当一个服务启动完成时，开机动画就会进行相应的更新。

### 进度控制

通过 `cache` 文件，也就是一个文本文件，来控制总的进度：

```bash
➜  ~  cat /var/lib/plymouth/boot-duration                  
0.198:-.mount
0.200:dev-hugepages.mount
0.201:kmod-static-nodes.service
0.201:systemd-remount-fs.service
...
0.796:deepin-accounts-daemon.service
0.797:libvirt-guests.service
0.814:upower.service
```

## 自定义开机动画

首先了解一下 `plymouth` 的相关命令

```bash
xiaolong@xiaolong-PC ~ $ plymouth-set-default-theme -h                                                                           

Plymouth theme chooser
usage: plymouth-set-default-theme { --list | --reset | <theme-name> [ --rebuild-initrd ] | --help }

  -h, --help             Show this help message
  -l, --list             Show available themes
  -r. --reset            Reset to default theme
  -R, --rebuild-initrd   Rebuild initrd (necessary after changing theme)
  <theme-name>           Name of new theme to use (see --list for available themes)
```

从帮助页面可以看到，该命令操作的是主题包类型，使用 `-l` 参数列出当前系统已有的主题

```bash
xiaolong@xiaolong-PC ~ $ plymouth-set-default-theme -l                                                                           
deepin-hidpi-logo
deepin-hidpi-ssd-logo
deepin-logo
deepin-ssd-logo
details
text
tribar
uos-hidpi-ssd-logo
uos-ssd-logo
```

这些主题文件都存放于 `/usr/share/plymouth/themes` 目录下

```bash
xiaolong@xiaolong-PC /usr/share/plymouth/themes $ ls                                                                            
deepin-hidpi-logo  deepin-hidpi-ssd-logo  deepin-logo  deepin-ssd-logo  details  text  tribar  uos-hidpi-ssd-logo  uos-ssd-logo
```

所以，自定义开机动画主要就是修改 `themes` 目录下，已有的主题，或者新建主题。

### 修改当前主题图片

使用 `plymouth-set-default-theme` 命令，查看当前系统使用的主题

```bash
xiaolong@xiaolong-PC /usr/share/plymouth/themes$ plymouth-set-default-theme                                                       
uos-ssd-logo
```

可以看到，当前系统上使用的是 `uos-ssd-logo` 主题包，该主题内容如下：

```bash
xiaolong@xiaolong-PC:/usr/share/plymouth/themes/uos-ssd-logo $ ls
boot.png  box.png  bullet.png  entry.png  lock.png  logo.png  uos-ssd-logo.grub  uos-ssd-logo.plymouth  uos-ssd-logo.script
```

`logo.png` 就是当前主题展示的图片，如果只是简单的替换当前主题的图片，可以直接替换该图片，然后执行 `update-initramfs -u` 命令重新生成 `initrd.img` 文件

```bash
xiaolong@xiaolong-PC:~|⇒  sudo update-initramfs -u
update-initramfs: Generating /boot/initrd.img-4.19.0-amd64-desktop
cryptsetup: WARNING: The initramfs image may not contain cryptsetup binaries 
    nor crypto modules. If that's on purpose, you may want to uninstall the 
    'cryptsetup-initramfs' package in order to disable the cryptsetup initramfs 
    integration and avoid this warning.
setupcon is missing. Please install the 'console-setup' package.
W: plymouth: The plugin label.so is missing, the selected theme might not work as expected.
W: plymouth: You might want to install the plymouth-themes package to fix this.
I: The initramfs will attempt to resume from /dev/nvme0n1p7
I: (UUID=ff08db2f-8533-468b-bb64-4ffe1ff962bf)
I: Set the RESUME variable to override this.
live-boot: core filesystems devices utils udev blockdev dns.
```

然后重启，即可看到开关机图片已经更改了。

### 自定义主题

要想实现自定义主题，从零搭建一个没有必要，可以参考已有的主题，比如，我们专业版其中一个主题 `uos-ssd-logo` 为例（也可以安装 `plymouth-themes` 主题包，参考里面的 `scripts` 主题）。

```bash
xiaolong@xiaolong-PC:/usr/share/plymouth/themes/uos-ssd-logo|⇒  tree                         
.
├── boot.png
├── box.png
├── bullet.png
├── entry.png
├── lock.png
├── logo.png
├── uos-ssd-logo.grub
├── uos-ssd-logo.plymouth
└── uos-ssd-logo.script

0 directories, 9 files
```

拷贝一份该主题，重命名为 `xiaolong-test`，将里面 `uos-ssd-logo`开始的文件，一并命名为 `xiaolong-test`，需要跟目录保持一致。

```bash
xiaolong@xiaolong-PC:/usr/share/plymouth/themes/xiaolong-test|⇒  ls
boot.png  box.png  bullet.png  entry.png  lock.png  logo.png  xiaolong-test.grub  xiaolong-test.plymouth  xiaolong-test.script
```

接着先修改 `xiaolong-test.plymouth` 文件，将其中的图片路径和脚本路径修改如下。

```bash
xiaolong@xiaolong-PC:/usr/share/plymouth/themes/xiaolong-test|⇒  cat xiaolong-test.plymouth
[Plymouth Theme]
Name= xiaolong test plymouth
Description= my first thems
ModuleName=script

[script]
ImageDir=/usr/share/plymouth/themes/xiaolong-test
ScriptFile=/usr/share/plymouth/themes/xiaolong-test/xiaolong-test.script
```

然后，打开 `xiaolong-test.script` 文件，查看其中代码，可以看到，加载的图片是当前目录下的 `logo.png` 图片。

```bash
...
# Set Background Color
Window.SetBackgroundTopColor(0, 0, 0);
Window.SetBackgroundBottomColor(0, 0, 0);

logo.image = Image("logo.png");		# 加载的图片名
logo.sprite = Sprite(logo.image);
logo.x = Window.GetX() + Window.GetWidth() / 2 - logo.image.GetWidth() / 2;
logo.y = Window.GetY() + Window.GetHeight() / 2 - logo.image.GetHeight() / 2;
logo.sprite.SetPosition(logo.x, logo.y, 10000);
...
```

如果想简单的创建属于自己的主题，那么这个脚本文件就不用修改，在当前目录下替换 `logo.png` 图片即可

想实现复杂的脚本，可以参考该脚本 [语法介绍](https://www.freedesktop.org/wiki/Software/Plymouth/Scripts/) 编写即可。

最后，主题创建好了，需要让它生效，一样使用 `plymouth-set-default-theme` 命令，设置为我们自己创建的主题，然后更新 `initrd`。

```bash
xiaolong@xiaolong-PC:/usr/share/plymouth/themes/xiaolong-test|⇒  plymouth-set-default-theme xiaolong-test 
xiaolong@xiaolong-PC:/usr/share/plymouth/themes/xiaolong-test|⇒  sudo plymouth-set-default-theme -R xiaolong-test              
update-initramfs: Generating /boot/initrd.img-4.19.0-amd64-desktop
cryptsetup: WARNING: The initramfs image may not contain cryptsetup binaries 
    nor crypto modules. If that's on purpose, you may want to uninstall the 
    'cryptsetup-initramfs' package in order to disable the cryptsetup initramfs 
    integration and avoid this warning.
setupcon is missing. Please install the 'console-setup' package.
W: plymouth: The plugin label.so is missing, the selected theme might not work as expected.
W: plymouth: You might want to install the plymouth-themes package to fix this.
I: The initramfs will attempt to resume from /dev/nvme0n1p7
I: (UUID=ff08db2f-8533-468b-bb64-4ffe1ff962bf)
I: Set the RESUME variable to override this.
live-boot: core filesystems devices utils udev blockdev dns.
```

重启即可看到，开关机 `logo` 变成了自己定义的图片。

## 主题下载

https://www.gnome-look.org/browse?cat=108&ord=latest

## 参考链接

https://www.bianchengquan.com/article/447033.html