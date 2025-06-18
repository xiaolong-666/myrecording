# Btrfs 文件系统知识点

## 基础概念

Btrsf（B-tree file system）是针对 Linux 开发的一个新的 CoW（copy-on-write，写时复制）文件系统。`当一个文件被改变和写回磁盘，它不会故意写回它原来的位置，而是被复制和存储在磁盘上的新位置（只有被改变的数据块被复制，而不是全部文件）。` 

`CoW` 优秀点：文件系统被修改和编辑的历史被保存了下来。`Btrfs` 保存文件旧版本的引用（inode）可以轻易地被访问。这个引用就是快照：文件系统在某个时间点的状态镜像。

## 子卷

子卷 `Subvolume`  允许将一个 `Btrfs` 文件系统划分成多个独立的子文件系统。意味着可以从 `Btrfs` 文件系统挂载子卷。可通过以下命令来验证指定目录是否是 `Btrfs` 。

```bash
root@test-PC:~# findmnt -no FSTYPE /
btrfs
```

### 创建和使用子卷

可以通过如下命令创建一个 `Btrfs` 子卷:

```bash
root@test-PC:/home/btrfs-subvolume-xiaolong# btrfs subvolume create first
Create subvolume './first'
root@test-PC:/home/btrfs-subvolume-xiaolong# ls -lh
总计 0
drwxr-xr-x 1 root root 0  2月19日 10:24 first

```

可以看到，创建的子卷类型为目录，可以像常规目录一样操作它：重名名、移动、在里面新建文件和目录等。

表现起来像个目录，可以使用 `btrfs` 工具命令列出子卷的信息

```bash
root@test-PC:/home/btrfs-subvolume-xiaolong# btrfs subvolume list -o ./
ID 283 gen 46361 top level 257 path @/home/btrfs-subvolume-xiaolong/first
```

重命名子卷

```bash
root@test-PC:/home/btrfs-subvolume-xiaolong# mv first second
root@test-PC:/home/btrfs-subvolume-xiaolong# btrfs subvolume list -o ./
ID 283 gen 46361 top level 257 path @/home/btrfs-subvolume-xiaolong/second

```

移除子卷（可直接 rm 删除，或者通过 Btrfs 子卷的删除命令）

```bash
root@test-PC:/home/btrfs-subvolume-xiaolong# rm second/third/ -rf
root@test-PC:/home/btrfs-subvolume-xiaolong# btrfs subvolume delete second
Delete subvolume (no-commit): '/home/btrfs-subvolume-xiaolong/second'

```

### 像单独的文件系统一样操作子卷





## 参考链接

https://linux.cn/lctt/A2ureStone