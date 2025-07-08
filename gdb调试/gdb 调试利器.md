# gdb 调试利器

GDB 是一个由GNU开源组织发布的、UNIX/LINUX操作系统下的、基于命令行的、功能强大的程序调试工具。对于Linux下工作的c++程序员，gdb是必不可少的工具。

## 启动gdb

对 C/C++ 程序的调试，需要在编译前就加上 `-g` 选项：

```bash
$ g++ -g hello.cpp -o hello
```

调试可执行文件：

```bash
$ gdb hello
```

调试 core 文件（core是程序非法执行后 core dump 后产生的文件）：

```bash
$ gdb <program> <core dump file>
```

调试服务程序：

```bash
$ gdb <program> <PID>
```

## 交互命令

### 运行

- run（简写r）：运行程序
- continue（简写c）：继续执行，到下一个断点处、或者运行结束
- next（简写n）：下一步，单步调试
- step（简写s）：进入函数内部
- until：退出循环体
- quit（简写q）：退出gdb

### 设置断点

-  break n （简写 b n）：在第n行处设置断点
- clear n：清除第n行的断点

### 查看源代码

- list（简写 l)：列出程序的源代码，默认每次显示10行

- list n：显示当前行号为中心的前后10行代码

### 查询运行信息

- where/bt：当前运行的堆栈列表

- bt ：显示当前调用堆栈

  