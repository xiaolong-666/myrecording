DKMS一探

## 简介

`DKMS` 全称是 `Dynamic Kernel Module Suopport` , 用来生成[Linux](https://zh.wikipedia.org/wiki/Linux)的[内核模块](https://zh.wikipedia.org/wiki/可加载内核模块)的一个框架，其源代码一般不在[Linux内核](https://zh.wikipedia.org/wiki/Linux内核)[源代码树](https://zh.wikipedia.org/w/index.php?title=源代码树&action=edit&redlink=1)。 当新的内核安装时，`DKMS`支持的[内核](https://zh.wikipedia.org/wiki/Linux内核)[设备驱动程序](https://zh.wikipedia.org/wiki/设备驱动程序) 到时会自动重建。 `DKMS` 可以用在两个方向：如果一个新的内核版本安装，自动[编译](https://zh.wikipedia.org/wiki/编译)所有的模块，或安装新的模块（驱动程序）在现有的系统版本上，而不需要任何的手动编译或预编译软件包需要。例如，这使得新的[显卡](https://zh.wikipedia.org/wiki/显卡)可以使用在旧的Linux系统上。

## 安装

```bash
sudo apt install dkms
```

`DKMS` 要求我们的代码目录必须以 `-` 的格式命名，主要命令如下：

> status: 查看管理状态
>
> add: 添加模块
>
> build: 编译模块
>
> install: 安装模块
>
> uninstall: 卸载模块
>
> reomve: 删除模块

更多详情可以通过 `man dkms` 查看。



## 使用

要使用 `DKMS` 管理模块，需要在源代码下面保护 `dkms.conf` 文件，关于如何编写不再此描述，本文只讲使用。

### 添加模块

将代码命名为 module-version 格式并复制到 `/usr/src/` 目录下, 执行以下命令添加即可

```bash
sudo dkms add hello/0.1
```

### 编译模块

```bash
sudo dkms build hello/0.1
```

### 安装模块

```bash
sudo dkms install hello/0.1
```

### 删除模块

```bash
sudo dkms remove hello/0.1 --all
```

## 制作deb包

使用`DKMS`更为常见的用法是制作`deb`包，用户可以直接从`deb` 安装，制作`deb`包需要安装如下工具：

```bash
sudo apt install dh-make libdigest-md5-file-perl
```

制作命令如下：

```bash
sudo dkms add hello/0.1		# 先添加
sudo dkms build hello/0.1	# 编译
sudo dkms mkdeb hello/0.1	# 打包
```

## 参考链接

[DKMS 简介](https://blog.csdn.net/fouweng/article/details/53435602)