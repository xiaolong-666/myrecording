# C项目转rust笔记

## 环境准备

- rust 安装

  ```bash
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```

  Rust 官方默认的 Cargo 源服务器为 crates.io，其同时也是 Rust 官方的 crate 管理仓库，放置在 github。访问较慢，可使用国内源。

```bash
# 创建或编辑 config.toml 文件
cat <<EOT > $HOME/.cargo/config.toml
[source.crates-io]
replace-with = 'tuna'

[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
EOT
```

- c2rust 安装

默认的安装方式 `cargo install c2rust` ，暂时不可用（官方已知问题），将会在后续版本修复，具体讨论可见：https://github.com/immunant/c2rust/issues/1125

故采用 github 源码的方式安装，下载最新的tag源码（v0.19.0）,进入源码目录

先安装编译依赖：

```bash
sudo yum install git gcc gcc-c++ llvm llvm-devel clang clang-devel make cmake ninja-build openssl-devel pkgconfig python3 clang-tools-extra
```

然后编译：

```bash
cargo build
```

**当遇见拉取 github 代码失败时，可通过临时修改一下源码地址，加上 https://ghfast.top/ 前缀（任意网站搜索，关键字 ghproxy ，使用可用的地址）**

> 比如拉取的原始地址：https://github.com/intel/tinycbor.git
>
> 增加后的地址：https://ghfast.top/https://github.com/intel/tinycbor.git



拉取源码后，使用 rpmbuild 命令解压源码应用补丁。

- 将源码包（如 .tar.gz）和所有补丁文件（.patch）放置于 ~/rpmbuild/SOURCES 目录下 
- 将 .spec 文件放置于 ~/rpmbuild/SPECS 目录下

rpmbuild -bp ~/rpmbuild/SPECS/your_package.spec



## iputils 项目

输出：7个二进制文件

- /usr/bin/arping

- /usr/bin/clockdiff 

- /usr/sbin/ping

- /usr/sbin/ping6 (链接到ping)

- /usr/sbin/tracepath

- /usr/sbin/tracepath6 (链接到tracepath)

- /usr/sbin/ifenslave

本项目，刚好是使用meson来组织的，编译后，自动在 builddir 目录下生成了 compile_commands.json 文件，故直接使用 c2rust来转换。

```bash
/home/amd/xiaolong/c2rust-0.19.0/target/debug/c2rust transpile compile_commands.json -e -o ./
```



Q1：\#[derive(Copy, Clone, BitfieldStruct)] 报错

> [dependencies]
>
> c2rust-bitfields= "0.3"
>
> 
>
> 源码中添加导入：use c2rust_bitfields::BitfieldStruct;



Q2：函数参数类型不匹配

> error[E0308]: arguments to this function are incorrect
>     --> src/ping/ping.rs:1970:31
>      |
> 1970 |                     ret_val = ping6_common::ping6_run(&mut rts, argc, argv, ai, &mut sock6);
>      |                               ^^^^^^^^^^^^^^^^^^^^^^^ --------              --  ---------- expected struct `ping6_common::socket_st`, found struct `ping::ping::socket_st`
>      |                                                       |                     |
>      |                                                       |                     expected struct `ping6_common::addrinfo`, found struct `ping::ping::addrinfo`
>      |                                                       expected struct `ping6_common::ping_rts`, found struct `ping::ping::ping_rts`
>      |
>      = note:    expected raw pointer `*mut ping6_common::ping_rts`
>              found mutable reference `&mut ping::ping::ping_rts`
>      = note: expected raw pointer `*mut ping6_common::addrinfo`
>                 found raw pointer `*mut ping::ping::addrinfo`
>      = note:    expected raw pointer `*mut ping6_common::socket_st`
>              found mutable reference `&mut ping::ping::socket_st`

转换出来的源文件，存在各自的的一样的数据结构定义。是将所有用到的.h 头文件的数据，在各自使用的rs文件都定义了一份。





转换出来的目录结构，不可直接使用。需要按照rust的项目规范，进行包、模块的组织。





AI转换词：

> 1. 获取文件列表：获取项目中所有 C 文件和h头文件的列表。
> 2. 逐个文件转换：针对每个 C 文件，逐行转换其内容到 Rust。
> 3. 模块映射：将 C 语言中的标准库和第三方库映射到 Rust 中的对应库。
> 4. 结构体和类型转换：确保所有结构体和类型在 Rust 中有对应的定义和实现，并用注释记录原始数据的位置。
> 5. 函数转换：逐行分析和转换每个函数，并确保所有调用的函数在 Rust 中也有实现。
> 6. 宏转换： 如果C的宏可以转换成函数，则可能转换成rust函数。
> 7. 错误处理：C 语言中的错误处理在 Rust 中使用 `Result` 类型，并处理可能的错误。
> 8. 内存管理：将 C 语言中的手动内存管理转换为 Rust 的所有权机制。



讨论：

1. 



[[bin]]
name = "arping"
path = "src/arping.rs"

[[bin]]
name = "clockdiff"
path = "src/clockdiff.rs"

[[bin]]
name = "tracepath"
path = "src/tracepath.rs"

[lib]
name = "c2rust_out"
path = "lib.rs"
crate-type = ["staticlib", "rlib"]





`ping` 命令需要创建 **ICMP 原始套接字**（`SOCK_RAW`），这需要 `CAP_NET_RAW` capability 权限。**普通用户默认没有此权限**，但主流发行版通过以下两种方式解决：

- **方案一：赋予 `CAP_NET_RAW` capability**
     通过 `setcap` 命令为 `/bin/ping` 文件添加 `cap_net_raw=ep`，使得普通用户执行 `ping` 时临时获得该权限。

    ```
    sudo setcap cap_net_raw+ep /bin/ping  # 赋予权限
    sudo getcap /bin/ping                 # 验证结果：cap_net_raw=ep
    ```

    这是现代发行版（如 Debian、Ubuntu）的默认做法 

- **方案二：设置 SUID 位**
       通过 `chmod u+s /bin/ping` 让 `ping` 以 root 身份运行，但这种方式会赋予程序完整的 root 权限，存在安全隐患，已逐渐被弃用.







### 第一周（2025/02/17-2025/02/21）

####  ping 命令转换情况

参数转换支持列表：

| 原始可选参数      | 参数描述                                                     | rust版本参数支持情况             |
| ----------------- | ------------------------------------------------------------ | -------------------------------- |
| <destination>     | dns name or ip address                                       | 已转换                           |
| -4                | use IPv4                                                     | 已转换                           |
| -6                | use IPv6                                                     | 已转换                           |
| -c <count>        | stop after <count> replies                                   | 已转换                           |
| -i <interval>     | seconds between sending each packet [default: 1.0]           | 已转换                           |
| -W <timeout>      | time to wait for response [default: 1.0]                     | 已转换                           |
| -t <ttl>          | define time to live [default: 64]                            | 已转换                           |
| -s <size>         | use <size> as number of data bytes to be sent [default: 56]  | 已转换                           |
| -v                | verbose output                                               | 已转换                           |
| -q                | quiet output                                                 | 已转换                           |
| -b                | allow pinging broadcast                                      | 已转换                           |
| -D                | print timestamps                                             | 已转换                           |
| -h, --help        | print help                                                   | 已转换                           |
| -V, --version     |                                                              | 已转换                           |
| -a                | use audible ping                                             | 已转换                           |
| -A                | use adaptive ping                                            | 已转换                           |
| -B                | sticky source address                                        | 已转换                           |
| -C                | call connect() syscall on socket creation                    | 已转换                           |
| -d                | use SO_DEBUG socket option                                   | 已转换                           |
| -e <identifier>   | define identifier for ping session, default is random for<br/>SOCK_RAW and kernel defined for SOCK_DGRAM<br/>Imply using SOCK_RAW (for IPv4 only for identifier 0) | 已转换                           |
| -f                | flood ping                                                   | 已转换                           |
| -I <interface>    | either interface name or address                             | 已转换                           |
| -L                | suppress loopback of multicast packets                       | 已转换                           |
| -l <preload>      | send <preload> number of packages while waiting replies      | 已转换                           |
| -m <mark>         | tag the packets going out                                    | 已转换                           |
| -M <pmtud opt>    | define mtu discovery, can be one of <do\|done\|want>         | 已转换                           |
| -n                | no dns name resolution                                       | 未转换(默认就是禁用的，不转换)   |
| -O                | report outstanding replies                                   | 已转换                           |
| -p <pattern>      | contents of padding byte                                     | 已转换                           |
| -Q <tclass>       | use quality of service <tclass> bits                         | 已转换                           |
| -S <size>         | use <size> as SO_SNDBUF socket option value                  | 已转换                           |
| -U                | print user-to-user latency                                   | 已转换（与默认显示的rtt没差别）  |
| -w <deadline>     | reply wait <deadline> in seconds                             | 已转换                           |
|                   |                                                              | 未转换                           |
| -R                | record route                                                 | 已转换                           |
| -T <timestamp>    | define timestamp, can be one of <tsonly\|tsandaddr\|tsprespec> | 已转换（只支持ipv4，验证有问题） |
| -F <flowlabel>    | define flow label, default is random                         | 已转换（只支持ipv6，未生效）     |
| -N <nodeinfo opt> | use icmp6 node info query, try <help> as argument            | 只加了入口（内部还未实现）       |



1. 转换 iputils 项目，该项目输出5个二进制文件，本周主要转换其中一个核心命令 ping 
2. 已初步转换ping命令完成，可以正常运行，ping通指定的ipv4地址。可选参数控制还未开始转换
3. 转换rust源码行数，大约1500（可编译通过运行）行左右：整体iputils项目进展20%左右



###  第二周（2025/02/24-2025/02/28）

> 可使用命令来监听发包的信息：sudo tcpdump -i any icmp and host www.baidu.com

1. ping 命令支持 ipv6 选项，能ping通本地的IPv6回环地址（::1），ping命令的可选参数（20/ 36）转换了60%左右。
2.  转换源码行数，约3500行左右：整体iputils项目进展40%左右

###  第三周（2025/03/03-2025/03/07）

> `arping` 的核心功能是通过 **ARP 协议**检测局域网内的 IPv4 主机的活跃状态或 MAC 地址绑定

参数转换支持列表：

| 原始可选参数   | 参数描述                                           | rust版本参数支持情况      |
| -------------- | -------------------------------------------------- | ------------------------- |
| <destination>  | dns name or ip address                             | 已转换                    |
| -f             | quit on first reply                                | 已转换                    |
| -q             | be quiet                                           | 已转换                    |
| -b             | keep on broadcasting, do not unicast               | 已转换                    |
| -D             | duplicate address detection mode                   | 已转换                    |
| -U             | unsolicited ARP mode, update your neighbours       | 已转换 (需要使用本机的IP) |
| -A             | ARP answer mode, update your neighbours            | 已转换 (需要使用本机的IP) |
| -c <count>     | how many packets to send                           | 已转换                    |
| -w <timeout>   | how long to wait for a reply                       | 已转换                    |
| -i  <interval> | seconds between sending each packet [default: 1.0] | 已转换                    |
| -I <device>    | which ethernet device to use                       | 已转换                    |



1. 完成ping 命令的翻译，所有参数均以转换完成，初步自测通过（-N/-F 未生效）。
2. 开始转换 arping 命令，完成arping命令转换。



###  第四周（2025/03/10-2025/03/14）



安装 `wireshark` 抓包工具，来判断发送或接收的包，是否符合预期。

```bash
sudo yum install wireshark
```

安装后，执行 `tshark` 来监听默认执行数据包

```bash
sudo tshark -i eno1 -Y "icmp" -T fields -e frame.time -e ip.src -e ip.dst -e icmp.type -e icmp.ident -e icmp.seq -e icmp.originate_timestamp -e icmp.receive_timestamp -e icmp.transmit_timestamp
```



可查看支持哪些字段提取显示：`tshark -G fields |  grep -i timestamp`

####  clockdiff 可选参数介绍

可执行命令抓取验证：

```bash
sudo tshark -i eno1 -Y "ip.options.timestamp" -V
```

- -o 

`clockdiff` 命令会使用 IP 时间戳选项和 ICMP Echo 请求来测量本地机器和远程机器之间的时钟差异。此时，IP 时间戳选项包含四个时间戳字段（每个字段 8 字节），并且这些字段被预先指定为两个地址之间的往返路径。

- -o1

使用 `-o1` 参数时，`clockdiff` 命令会使用三项 IP 时间戳选项和 ICMP Echo 请求来测量机器之间的时钟差异。此时，IP 时间戳选项包含三个时间戳字段（每个字段 8 字节），并且这些字段被预先指定为三个地址之间的路径。



####  tracepath 命令

> 通过发送 UDP 数据包并监听 ICMP 响应来追踪网络路径

| 原始可选参数  | 参数描述               | rust版本参数支持情况 |
| ------------- | ---------------------- | -------------------- |
| <destination> | dns name or ip address | 已转换               |
| -4            | use IPv4               | 已转换               |
| -6            | use IPv6               | 发送后接收不到       |
| -b            | print both name and ip | 已转换               |
| -l <length>   | use packet <length>    | 已转换               |
| -m <hops>     | use maximum <hops>     | 已转换               |
| -n            | no dns name resolution | 已转换               |
| -p <port>     | use destination <port> | 已转换               |



可使用 `tshark` 来抓取可选参数是否设置成功，比如端口号设置，包的长度。

```bash
sudo tshark -i eno1 -Y "icmp" -T fields -e frame.time -e ip.src -e udp.dstport -e udp.length

```

抓取本地回环，ipv6的包，参考命令：

```bash
sudo tshark -i lo -Y "ipv6" -T fields -e frame.time -e ipv6.src -e ipv6.dst -e udp.dstport -e udp.length -e ipv6.hlim
```



###  第五周 （2025/03/17-2025/03/21）

####   ifenslave 命令的转换

> `ifenslave` 控制 Linux 实现的并行运行多个网络接口，通过将多个网络接口绑定到一个主接口（bonding device）上，从而实现负载均衡和冗余

github上的源码在 RMerl/asuswrt-merlin.ng



创建两个虚拟网卡，并启用

```bash
sudo ip link add dummy0 type dummy
sudo ip link add dummy1 type dummy
sudo ip link set dummy0 up
sudo ip link set dummy1 up

# 创建 bond0 接口
sudo ip link add bond0 type bond mode active-backup
# 激活 bond0
sudo ip link set bond0 up

# 添加从节点
sudo ifenslave bond0 dummy0 dummy1

# 为 bond0 分配 IP
sudo ip addr add 192.168.1.100/24 dev bond0
```



清理虚拟接口和Bonding接口

```bash
sudo ip link del dummy0
sudo ip link del dummy1
sudo ip link del bond0
```



## zip 项目

### 项目分析

该项目生成了以下四个主要的二进制文件：

1. `zip`：主要的ZIP压缩和解压缩工具。
2. `zipnote`：用于编辑ZIP文件注释的工具。
3. `zipsplit`：用于将大型ZIP文件拆分成多个较小文件的工具。
4. `zipcloak`：用于加密和解密ZIP文件的工具



以下是每个二进制文件使用的源文件列表：

1. **zip**：
   - 主源文件：`zip.o zipfile.o zipup.o fileio.o util.o globals.o crypt.o ttyio.o unix.o crc32.o zbz2err.o`
   - 其他源文件：`deflate.o trees.o`
   - 总共使用了 12 个源文件。
2. **zipnote**：
   - 主源文件：`zipnote.o`
   - 其他源文件：`zipfile_.o fileio_.o util_.o globals.o unix_.o`
   - 总共使用了 6 个源文件。
3. **zipcloak**：
   - 主源文件：`zipcloak.o`
   - 其他源文件：`zipfile_.o fileio_.o util_.o globals.o unix_.o crc32_.o crypt_.o ttyio.o`
   - 总共使用了 8 个源文件。
4. **zipsplit**：
   - 主源文件：`zipsplit.o`
   - 其他源文件：`zipfile_.o fileio_.o util_.o globals.o unix_.o`
   - 总共使用了 6 个源文件。

### 参数介绍

#### **基本用法**

`zip` 工具用于将文件打包并压缩到 ZIP 归档文件中。默认行为是添加或替换归档文件中的条目。

##### **基本命令行语法**

```bash
zip [选项] 归档文件名 文件1 文件2 ...
```

##### **示例**

- 将 file.txt 添加到 z.zip 中（如果 z.zip不存在则创建）：

  ```bash
  zip z file.txt
  ```

- 压缩当前目录下的所有文件：

  ```bash
  zip z *
  ```

- 压缩当前目录及其子目录中的所有文件：

  ```bash
  zip -r z .
  ```

------

#### **基本模式**

##### **外部模式（从文件系统中选择文件）**

- **`add`**（默认）：添加新文件或更新归档文件中的现有文件。
- **`-u`（`update`）**：仅添加新文件或更新日期较新的文件。
- **`-f`（`freshen`）**：仅更新归档文件中的现有文件（不添加新文件）。
- **`-FS`（`filesync`）**：如果文件日期或大小发生变化则更新，如果文件在操作系统中不存在则删除归档文件中的条目。

##### **内部模式（从归档文件中选择条目）**

- **`-d`（`delete`）**：从归档文件中删除文件。
- **`-U`（`copy`）**：选择归档文件中的条目复制到新归档文件中（需与 `--out` 选项一起使用）。

------

#### **基本选项**

- **`-r`**：递归进入目录。
- **`-m`**：在创建归档文件后删除原始文件（将文件移动到归档文件中）。
- **`-j`**：忽略目录名，仅存储文件名。
- **`-q`**：静默操作。
- **`-v`**：详细操作（仅 `zip -v` 显示版本信息）。
- **`-c`**：为每个条目添加一行注释。
- **`-z`**：为归档文件添加注释（以 `.` 或 EOF 结束）。
- **`-@`**：从标准输入读取要压缩的文件名（每行一个路径）。
- **`-o`**：使归档文件的日期与最新条目的日期相同。

------

#### **高级功能**

##### **通配符**

- **`?`**：匹配任意单个字符。
- **`\*`**：匹配任意数量的字符（包括零个字符）。
- **`[list]`**：匹配列表中的字符（支持范围，如 `[a-c]` 或排除 `[!b]`）。

##### **包含与排除**

- **`-i`**：仅包含匹配指定模式的文件。
- **`-x`**：排除匹配指定模式的文件。

##### **日期过滤**

- **`-t date`**：排除指定日期之前的文件（包含该日期及之后的文件）。
- **`-tt date`**：包含指定日期之前的文件。

##### **删除与文件同步**

- **`-d`**：从归档文件中删除匹配指定模式的条目。
- **`-FS`**：文件同步模式，更新日期或大小不匹配的条目，并删除操作系统中不存在的条目。

##### **压缩**

- **`-0`**：仅存储文件（不压缩）。
- **`-1` 到 `-9`**：压缩速度从最快到最佳（默认是 `-6`）。
- **`-Z cm`**：设置压缩方法（`store`、`deflate`、`bzip2`）。

##### **加密**

- **`-e`**：使用标准 PKZip 2.0 加密（提示输入密码）。
- **`-P pswd`**：使用标准加密，密码为 `pswd`。

##### **分卷归档**

- **`-s ssize`**：创建分卷归档，每个分卷大小为 `ssize`（例如 `100k` 表示 100 KB）。
- **`-sp`**：在每个分卷关闭后暂停（允许更换磁盘）。
- **`-sb`**：暂停时响铃。
- **`-sv`**：详细显示分卷创建过程。

##### **输出到新归档**

- **`--out oa`**：将结果输出到新归档文件 `oa` 中（不修改输入归档文件）。

##### **复制模式**

- **`-U`**：从旧归档文件中选择条目复制到新归档文件中。

##### **流式处理与 FIFO**

- **`prog1 | zip -ll z -`**：将 `prog1` 的输出压缩到 `z.zip` 中，并转换行尾符。
- **`zip - -R "\*.c" | prog2`**：压缩当前目录下的 `.c` 文件并流式传输到 `prog2`。

##### **日志记录**

- **`-lf path`**：将日志写入指定文件（覆盖现有文件）。
- **`-la`**：追加到现有日志文件。
- **`-li`**：在日志中包含信息消息（默认仅包含警告和错误）。

##### **测试归档**

- **`-T`**：在更新归档文件之前使用 `unzip` 测试临时归档文件。
- **`-TT cmd`**：使用指定命令测试归档文件（默认是 `unzip -tqq`）。

##### **修复归档**

- **`-F`**：尝试修复基本完整的归档文件（优先使用）。
- **`-FF`**：尝试修复损坏的归档文件（可能不完整）。

##### **Unicode 支持**

- **`-UN=Quit`**：如果 Unicode 路径与标准路径不匹配则退出。
- **`-UN=Warn`**：如果 Unicode 路径与标准路径不匹配则警告。
- **`-UN=Ignore`**：如果 Unicode 路径与标准路径不匹配则忽略。
- **`-UN=No`**：忽略所有 Unicode 路径。

------

#### **其他选项**

- **`-b dir`**：在指定目录中创建临时归档文件。
- **`-MM`**：输入模式必须匹配至少一个文件，且匹配的文件必须可读。
- **`-nw`**：禁用通配符（将通配符视为普通字符）。
- **`-sc`**：显示处理后的命令行参数并退出。
- **`-sd`**：显示调试信息。
- **`-so`**：显示当前系统支持的所有选项。

------

#### **注意事项**

- **`-FS`** 和 **`--out`** 选项会删除或覆盖文件，请谨慎使用。
- 分卷归档无法更新，需使用 **`--out`** 创建新归档文件。
- 流式处理创建的归档文件可能不兼容某些解压工具。

### 第六周（2025/03/25-2025/03/29）

开始进行 zip 项目的梳理，决定采用 rust 三方库 zip来封装实现

TODO: 默认add文件时，zip文件中已有的同名文件要进行更新替换。-u update 是采用的临时压缩文件，同步。add未处理该场景。

- 完成添加文件到压缩包的主命令

- 完成 -u 参数的验证
- 完成 -f 参数验证
- 完成 -FS 参数，实际实现为 -S 短参数
- 



### 第七周（2025/03/31-2025/04/03）

add完成新的文件处理方式。

单个文件的压缩率计算done

bug：压缩文件中存在加密文件，再次对该压缩文件进行更新，将加密文件拷贝到新的文件中后，无法打开该文件了。





** 引入上游库，是以什么形式存在到二进制的？ **

> 在 Rust 项目中，第三方库的引入和最终二进制文件的构成是这样的：
>
> 1. **依赖库的存在形式**：
>    - 在开发阶段：以源代码形式存在，存储在 `Cargo.lock` 和 `~/.cargo/registry` 中
>    - 在编译阶段：会被编译成静态库（`.rlib` 文件）或动态库（视情况而定）
>    - 在最终二进制中：默认情况下会被静态链接到你的可执行文件中
> 2. **二进制文件中的内容**：
>    - 默认情况下，Rust 会进行静态链接，所有用到的第三方库代码都会被编译进最终的可执行文件
>    - 不会额外引入 `.dll` 或 `.so` 等动态库文件（除非显式使用了 `dylib` 类型的依赖）



utzip 相比 原生的 zip 依赖多了：

- /usr/lib64/liblzma.so.5        （xz-libs-5.4.4-2.uos25.01.x86_64）
- /usr/lib64/libm.so.6            （glibc-2.38-36.uos25.07.x86_64）
- /usr/lib64/libgcc_s.so.1      （libgcc-12.3.1-35.uos25.08.x86_64）





我节前节后请假，回老家一趟，总共调休两天。
本周utzip项目进展：

1. 完成压缩文件的添加、更新、压缩率的计算。

2. 整体进展15%。目前自行实现的代码行数约2000行（zip），整体完整功能（zipsplist/zipnote/zipcloak）实现预期7000行左右



## 第八周 （2025/04/08-2025/04/11）

1. 完成删除条目
2. TODO: -U copy 参数待实现





下载rpm源码，需要先添加源码仓库，在 `/etc/yum.repos.d/uos.repo`

> [UnionTechOS-$releasever-source] 
> name = UnionTechOS $releasever source
> baseurl = https://cdimage.uniontech.com/server-dev/v25/2500/SubTR6/compose/everything/source/tree/
> enabled = 1 
> gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-uos-release
> gpgcheck = 1 
> skip_if_unavailable = 1 



```
 sudo yum makecache
 dnf download --source grep
```

安装源码包，安装后源码通常会被解压到 `~/rpmbuild/SOURCES/` 目录

```bash
rpm -ivh <package-name>.src.rpm
```

然后打上补丁

```
rpmbuild -bp ~/rpmbuild/SPECS/grep.spec
```



iproute: 支持的子命令太多

grep：

diffutils: 源码量最大

parted: 操作分区，原理较为复杂



严格拷贝文件

```bash
 rsync -avP --delete --exclude='.vscode' --exclude='target' --exclude='.git' --exclude='Cargo.lock' amd@10.8.12.109:/home/amd/xiaolong/utiputils/  ./
```



 

### zip项目分析 (不采用上游三方库，自己翻译)

主要用于创建和管理ZIP格式的压缩文件。以下是项目主要目录和文件的简要分析：

#### 主要目录结构分析

1. **平台相关目录**：
   - `aosvs/` - AOS/VS操作系统相关代码
   - `amiga/` - Amiga系统移植代码
   - `beos/` - BeOS系统移植代码
   - `msdos/` - MS-DOS系统移植代码
   - `os2/` - OS/2系统移植代码
   - `win32/` - Windows系统移植代码（包含VC6项目文件）
   - `unix/` - Unix/Linux系统移植代码
   - `vms/` - VMS系统移植代码
   - `macos/` - Mac OS系统移植代码
2. **核心代码目录**：
   - `bzip2/` - BZIP2压缩算法实现
   - `src/` - 核心源代码（可能）
   - `windll/` - Windows DLL接口代码
3. **文档相关**：
   - `man/` - 手册页(man pages)
   - `proginfo/` - 程序信息文件
4. **特殊平台支持**：
   - `atari/` - Atari系统支持
   - `qdos/` - QDOS系统支持
   - `tandem/` - Tandem系统支持
   - `theos/` - THEOS系统支持

#### 重要文件说明

1. **核心源代码文件**：
   - `zip.c` - 主程序入口
   - `crc32.c` - CRC32校验计算
   - `deflate.c` - 压缩算法实现
   - `fileio.c` - 文件I/O操作
   - `globals.c` - 全局变量定义
2. **项目构建文件**：
   - `Makefile` - 各平台下的构建脚本
   - `win32/vc6/*.dsp` - VC6项目文件
   - `win32/vc6bz2/` - 带BZIP2支持的VC6项目
3. **文档和说明文件**：
   - `README` - 基本说明
   - `CHANGES` - 版本变更记录
   - `LICENSE` - 许可信息
   - `TODO` - 待办事项
   - `WHATSNEW` - 新特性说明
