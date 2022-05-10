# 用户家目录skel模板介绍

`/etc/skel` 这个目录，大家平时可能都没太注意，但是却与用户存在密切的联系。

`skel` 是 `skeleton` 的缩写，每当你新建立一个用户时（通过 `useradd` 命令），`/etc/skel` 目录下的文件都会原封不动的复制到新建用户的家目录下。

该目录下的内容如下：

```bash
root@live:/etc/skel# ls -al
total 72
drwxr-xr-x  13 root root  4096 Oct 29  2021 .
drwxr-xr-x 129 root root 12288 May  5 09:33 ..
drwxr-xr-x   2 root root  4096 Oct 29  2021 .Public
drwxr-xr-x   2 root root  4096 Oct 29  2021 .Templates
-rw-r--r--   1 root root   220 Dec  3  2019 .bash_logout
-rw-r--r--   1 root root  3748 Oct 29  2021 .bashrc
drwxr-xr-x   5 root root  4096 Oct 29  2021 .config
drwxr-xr-x   3 root root  4096 Oct 29  2021 .icons
drwxr-xr-x   3 root root  4096 Oct 29  2021 .local
-rw-r--r--   1 root root   807 Dec  3  2019 .profile
drwxr-xr-x   2 root root  4096 Oct 29  2021 Desktop
drwxr-xr-x   2 root root  4096 Oct 29  2021 Documents
drwxr-xr-x   2 root root  4096 Oct 29  2021 Downloads
drwxr-xr-x   2 root root  4096 Oct 29  2021 Music
drwxr-xr-x   3 root root  4096 Oct 29  2021 Pictures
drwxr-xr-x   2 root root  4096 Oct 29  2021 Videos
```

可以看到，熟悉的一系列配置文件 `.bashrc` 等。所以大家应该清楚了，为什么新建立一个用户，用户的目录下就自动存在这些文件了。



那么既然知道，这是新建用户的模板目录，当然也可以对它进行修改，比如添加一个 `desktop` 文件，使后续新建立的所有用户桌面都存在该图标。那么只需在 `/etc/skel/Desktop/` 目录下新建立一个 `desktop` 文件即可。