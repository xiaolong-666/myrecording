# Pbuilder 使用

pbuilder(personal Debian package builder)是Debian环境下维护deb包的专业工具，能够为每个deb包创建纯净的编译构建环境，自动解析和安装依赖包，并且不污染宿主系统。

## 安装

```bash
sudo apt install pbuilder debootstrap devscripts
```

## 修改Pbuilder配置文件

假如使用的仓库地址为内网专用设备版mars/1010仓库，地址：`deb http://****/stable/device mars/1010 main contrib non-free`

修改`/etc/pbuilderrc`文件：

```bash
MIRRORSITE=http://****/stable/device
DEBOOTSTRAPOPTS=(
        "--no-check-gpg"
        "--variant=buildd"
        "--include=dbus,deepin-keyring,python3,python3-apt,wget"
)
COMPONENTS="main contrib non-free"
DISTRIBUTION=mars/1020
OTHERMIRROR="deb http://****/unstable/device mars/1020 main contrib non-free"
```

`MIRRORSITE`: 仓库地址，此处填写的就是专用设备的内网仓库地址，需注意，没有包含codename

`DEBOOTSTRAPOPTS`：debootstrap的参数。debootstrap 是拉取最小的基础文件系统的工具

	1. "--no-check-gpg"  是由于内网仓库的包没有签名，我们需要跳过gpg 签名检查
	2. "--variant=buildd" 在 chroot 环境中安装 build-essential 包，这可能是您想要的，因为您将编译包
	3. "--include=" 指定额外预装的包

`COMPONENTS`: 发行版分类

`DISTRIBUTION`：codename即发行版代号

`OTHERMIRROR`: 指定其它的仓库地址



更多配置参数详见：[man pbuilderrc](https://manpages.debian.org/jessie/pbuilder/pbuilderrc.5.en.html)

## 创建一个Pbuilder tgz

配置好上述参数后，即可使用`pbuilder create`命令，拉取基础文件系统，并将其自动压缩成tgz文件

```bash
sudo pbuilder create --basetgz xiaolong.tgz                                                        
W: /root/.pbuilderrc does not exist
W: cgroups are not available on the host, not using them.
I: Distribution is mars/1010.
I: Current time: Mon Jul  5 13:41:25 CST 2021
I: pbuilder-time-stamp: 1625463685
I: Building the build environment
I: running debootstrap
/usr/sbin/debootstrap
I: Target architecture can be executed
...
I: unmounting dev/ptmx filesystem
I: unmounting dev/pts filesystem
I: unmounting dev/shm filesystem
I: unmounting proc filesystem
I: unmounting sys filesystem
I: creating base tarball [/home/xiaolong/xiaolong.tgz]
I: cleaning the build env 
I: removing directory /var/cache/pbuilder/build/1066 and its subdirectories

```

**需要在/usr/share/debootstrap/scripts/ 下存在于codename相同文件名的文件，如/usr/share/debootstrap/scripts/mars/1010， 如果没有需要手动复制过去。**

```bash
mkdir -p /usr/share/debootstrap/scripts/mars/
sudo install -Dvm644 /usr/share/debootstrap/scripts/buster /usr/share/debootstrap/scripts/mars/1010
```
`--basetgz xiaolong.tgz`: 指定保存的文件名，不指定的话，默认保存在`/var/cache/pbuilder/`目录下

如果一切顺利，那么你会在当前目录下看到生成的`xiaolong.tgz`base基础包，这个基础环境就可以用于构建使用。


## 更新基础base包
如果想更新基础的base包，使用`pbuilder update`命令即可更新指定发行版的base.tgz。此外，通过指定--override-config选项，可以使用 base.tgz 的配置文件中的给定选项和设置来安装新的 apt-line。
```bash
[xiaolong@xiaolong-PC ~ ]$ sudo pbuilder update --basetgz xiaolong.tgz --override-config 
W: /root/.pbuilderrc does not exist
W: cgroups are not available on the host, not using them.
I: Upgrading for distribution mars/1010
I: Current time: Mon Jul  5 14:11:41 CST 2021
I: pbuilder-time-stamp: 1625465501
I: Building the build Environment
I: extracting base tarball [/home/xiaolong/xiaolong.tgz]
I: copying local configuration
I: Installing apt-lines
I: mounting /proc filesystem
I: mounting /sys filesystem
I: creating /{dev,run}/shm
I: mounting /dev/pts filesystem
I: redirecting /dev/ptmx to /dev/pts/ptmx
I: policy-rc.d already exists
I: Refreshing the base.tgz 
...
I: creating base tarball [/home/xiaolong/xiaolong.tgz]
I: cleaning the build env 
I: removing directory /var/cache/pbuilder/build/10962 and its subdirectories
```

## build构建
### 使用apt source源码构建
在使用base.tgz创建的 chroot 环境中构建.dsc-file指定的包。此处以定制工具`iso-tailor`的源包为例：
```bash
sudo pbuilder build --basetgz ~/xiaolong.tgz iso-tailor_3.0.1.dsc 
```
现在分析一下过程：
1. 在`/var/cache/pbuilder/build`目录中创建一个临时目录，存放解压后的基础文件系统
2. 挂载相应环境，如`/proc /sys /dev/pts`等
3. 拷贝源包数据`iso-tailor_3.0.1.dsc iso-tailor_3.0.1.tar.xz`到基础文件系统中
4. 使用chroot技术，切换根目录
5. 分析源包的依赖关系，并安装编译缺少的包
6. 编译，并将编译结果存放于`/var/cache/pbuilder/result`目录下
7. 卸载环境，并清理编译环境，退出
### 本地源码构建
注意将base基础包放到`/var/cache/pbuilder/base.tgz`，然后进入源码目录，执行`sudo pdebuild`命令即可：
```bash
xiaolong-PC :: gerrit/isotailor » sudo pdebuild                           
W: /root/.pbuilderrc does not exist
dh clean
   dh_clean
        rm -f debian/debhelper-build-stamp
        rm -rf debian/.debhelper/
        rm -f -- debian/iso-tailor.substvars debian/files
        rm -fr -- debian/iso-tailor/ debian/tmp/
dpkg-source: info: using source format '3.0 (native)'
dpkg-source: info: building iso-tailor in iso-tailor_3.0.02.tar.xz
dpkg-source: info: building iso-tailor in iso-tailor_3.0.02.dsc
...
I: cleaning the build env 
I: removing directory /var/cache/pbuilder/build/28844 and its subdirectories
I: Current time: Thu Jul 29 10:17:48 CST 2021
I: pbuilder-time-stamp: 1627525068
```
过程如下：
1. pdebuild命令会先调用dh clean清理模块，清理环境
2. 然后使用dpkg-source，生成*.dsc和*.tar.xz文件，这两个文件是pbuilder使用的核心文件
3. 准备chroot环境，将会使用`/var/cache/pbuilder/base.tgz`作为基础环境
4. 分析依赖关系，生成临时`pbuilder-satisfydepends-dummy`软件包，安装需要的依赖
5. 构建
6. 清理环境并退出，生成的文件存放于`/var/cache/pbuilder/result`目录下，如果没有修改存放路径


## clean
清理构建的目录，执行`pbuilder clean`
```bash
$ sudo pbuilder clean 
W: /root/.pbuilderrc does not exist
W: cgroups are not available on the host, not using them.
I: Cleaning [/var/cache/pbuilder/build]
I: removing directory /var/cache/pbuilder/build and its subdirectories
I: Cleaning [/var/cache/pbuilder/aptcache/]
```
## login
以`chroot`的方式登陆基础文件系统即base系统
```bash
$ sudo pbuilder login --basetgz ~/xiaolong.tgz
W: /root/.pbuilderrc does not exist
W: cgroups are not available on the host, not using them.
I: Building the build Environment
I: extracting base tarball [/home/xiaolong/xiaolong.tgz]
I: copying local configuration
I: mounting /proc filesystem
I: mounting /sys filesystem
I: creating /{dev,run}/shm
I: mounting /dev/pts filesystem
I: redirecting /dev/ptmx to /dev/pts/ptmx
I: mounting /dev/pts/2 over /dev/console
I: policy-rc.d already exists
I: Obtaining the cached apt archive contents
I: entering the shell
I: File extracted to: /var/cache/pbuilder/build/30046
root@xiaolong-PC:/# 
```

## 参考

https://wiki.ubuntu.com/PbuilderHowto