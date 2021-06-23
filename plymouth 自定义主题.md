# plymouth自定义主题
本文主要介绍，如何在系统启动时，使用自定义主题，以目前使用的uos系统为例

plymouth使用的主题都存放于`/usr/share/plymouth/themes`目录下
## 创建对应目录
在`/usr/share/plymouth/themes`目录下，创建一个目录，可以参考该目录下的其他主题，最简单的方式就是拷贝一个现成的主题，重命名为自己的主题，然后修改相应内容。
我本次拷贝了`uos-ssd-logo`主题，修改为`xiaolong-test`，然后将目录里面的三个脚本依次修改名称为`xiaolong-test.grub`、`xiaolong-test.plymouth`、`xiaolong-test.script`

```bash
devops@devops-PC:/usr/share/plymouth/themes/xiaolong-test$ ls
boot.png  box.png  bullet.png  entry.png  lock.png  logo.png  xiaolong-test.grub  xiaolong-test.plymouth  xiaolong-test.script
```
## 修改*.plymouth
修改该文件中的`ImageDir`和`ScriptFile`字段为自己定义的名称。
```bash
devops@devops-PC:/usr/share/plymouth/themes/xiaolong-test$ cat xiaolong-test.plymouth 
[Plymouth Theme]
Name=Deepin Logo for SSD
Description=Deepin logo plymouth for ssd
ModuleName=script

[script]
ImageDir=/usr/share/plymouth/themes/xiaolong-test
ScriptFile=/usr/share/plymouth/themes/xiaolong-test/xiaolong-test.script
```
## *.script
查看该文件可以知道，引导的图片通过这个脚本中的`logo.image = Image("logo.png")`加载，手动替换掉该logo图片，即可实现自定义开机logo图片，这都是静态的图片。

**若只是简单的替换动画，不需要改动本脚本。如果想要更炫的，请看本节后续内容。**

开机加载动态的图片，核心也是修改该脚本，通过循环加载一系列静态的图片来构成动态图，此处修改为动画部分如下：
```bash
Window.SetBackgroundTopColor(0, 0, 0);
Window.SetBackgroundBottomColor(0, 0, 0);

for (i=0; i<22; i++){
  miku[i].image = Image("miku" + i + ".png");
}

logo.image = Image("miku0.png");
logo.sprite = Sprite(logo.image);
logo.sprite.SetX (Window.GetWidth()  / 2 - logo.image.GetWidth()  / 2);
logo.sprite.SetY (Window.GetHeight() / 2 - logo.image.GetHeight() / 2);
logo.sprite.SetZ (100);
progress = 0.0;

timer = 0;

fun refresh (){
    timer++;
    speed = 2;
        if(timer % speed == 0){
                pic_num = (timer/speed) % 22;
                logo.sprite.SetImage(miku[pic_num].image);
        }
    
}
Plymouth.SetRefreshFunction (refresh);
```
在当前目录下，存放一系统静态图
```bash
devops@devops-PC:/usr/share/plymouth/themes/xiaolong-test$ ls                             
boot.png    entry.png  miku0.png   miku12.png  miku15.png  miku18.png  miku20.png  miku3.png  miku6.png  miku9.png    xiaolong-test.script                                          
box.png     lock.png   miku10.png  miku13.png  miku16.png  miku19.png  miku21.png  miku4.png  miku7.png  xiaolong-test.grub                                                                    
bullet.png  logo.png   miku11.png  miku14.png  miku17.png  miku1.png   miku2.png   miku5.png  miku8.png  xiaolong-test.plymouth  
```

### 自定义开机和关机logo不一致
在最近某个项目中，客户有这方面需求，要求开机展示一张带有初始化的图片，关机时展示另一张结束服务的图片。目前系统中都是默认展示的一张图片，通过设置透明度进行显隐操作。
为了达到目的，需要自己进行scripts脚本的编写。[参考链接](https://www.freedesktop.org/wiki/Software/Plymouth/Scripts/)
实现的核心部分在于，判断是开机、关机和重启等命令，展示不同的图片，查阅上述资料可知：`Plymouth.GetMode()`函数会返回一个字符串，取值为：`boot`、`shutdown`、`suspend`、`resume` 。这就对应着开关机状态。

```bash
if (Plymouth.GetMode() == "boot")	# 如果是开机引导
{
	logo.image = Image("logo.png");
	logo.sprite = Sprite(logo.image);
	logo.x = Window.GetX() + Window.GetWidth()/2 - logo.image.GetWidth()/2;
	logo.y = Window.GetY() + Window.GetHeight()/2 - logo.image.GetHeight()/2;
	logo.sprite.SetPosition(logo.x,logo.y,10000);
	Plymouth.SetRefreshFunction(boot_callback);	# 回调函数，在里面设置logo.sprite的透明度
}else
{
	...			# 其它模式展示的图片,跟上面类似
}
```

**其中核心部分是，script脚本中的图片对象，Image()和Sprite()**
```bash
logo.image = Image("logo.png");
logo.sprite = Sprite(logo.image);
```
请注意，上述对象的生命周期一旦存在就会被渲染，如果有多个对象，请放在对应模式的函数中，不然就会进行叠加渲染。

同时还有的项目，需要logo铺满屏幕，即全屏展示。
对于上面的`Image()`对象，调用`Scale()`函数，缩放到当前屏幕的大小即可，在生成`sprite`即可。[参考链接](https://joekuan.wordpress.com/2010/08/05/plymouth-create-your-own-splash-screen-with-scrolling-boot-messages/)
```bash
logo.image = Image("logo.png");
resize_image = logo.image.Scale(Window.GetWidth(),Window.GetHeight());
logo.sprite = Sprite(resize_image);
logo.x = Window.GetX();
logo.y = Window.GetY();
logo.sprite.SetPosition(logo.x,logo.y,10000);
```


## 查看主题，并设置默认主题
使用`plymouth-set-default-theme`可以修改当前使用的主题，`-h`参数有详细介绍
```bash
devops@devops-PC:/usr/share/plymouth/themes/xiaolong-test$ plymouth-set-default-theme -h
Plymouth theme chooser
usage: plymouth-set-default-theme { --list | --reset | <theme-name> [ --rebuild-initrd ] | --help }

  -h, --help             Show this help message
  -l, --list             Show available themes
  -r. --reset            Reset to default theme
  -R, --rebuild-initrd   Rebuild initrd (necessary after changing theme)
  <theme-name>           Name of new theme to use (see --list for available themes)
```
可以查看到默认使用的`uos-ssd-logo`，设置后，再次查看，变为`xiaolong-test`
```bash
devops@devops-PC:/usr/share/plymouth/themes/xiaolong-test$ plymouth-set-default-theme 
uos-ssd-logo
devops@devops-PC:/usr/share/plymouth/themes/xiaolong-test$ sudo plymouth-set-default-theme xiaolong-test
devops@devops-PC:/usr/share/plymouth/themes/xiaolong-test$ plymouth-set-default-theme 
xiaolong-test
```
## 更新grub
执行`sudo update-initramfs -u`更新引导
```bash
devops@devops-PC:/usr/share/plymouth/themes$ sudo update-initramfs -u
update-initramfs: Generating /boot/initrd.img-4.19.0-desktop-amd64
cryptsetup: WARNING: The initramfs image may not contain cryptsetup binaries 
    nor crypto modules. If that's on purpose, you may want to uninstall the 
    'cryptsetup-initramfs' package in order to disable the cryptsetup initramfs 
    integration and avoid this warning.
W: plymouth: The plugin label.so is missing, the selected theme might not work as expected.
W: plymouth: You might want to install the plymouth-themes package to fix this.
I: The initramfs will attempt to resume from /dev/vda4
I: (UUID=e9a250b8-6e4c-4127-a239-fcbfc351b3b0)
I: Set the RESUME variable to override this.
live-boot: core filesystems devices utils udev blockdev dns.
```

自此，开机后就可以看到图片已经换成自己的了！



## 参考链接
[plymouth scripts](https://www.freedesktop.org/wiki/Software/Plymouth/Scripts/)