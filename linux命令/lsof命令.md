# lsof 命令

lsof (list open files) 是一个列出当前系统打开文件的工具。在 `Linux` 系统下，一切皆文件，都可以使用该命令来查看。

lsof打开的文件可以是：

- 普通文件
- 目录
- 设备文件
- 共享库等
- ...

命令参数：

- -a：列出打开文件存在的进程
- -c<进程名>：列出指定进程所打开的文件
- -p<进程号>：列出指定进程号所打开的文件
- ...

## 使用

1. 无任何参数执行

```bash
COMMAND     PID   TID TASKCMD               USER   FD      TYPE             DEVICE  SIZE/OFF       NODE NAME
systemd       1                             root  cwd   unknown                                         /proc/1/cwd (readlink: Permission denied)
systemd       1                             root  rtd   unknown                                         /proc/1/root (readlink: Permission denied)
systemd       1                             root  txt   unknown                                         /proc/1/exe (readlink: Permission denied)
systemd       1                             root NOFD                                                   /proc/1/fd (opendir: Permission denied)
...
```

**lsof输出各列信息的意义如下：**

COMMAND：进程的名称

PID：进程标识符

PPID：父进程标识符（需要指定-R参数）

USER：进程所有者

PGID：进程所属组

FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等

TYPE：文件类型，如DIR、REG等，常见的文件类型

（1）DIR：表示目录

（2）CHR：表示字符类型

（3）BLK：块设备类型

（4）UNIX： UNIX 域套接字

（5）FIFO：先进先出 (FIFO) 队列

（6）IPv4：网际协议 (IP) 套接字

DEVICE：指定磁盘的名称

SIZE：文件的大小

NODE：索引节点（文件在磁盘上的标识）

NAME：打开文件的确切名称



## 参考链接

https://www.cnblogs.com/peida/archive/2013/02/26/2932972.html