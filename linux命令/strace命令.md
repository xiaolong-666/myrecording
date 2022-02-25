# strace 命令

strace 是一个可用于诊断、调试和教学的 `Linux` 用户空间跟踪器。我们用它来监控用户空间进程和内核的交互，比如系统调用、信号传递、进程状态变更等。

> strace 底层使用内核的 ptrace 特性来实现

`strace` 是跟踪用户空间进程的系统调用和信号的。

> 系统调用：
>
> > 指运行在用户空间的程序向操作系统内核请求需要更高权限运行的服务

## 使用

`strace` 有两种运行模式。

1. 通过它启动要跟踪的进程

> 在原本的命令前加上 strace 即可。比如跟踪 `ls -lh /var/log/messages`
>
> ```bash
> strace ls -lh /var/log/messages
> ```

2. 跟踪已经在运行的进程，在不中断进程执行的情况下，理解它在干嘛。

> 给 strace 传递个 -p pid 选项即可。



## 参考链接

https://www.linuxidc.com/Linux/2018-01/150654.htm