# 读取kvm虚拟机镜像数据

kvm虚拟机的磁盘存储文件一般有两种格式：压缩版的qcow2文件和裸数据raw格式。virt-ls等命令不支持qcow2格式，必须要把qcow2转换成raw格式。

需要先安装对应工具：
```bash
sudo apt install libguestfs-tools
```

1. 找到虚拟机磁盘对应的存储文件，此处为`/var/lib/libvirt/images/test.qcow2`
2. 使用qemu-img进行转换
```bash
 sudo qemu-img convert -f qcow2 -O raw test.qcow2 test.img
```
3. 查看虚拟机的磁盘里的`/var/log`目录下有哪些文件：
```bash
sudo virt-ls -a test.img /var/log
```
4. 如果有`deepin-installer.log`，就使用`virt-copy-out`命令，将其拷贝出来
```bash
sudo virt-copy-out -a test.img /var/log/deepin-installer.log ~/
```

如果使用`virt-ls`命令报错，可以使用`mount`命令挂载指定分区到目录下，需要先使用`fdisk`命令查看分区信息：
```bash
devops@kvm-amd-01:/var/lib/libvirt/images$ sudo fdisk test.img

Welcome to fdisk (util-linux 2.33.1).                                                                                                  
Changes will remain in memory only, until you decide to write them.                                                                    
Be careful before using the write command.

Command (m for help): p
Disk test.img: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 84E35FBA-FB6D-4246-A438-4C9F70E20397

Device         Start       End   Sectors  Size Type
test.img1       2048    616447    614400  300M EFI System
test.img2     616448   3762175   3145728  1.5G Linux filesystem
test.img3    3762176  35219455  31457280   15G Linux filesystem
test.img4   35219456  66676735  31457280   15G Linux filesystem
test.img5   66676736 174063615 107386880 51.2G Linux filesystem
test.img6  174063616 197132287  23068672   11G Linux filesystem
test.img7  197132288 209713151  12580864    6G Linux swap
```
找到分区的起始扇区，然后乘上字节数，作为下面mount挂载的offset偏移量。
比如此处data分区偏移量：66676736 * 512
```bash
sudo mount -o loop,offset=34138488832 test.img /mnt
```


