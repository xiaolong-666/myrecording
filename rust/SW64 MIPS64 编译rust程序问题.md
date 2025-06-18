# SW64 MIPS64 编译rust程序问题

## SW64 编译问题

1. 在最新的V25系统上安装开发环境： `sudo apt cargo` ，安装cargo时，会自动安装上 rustc 的对应版本。

2. 使用 cargo 命令 初始化一个 helloworld 项目，并添加 libc 的依赖 (c 转换为 rust项目)，该库为核心库，很多三方库底层都依赖 它。此处只是为了方便，添加了一个依赖。实际项目中还使用了很多库。

   ```bash
   uos@uos-PC:~/rust$ cargo init xiaolong-test
       Creating binary (application) package
   note: see more `Cargo.toml` keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
   # 添加libc依赖
   uos@uos-PC:~/rust/xiaolong-test$ cargo add libc
   # 初始项目组织如下：
   uos@uos-PC:~/rust/xiaolong-test$ tree ./
   ./
   ├── Cargo.toml
   └── src
       └── main.rs
   
   1 directory, 2 files
   # 主程序如下：
   uos@uos-PC:~/rust/xiaolong-test$ cat src/main.rs 
   fn main() {
       println!("Hello, world!");
   }
   # 依赖文件内容如下：
   uos@uos-PC:~/rust/xiaolong-test$ cat Cargo.toml 
   [package]
   name = "xiaolong-test"
   version = "0.1.0"
   edition = "2021"
   
   [dependencies]
   libc = "0.2.172"
   
   # 编译
   uos@uos-PC:~/rust/xiaolong-test$ cargo build
   ```

2. 编译现象：

   > error[E0425]: cannot find value `O_NONBLOCK` in this scope
   >    --> /home/uos/.cargo/registry/src/index.crates.io-6f17d22bba15001f/libc-0.2.172/src/unix/linux_like/linux/gnu/mod.rs:753:34
   >     |
   > 753 | pub const SOCK_NONBLOCK: c_int = O_NONBLOCK;
   >     |                                  ^^^^^^^^^^ not found in this scope
   >
   > error[E0425]: cannot find value `O_NONBLOCK` in this scope
   >    --> /home/uos/.cargo/registry/src/index.crates.io-6f17d22bba15001f/libc-0.2.172/src/unix/linux_like/linux/gnu/mod.rs:754:36
   >     |
   > 753 | pub const SOCK_NONBLOCK: c_int = O_NONBLOCK;
   >     | -------------------------------------------- similarly named constant `SOCK_NONBLOCK` defined here
   > 754 | pub const PIDFD_NONBLOCK: c_uint = O_NONBLOCK as c_uint;
   >     |                                    ^^^^^^^^^^ help: a constant with a similar name exists: `SOCK_NONBLOCK`
   >
   > error[E0425]: cannot find value `EOPNOTSUPP` in this scope
   >    --> /home/uos/.cargo/registry/src/index.crates.io-6f17d22bba15001f/libc-0.2.172/src/unix/linux_like/linux/gnu/mod.rs:791:28
   >     |
   > 791 | pub const ENOTSUP: c_int = EOPNOTSUPP;
   >     |                            ^^^^^^^^^^ not found in this scope
   >
   > ...
   >
   > error[E0412]: cannot find type `sigset_t` in the crate root
   >     --> /home/uos/.cargo/registry/src/index.crates.io-6f17d22bba15001f/libc-0.2.172/src/unix/linux_like/linux/gnu/mod.rs:1466:32
   >      |
   > 1466 |         sigmask: *const crate::sigset_t,
   >      |                                ^^^^^^^^ not found in the crate root
   >
   > error[E0412]: cannot find type `rlim_t` in the crate root
   >    --> /home/uos/.cargo/registry/src/index.crates.io-6f17d22bba15001f/libc-0.2.172/src/unix/linux_like/linux/arch/generic/mod.rs:380:33
   >     |
   > 380 | pub const RLIM_INFINITY: crate::rlim_t = !0;
   >     |                                 ^^^^^^ help: a struct with a similar name exists: `rlimit`
   >     |
   >    ::: /home/uos/.cargo/registry/src/index.crates.io-6f17d22bba15001f/libc-0.2.172/src/macros.rs:118:13
   >     |
   > 118 |             pub struct $i { $($field)* }
   >     |             ------------- similarly named struct `rlimit` defined here
   >
   > Some errors have detailed explanations: E0412, E0425, E0573.
   > For more information about an error, try `rustc --explain E0412`.
   > error: could not compile `libc` (lib) due to 257 previous errors

   可以看到，空的项目，只是添加了官方的 libc 依赖，就报了257个错误。该库作为核心官方库，我们无法解决。

## MIPS64 编译问题

1. 在最小的V25系统上安装开发环境： `sudo apt install cargo`
2. 初始化一个 helloworld 项目，添加部分项目中用到的依赖 libc 、pnet。 此处只是为了方便，添加了两个依赖。实际项目中还使用了很多库。

```bash
mips@mips-PC:~/rust$ cargo init xiaolong-test
    Creating binary (application) package
note: see more `Cargo.toml` keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
# 添加libc依赖
mips@mips-PC:~/rust/xiaolong-test$ cargo add libc pnet
# 初始项目组织如下：
mips@mips-PC:~/rust/xiaolong-test$ tree ./
./
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
# 主程序如下：
mips@mips-PC:~/rust/xiaolong-test$ cat src/main.rs 
fn main() {
    println!("Hello, world!");
}
# 依赖文件内容如下：
uos@uos-PC:~/rust/xiaolong-test$ cat Cargo.toml 
[package]
name = "xiaolong-test"
version = "0.1.0"
edition = "2021"

[dependencies]
libc = "0.2.172"
pnet = "0.35.0"

# 编译
mips@mips-PC:~/rust/xiaolong-test$ cargo build
```

3. 编译报错，底层间接依赖，需要高版本的 rustc：

   > mips@mips-PC:~/xiaolong-test$ cargo build 
   > error: package `regex-automata v0.4.9` cannot be built because it requires rustc 1.65 or newer, while the currently active rustc version is 1.63.0
   > Either upgrade to rustc 1.65 or newer, or use
   > cargo update -p regex-automata@0.4.9 --precise ver
   > where `ver` is the latest version of `regex-automata` supporting rustc 1.63.0

当前mips上的rustc版本为 1.63 。与其它架构的版本不一致，amd64、arm64、loong64、sw64的架构均为 1.81。

```bash
mips@mips-PC:~/xiaolong-test$ apt policy rustc
rustc:
  已安装：1.63.0+dfsg1-2
  候选： 1.63.0+dfsg1-2
  版本列表：
 *** 1.63.0+dfsg1-2 500
        500 http://pools.uniontech.com/desktop-professional-V25 snipe/main mips64el Packages
        100 /usr/lib/dpkg/var/status

# 对比其它架构的版本，差距过大
uos@uos-PC:~$ apt policy rustc
rustc:
  已安装：1.81
  候选： 1.81
  版本列表：
 *** 1.81 500
        500 https://professional-packages.chinauos.com/desktop-professional-V25 snipe/main sw64 Packages
        500 http://pools.uniontech.com/desktop-professional-V25 snipe/main sw64 Packages
        100 /usr/lib/dpkg/var/status

```

为了保持同源异构，不针对mips架构额外去降级全部的依赖版本，且降级到低版本库，会导致现有源码大调整。故mips架构需要同步升级rustc到1.81，与其它架构保持一致。

