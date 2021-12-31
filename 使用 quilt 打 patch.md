# 使用quilt打patch
`debian`上游源代码，一般都使用`3.0(quilt)`方式打包。一般修复`bug`和其它小的改动都可以通过打`path`的方式来实现。需注意，只能使用`quilt`命令来生成`patch`包。

## 设置quilt
quilt 程序是 Debian 打包过程中采用的补丁管理工具。我们只需要在默认配置的基础上，加以少许修改即可。首先我们来创建一个别名 `dquilt`，以方便打包之需： 添加以下几行内容到 `~/.bashrc` 文件中。其中第二行可以给 `dquilt` 命令提供与 `quilt` 命令相同的 `shell` 补全：
```bash
alias dquilt="quilt --quiltrc=${HOME}/.quiltrc-dpkg"
complete -F _quilt_completion -o filenames dquilt
```
现在我们来创建 ~/.quiltrc-dpkg 文件：
```bash
d=. ; while [ ! -d $d/debian -a $(readlink -e $d) != / ]; do d=$d/..; done
if [ -d $d/debian ] && [ -z $QUILT_PATCHES ]; then
    # if in Debian packaging tree with unset $QUILT_PATCHES
    QUILT_PATCHES="debian/patches"
    QUILT_PATCH_OPTS="--reject-format=unified"
    QUILT_DIFF_ARGS="-p ab --no-timestamps --no-index --color=auto"
    QUILT_REFRESH_ARGS="-p ab --no-timestamps --no-index"
    QUILT_COLORS="diff_hdr=1;32:diff_add=1;34:diff_rem=1;31:diff_hunk=1;33:diff_ctx=35:diff_cctx=33"
    if ! [ -d $d/debian/patches ]; then mkdir $d/debian/patches; fi
fi
```
## 下载源码
修复`CVE`漏洞，均使用`apt source`方式下载的源码，此处以`scapy`为例
```bash
devops@devops-PC:~/cve/temp$ apt source scapy
正在读取软件包列表... 完成
提示：scapy 的打包工作被维护于以下位置的 Git 版本控制系统中：
https://salsa.debian.org/ineteng-team/scapy.git
请使用：
git clone https://salsa.debian.org/ineteng-team/scapy.git
获得该软件包的最近更新(可能尚未正式发布)。
需要下载 3,190 kB 的源代码包。
...
已下载 3,190 kB，耗时 0秒 (87.9 MB/s)
dpkg-source: info: extracting scapy in scapy-2.4.0
dpkg-source: info: unpacking scapy_2.4.0.orig.tar.gz
dpkg-source: info: unpacking scapy_2.4.0-2.debian.tar.xz
dpkg-source: info: using patch list from debian/patches/series
dpkg-source: info: applying setup.py.patch
dpkg-source: info: applying paths.patch
devops@devops-PC:~/cve/temp$ ls
scapy-2.4.0  scapy_2.4.0-2.debian.tar.xz  scapy_2.4.0-2.dsc  scapy_2.4.0.orig.tar.gz
```
注意：自动解压的目录，里面已经成功应用了所有`patch`，如果用`apt source`下载的源码来初始化仓库，还需要进行以下两步，移除应用的`patch`
```bash
devops@devops-PC:~/cve/temp/scapy-2.4.0$ quilt pop -a
Removing patch debian/patches/paths.patch
Restoring scapy/config.py
Restoring scapy/utils6.py

Removing patch debian/patches/setup.py.patch
Restoring setup.py

No patches applied
devops@devops-PC:~/cve/temp/scapy-2.4.0$ rm -rf .pc/
```
然后，就可以将`scapy-2.4.0`目录里的所有文件，用来初始化`gerrit`仓库了

## 应用旧的patch
进入源码目录，并将旧的`patch`应用到源码上
```bash
devops@devops-PC:~/cve/temp$ cd scapy-2.4.0/
devops@devops-PC:~/cve/temp/scapy-2.4.0$ dpkg-source --before-build ./
```
## 使用qulit生成新的patch
1. 生成新的`patch`文件
补丁命名规范，参考`CVE`平台上的漏洞修复流程介绍。
>CVE安全漏洞补丁统一命名格式： uniontech-<CVE ID>.patch,
>如 CVE ID 为 CVE-2020-0452 则 patch 文件名应该是 uniontech-CVE-2020-0452.patch
>如果一份补丁解决两个cve漏洞，则补丁名字为uniontech-<CVE ID>-<CVE ID>.patch。
>对于 uos 的安全漏洞命名为 <UT-年份-编号>，补丁名应为 uniontech-UT-年份-编号.patch

我本次创建如下：
```bash
devops@devops-PC:~/cve/temp/scapy-2.4.0$ quilt new uniontech-CVE-2019-1010142.patch
Patch debian/patches/uniontech-CVE-2019-1010142.patch is now on top
```
2. 添加需要修改的文件
```bash
devops@devops-PC:~/cve/temp/scapy-2.4.0$ quilt add scapy/layers/eap.py scapy/layers/radius.py test/regression.uts 
File scapy/layers/eap.py added to patch debian/patches/uniontech-CVE-2019-1010142.patch
File scapy/layers/radius.py added to patch debian/patches/uniontech-CVE-2019-1010142.patch
File test/regression.uts added to patch debian/patches/uniontech-CVE-2019-1010142.patch
```
3. 手动修改相应的文件
修改完成后，可以使用`quilt file`列出添加的文件；`quilt diff`查看改变的地方
```bash
devops@devops-PC:~/cve/temp/scapy-2.4.0$ quilt file
scapy/layers/eap.py
scapy/layers/radius.py
test/regression.uts
devops@devops-PC:~/cve/temp/scapy-2.4.0$ quilt diff
Index: scapy-2.4.0/scapy/layers/radius.py
===================================================================
--- scapy-2.4.0.orig/scapy/layers/radius.py
+++ scapy-2.4.0/scapy/layers/radius.py
@@ -9,11 +9,10 @@ RADIUS (Remote Authentication Dial In Us
 """
 
 import struct
-import logging
 import hashlib
 ...
```

4. 自动生成补丁文件
```bash
devops@devops-PC:~/cve/temp/scapy-2.4.0$ quilt refresh 
Refreshed patch debian/patches/uniontech-CVE-2019-1010142.patch
```
执行以上命令后，已经将本次的所有变更，自动生成了`patch`文件，并存放于`debian/patches`目录中
```bash
devops@devops-PC:~/cve/temp/scapy-2.4.0$ ls debian/patches/
paths.patch  uniontech-CVE-2019-1010142.patch  series  setup.py.patch
```
同时，自动在`series`文件中追加了当前的`patch`条目。
```bash
devops@devops-PC:~/cve/temp/scapy-2.4.0$ cat debian/patches/series 
setup.py.patch
paths.patch
uniontech-CVE-2019-1010142.patch
```
5. 添加补丁信息头
补丁要按照模板填写,需要在补丁上面添加 `patch `来源、作者、上游描述、补丁链接等信息。
模板如下：
>From：上游提交commit
Author: 上游提交作者
Date: 作者提交时间
Bug-Debian: debian的cve-tracker链接
Subject: 描述

使用`quilt header -e`按照模板编辑即可
```bash
devops@devops-PC:~/cve/temp/scapy-2.4.0$ quilt header -e
Replaced header of patch debian/patches/deepin-CVE-2019-1010142.patch
```

## 上传
上面步骤已经生成了`patch`文件，且`dpkg-buildpackage`会自动应用。那么只需要像上游提交你的`patch`文件和`series`文件，如果是需要进行打包发布，还应该更改`changlog`文件

`commit`信息规范
漏洞补丁提交到远端仓库时要注意`commit`的信息及格式,具体格式参考如下:

>第一行:（CVE号）本次修改的简短描述
第二行： 对本次修改的详细说明
第三行：log日志信息
第四行： CVE-Debian:上游debian CVE漏洞地址
第五行：changeID





