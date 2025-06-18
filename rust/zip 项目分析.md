# zip 项目分析

这个Makefile文件是用于构建Zip工具套件的Unix版本，包括zip、zipnote、zipcloak和zipsplit四个主要程序。以下是对该Makefile的分析：

### 使用的源文件数量

1. **核心源文件**：
   - `zip.c` - 主程序
   - `zipfile.c` - ZIP文件处理
   - `zipup.c` - 文件打包
   - `fileio.c` - 文件I/O操作
   - `util.c` - 实用函数
   - `globals.c` - 全局变量
   - `crypt.c` - 加密功能
   - `ttyio.c` - 终端I/O
   - `unix/unix.c` - Unix平台特定代码
   - `crc32.c` - CRC校验
   - `zbz2err.c` - BZIP2错误处理
   - `deflate.c` - 压缩算法
   - `trees.c` - 压缩树结构
2. **汇编文件**：
   - `match.S` - 匹配算法(汇编优化)
   - `crc_i386.S` - CRC32校验(汇编优化)
3. **头文件**：
   - `zip.h` - 主头文件
   - `ziperr.h` - 错误定义
   - `tailor.h` - 平台适配
   - `unix/osdep.h` - Unix系统依赖
   - `crc32.h` - CRC32定义
   - `crypt.h` - 加密定义
   - `ttyio.h` - 终端I/O定义
   - `revision.h` - 版本信息
   - `unix/zipup.h` - 打包功能定义
4. **手册文件**：
   - `man/zip.1`
   - `man/zipcloak.1`
   - `man/zipnote.1`
   - `man/zipsplit.1`

### 生成的二进制文件数量

1. **主要可执行程序**：
   - `zip` - 主压缩程序
   - `zipnote` - ZIP注释编辑器
   - `zipcloak` - ZIP加密工具
   - `zipsplit` - ZIP分割工具
2. **中间目标文件**：
   - 每个.c文件对应一个.o文件
   - 特殊处理的_.o文件(用于工具程序)
   - 汇编生成的match.o和crc_i386.o
3. **BZIP2库**：
   - `bzip2/libbz2.a` - 可选压缩库
4. **文档文件**：
   - `zip.txt`
   - `zipcloak.txt`
   - `zipnote.txt`
   - `zipsplit.txt`

### 总结

- **使用的源文件总数**：约15-20个核心C文件 + 2个汇编文件 + 多个头文件
- **生成的二进制文件**：4个主要可执行程序 + 多个中间目标文件 + 可选库文



## 4.11 记录

自己实现的zipWriter 对于Store模式还是会增加压缩头，待修改

## 4.14 - 4.19 记录

1. **压缩效率判断**：Zip会评估文件压缩后的效果，如果压缩后大小几乎没有减少(甚至可能变大)，则会自动选择Stored方式
2. **文件大小阈值**：虽然没有明确的固定阈值，但通常小文件(如小于100字节)会被直接存储，因为压缩带来的收益很小
3. **文件类型判断**：某些已经压缩过的文件格式(如.zip/.gz/.jpg等)默认会直接存储





## utzip基本命令测试：

生成的压缩文件，可使用 unzip -l xxx.zip 查看内容。

### 基本模式

- add 新增压缩文件

```bash
utzip test.zip aaa.txt bbb.txt
```

- update (-u) 更新已经存在的文件，压缩包不存的则会添加

```bash
utzip -u test.zip aaa.txt bbb.txt ccc.txt
```

- freshen (-f) 只更新压缩文件中存在的文件，不新增文件

```bash
uztip -f test.zip
```

- filesync (--FS) 当文件的时间或大小改变，或不匹配的则删除，严格同步

```bash
utzip --FS test.zip src/
```

- delete (-d) 从压缩文件中删除指定的文件

```bash
utzip test.zip -d aaa.txt
```

- copy (-U) 从压缩文件中拷贝指定的文件，并生成新的压缩文件，需要搭配 --out 参数一起使用

```bash
utzip test.zip -U aaa.txt --out test2.zip
```

### 基础选项

- -r / -R：-r 递归目录。-R "*.rs" 递归当前目录下的所有匹配文件，忽略目录

```bash
utzip test.zip -r src # 递归添加src目录下的所有文件到test.zip文件中


utzip test.zip ../zipsplit.idx  -R "*.toml" -v
updating: Cargo.toml  (in=945) (out=474) (Deflated 49.84%)
updating: rust-toolchain.toml  (in=31) (out=31) (Stored 0.00%)
adding: ../zipsplit.idx  (in=1101) (out=252) (Deflated 77.11%)
total bytes=2077, compressed=757 -> 63.55% savings

```



- -m ： 移动文件到压缩文件中

```bash
utzip -m test.zip -r src # 执行完毕后，src目录会被删除，所有数据已经到压缩文件中了
```

- -j：使用文件名，不带有目录的，

```bash
utzip test2.zip -j -r src
zip warning:    first full name: src/utils/mod.rs
                second full name: src/encryption/mod.rs
                name in zip file repeated: mod.rs
                this may be a result of using -j

Error: zip error: Invalid command arguments (cannot repeat names in zip file)

```

- -q: 安静模式

```bash
utzip test2.zip -j -r src -q
Error: zip error: Invalid command arguments (cannot repeat names in zip file)

```

- -v: 详细模式（不跟任何参数则打印版本信息）

```
utzip -v test.zip src -r -u
zip diagnostic: src/bin/zipsplit.rs up to date
updating: src/cli.rs  (in=10991) (out=3505) (Deflated 68.11%)
...
zip diagnostic: src/zip.rs up to date
total bytes=150879, compressed=41915 -> 72.22% savings

```

- -c : 文件注释（为操作的文件交互式添加注释内容）

```bash
utzip test.zip src -v -c
updating: src/cli.rs  (in=11025) (out=3524) (Deflated 68.04%)
updating: src/error.rs  (in=1030) (out=385) (Deflated 62.62%)
updating: src/lib.rs  (in=94) (out=57) (Deflated 39.36%)
updating: src/main.rs  (in=1802) (out=764) (Deflated 57.60%)
updating: src/zip.rs  (in=48665) (out=9903) (Deflated 79.65%)
Enter comment for src/cli.rs (press Enter to skip):
cli
Enter comment for src/error.rs (press Enter to skip):
error
Enter comment for src/lib.rs (press Enter to skip):
lib
Enter comment for src/main.rs (press Enter to skip):
main
Enter comment for src/zip.rs (press Enter to skip):
zip
total bytes=62616, compressed=14633 -> 76.63% savings

```

添加完成后，可以使用 7z 来查看详细信息 `7z l -slt test.zip ` 

- -z：归档文件注释

```bash
[amd@localhost utzip]$ RUST_LOG= ./target/debug/utzip test1.zip src/zip.rs -zv
adding: src/zip.rs (in=50720) (out=10229) (Deflated 79.83%)
Enter new zip file comment (end with .):
xiaolong 
hello
test
.
total bytes=50720, compressed=10229 -> 79.83% savings

```

- -@ : 用于从标准输入(stdin)读取要压缩的文件列表

```bash
echo -e "file1.txt\nfile2.txt" | zip archive.zip -@
```

- -o: 将压缩文件的时间戳设置为压缩文件里面的最新时间戳

```bash
utzip -o  test.zip src/utils/file_system.rs src/utils/mod.rs -vr 
adding: src/utils/file_system.rs  (in=2) (out=2) (Stored 0.00%)
adding: src/utils/mod.rs  (in=57) (out=51) (Deflated 10.53%)
total bytes=59, compressed=53 -> 10.17% savings

...
# 使用 7z l ../test.zip 查看
Path = ../test.zip
Type = zip
Physical Size = 307

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2025-04-01 10:51:00 .....            2            2  src/utils/file_system.rs
2025-03-27 16:09:42 .....           57           51  src/utils/mod.rs
------------------- ----- ------------ ------------  ------------------------
2025-04-01 10:51:00                 59           53  2 files
[amd@localhost tmp]$ ls -lh ../test.zip 
-rw-r--r-- 1 amd amd 307  4月 1日 10:51 ../test.zip

```

### 其它选项

- -0 / -1 /-9： 压缩级别，0是不压缩、1是快速压缩、9是最高压缩。-0 -> -9 都支持

  ```bash
  utzip test.zip src -r -0 -v
  adding: src/  (in=0) (out=0) (Stored 0.00%)
  adding: src/bin/  (in=0) (out=0) (Stored 0.00%)
  adding: src/bin/zipsplit.rs  (in=705) (out=705) (Stored 0.00%)
  adding: src/cli.rs  (in=11025) (out=11025) (Stored 0.00%)
  adding: src/commands/  (in=0) (out=0) (Stored 0.00%)
  adding: src/commands/add.rs  (in=9859) (out=9859) (Stored 0.00%)
  ...
  ```

- -Z : 设置压缩方式，可选值为：`store` `deflate` `bzip2` 

  ```bash
  utzip test.zip Cargo* -Z bzip2
  updating: Cargo.toml (Bzip2 44.43%)
  updating: Cargo.lock (Bzip2 78.05%)
  updating: Cargo-test.TOML (Bzip2 43.92%)
  
  ```

- -l /--ll ：转换文本文件行结束符从LF(Unix风格)转换为CRLF(Windows风格); --ll 作用相反，将CRLF转换为LF

  ```bash
  utzip test1.zip src/main.rs -v -l
  updating: src/main.rs  (in=1799) (out=763) (Deflated 57.59%)
  total bytes=1799, compressed=763 -> 57.59% savings
  
  ```
  
  转换后解压文件，使用 file 命令检查类型： `file src/main.rs` 
  
  ```bash
  file src/main.rs 
  src/main.rs: C source, Unicode text, UTF-8 text, with CRLF line terminators
  ```
  
  
  
  - Windows格式会显示"with CRLF line terminators"
  - Unix格式会显示"with LF line terminators"
  
- -i / -x: 都是文件筛选的选项，但功能相反

  > `-i`和`-x`筛选的行为取决于当前的操作模式：
  >
  > 1. **对于添加/更新文件的操作**（默认模式、-u更新、-f刷新）：
  >    - 筛选仅针对当前命令行指定的文件/路径
  >    - 不会影响归档文件中已有的条目（除非使用-FS文件同步模式）
  > 2. **对于删除操作**（-d）：
  >    - 筛选仅针对归档文件中已有的条目
  >    - 不能使用-i选项，只能使用-x排除特定条目
  > 3. **对于复制模式**（-U/--copy）：
  >    - 筛选针对归档文件中已有的条目
  >    - 可以同时使用-i和-x来精确选择要复制的条目
  >
  > 关键区别：
  >
  > - 外部操作（添加/更新）的筛选针对文件系统
  > - 内部操作（删除/复制）的筛选针对归档内容
  
  ```bash
  utzip test2.zip src -r  -i "*commands*" -x "*mod*" 
  updating: src/commands/ (0.00%)
  updating: src/commands/add.rs (75.37%)
  updating: src/commands/copy.rs (60.60%)
  updating: src/commands/delete.rs (62.97%)
  updating: src/commands/extract.rs (67.02%)
  updating: src/commands/fix.rs (74.47%)
  updating: src/commands/list.rs (66.37%)
  updating: src/commands/split.rs (29.15%)
  updating: src/commands/update.rs (77.15%)
  ```
  
- -D : （不添加目录条目）主要用于添加/更新文件的操作模式
  
  ```bash
  utzip test2.zip src -r -D
  updating: src/commands/add.rs (75.37%)
  adding: src/bin/zipsplit.rs (41.13%)
  adding: src/cli.rs (67.54%)
  adding: src/commands/copy.rs (60.60%)
  adding: src/commands/delete.rs (62.97%)
  adding: src/commands/extract.rs (67.02%)
  
  ```
  
- -A / -J: 用于处理自解压文件(SFX)的两个相关选项：`-A` (调整自解压文件偏移)，`-J` (移除自解压头)

  ```bash
  # 1. 创建自解压文件
  cat /usr/bin/unzipsfx data.zip > installer.exe
  
  # 2. 调整自解压文件偏移(可选)
  utzip -A installer.exe
  
  # 3. 还原为普通ZIP文件
  utzip -J installer.exe -out data_restored.zip
  ```

- -X : 移除 ZIP 文件中的额外字段(extra fields)

  使用 `unzip -Zv xxx.zip` 查看详细字段，使用该参数后，主要是额外的utc等时间没有

  ```bash
  Central directory entry #4:
  ---------------------------
  
    src/bin/
  
    offset of local header from start of archive:   21325
                                                    (000000000000534Dh) bytes
    file system or operating system of origin:      Unix
    version of encoding software:                   3.0
    minimum file system compatibility required:     MS-DOS, OS/2 or NT FAT
    minimum software version required to extract:   1.0
    compression method:                             none (stored)
    file security status:                           not encrypted
    extended local header:                          no
    file last modified on (DOS date/time):          2025 Apr 17 15:37:12
    file last modified on (UT extra field modtime): 2025 Apr 17 15:37:11 local ## 使用后没有该行
    file last modified on (UT extra field modtime): 2025 Apr 17 07:37:11 UTC  ## 使用后没有该行
    32-bit CRC value (hex):                         00000000
    compressed size:                                0 bytes
    uncompressed size:                              0 bytes
    length of filename:                             8 characters
    length of extra field:                          24 bytes
    length of file comment:                         0 characters
    disk number on which file begins:               disk 1
    apparent file type:                             binary
    Unix file attributes (040755 octal):            drwxr-xr-x
    MS-DOS file attributes (10 hex):                dir 
  ```

- -y: 将符号链接存储为链接本身而非目标文件 (默认是读取目标文件，使用后只保存链接本身，且不压缩)

  ```bash
  [amd@localhost tmp]$ utzip ut.zip zip.rs aaa.link -y
  adding: aaa.link (Stored 0.00%)
  adding: zip.rs (Deflated 80.19%)
  [amd@localhost tmp]$ ls -lh
  总计 100K
  lrwxrwxrwx 1 amd amd   16  4月17日 17:03 aaa.link -> xiaolong/aaa.txt
  
  ```

- -e / -P : 加密压缩文件，-e 交互输入密码，-P 预置密码，同时出现，-e 优先级更高

  ```bash
  utzip ut.zip zip.rs aaa.link -y -P 123 -e 
  Enter passwd: 
  Verify passwd: 
  updating: aaa.link (Stored 0.00%)
  updating: zip.rs (Deflated 80.17%)
  ```

- -n : 指定不压缩某些特定后缀名的文件。默认这些后缀不压缩 "Z:.zip:.zoo:.arc:.lzh:.arj"，使用该参数可追加

  ```bash
  utzip ut.zip zip.rs xiaolong/aaa.txt test1.exe -n ".exe:.txt"
  adding: test1.exe (Stored 0.00%)
  adding: xiaolong/aaa.txt (Stored 0.00%)
  adding: zip.rs (Deflated 80.19%)
  
  ```

- -T / -TT : 测试压缩文件。-T （更新ZIP文件前测试临时ZIP文件的完整性）， --TT "cmd" 使用指定命令测试文件的完整性

  ```bash
  utzip test.zip -T src -e -r
  Enter passwd: 
  Verify passwd: 
  updating: src/ (Stored 0.00%)
  updating: src/zip.rs (Deflated 79.69%)
  updating: src/bin/ (Stored 0.00%)
  [test.tmp] src/ password: 
  test of test.zip OK
  
  ```

- -t / -tt : 基于日期过滤文件，主要用于在 **添加(add)、更新(update)或刷新(freshen)文件 **时进行日期过滤

  - `-t` 选项（从指定日期开始包含文件）
  - `-tt` 选项（在指定日期之前包含文件）

  ```bash
  utzip test.zip src -r -t 04232025 --tt 2025-04-27 -v
  zip diagnostic: src/bin/ missing or early
  zip diagnostic: src/bin/zipcloak.rs missing or early
  zip diagnostic: src/zipnote.rs missing or early
  zip diagnostic: src/zipsplit.rs missing or early
  updating: src/  (in=0) (out=0) (Stored 0.00%)
  updating: src/lib.rs  (in=147) (out=72) (Deflated 51.02%)
  updating: src/bin/zipsplit.rs  (in=110) (out=94) (Deflated 14.55%)
  updating: src/encryption/zipcrypt.rs  (in=11297) (out=4386) (Deflated 61.18%)
  updating: src/commands/  (in=0) (out=0) (Stored 0.00%)
  updating: src/commands/mod.rs  (in=163) (out=124) (Deflated 23.93%)
  updating: src/zipcloak.rs  (in=5368) (out=1645) (Deflated 69.36%)
  total bytes=246874, compressed=63393 -> 74.32% savings
  
  ```

- -s / -sp / -sb / -sv : 分卷压缩包

  为了便于解析，短参数形式做了修改：-sp --> --sp    -sb --> --sb   -sv --> --sv

  --sp: 切换分卷时，暂停，可交互式输入保存位置

  --sv: 分卷详细输出

  --sb: 分卷时响铃

  ```bash
  [amd@localhost tmp]$ utzip test.zip ../target/debug/utzip -s 5m --sp --sv --sb
  splitsize = 5242880
        Closing split test.z01
  Opening disk 2
  Hit ENTER to write to default path of
    (current directory)
  or enter a new directory path (. for cur dir) and hit ENTER
  Path (or hit ENTER to continue): /home/amd/xiaolong/test/
  Writing to:
    /home/amd/xiaolong/test/
  Path (or hit ENTER to continue): 
        Closing split /home/amd/xiaolong/test/test.z02
  Opening disk 3
  Hit ENTER to write to default path of
    /home/amd/xiaolong/test/
  or enter a new directory path (. for cur dir) and hit ENTER
  Path (or hit ENTER to continue): 
  adding: ../target/debug/utzip (Deflated 69.29%)
  ```

- -lf / -li / -la ：保存日志文件到指定的文件中。当前实现均实现为长参数模式即：--lf / --li / --la

  --lf :  指定保存的文件路径
  
  --li：记录标准输出
  
  --la：追加到已经存在的文件中
  
  ```bash
  utzip test.zip ../src/utils/  -r -v --lf test.log --li
    adding: ../src/utils/ (in=0) (out=0)  (Stored 0.00%)
    adding: ../src/utils/common.rs (in=37702) (out=10299)  (Deflated 72.68%)
    adding: ../src/utils/logfile.rs (in=2652) (out=902)  (Deflated 65.99%)
    adding: ../src/utils/mod.rs (in=33) (out=33)  (Stored 0.00%)
    adding: ../src/utils/progress.rs (in=3244) (out=785)  (Deflated 75.80%)
  total bytes=43631, compressed=12019 -> 72.45% savings
  [amd@localhost tmp]$ cat test.log 
  ---------
  Zip log opened Tue May 13 20:01:25 2025
  command line arguments:
   test.zip ../src/utils/ -r -v --lf test.log --li
  
    adding: ../src/utils/ (in=0) (out=0)  (Stored 0.00%)
    adding: ../src/utils/common.rs (in=37702) (out=10299)  (Deflated 72.68%)
    adding: ../src/utils/logfile.rs (in=2652) (out=902)  (Deflated 65.99%)
    adding: ../src/utils/mod.rs (in=33) (out=33)  (Stored 0.00%)
    adding: ../src/utils/progress.rs (in=3244) (out=785)  (Deflated 75.80%)
  total bytes=43631, compressed=12019 -> 72.45% savings
  
  Total 5 entries (43K bytes)
  Tue May 13 20:01:25 2025
  ```
  
- --dif ：仅包含与输入存档相比已更改或新增的文件

  - 比较当前文件与参考存档(full_backup.zip)的差异
  - 只打包两种文件：
    - 新增的文件（参考存档中不存在的）
    - 修改过的文件（时间戳或文件大小发生变化）

  ```bash
  [amd@localhost tmp]$ utzip test.zip ../src -r --dif --out a.zip -v
    updating: ../src/commands/ (in=0) (out=0)  (Stored 0.00%)
    adding: ../src/commands/list.rs (in=3024) (out=1054)  (Deflated 65.15%)
    updating: ../src/commands/mod.rs (in=156) (out=117)  (Deflated 25.00%)
    updating: ../src/utils/common.rs (in=39009) (out=10628)  (Deflated 72.76%)
  total bytes=42189, compressed=11799 -> 72.03% savings
  ```

- --sf : 显示将要操作的文件列表后立即退出程序（也就是查看归档列表）

  --su：同时显示utf8 和 unicode 名字

  --sU:  只显示 unicode 名字。可以搭配 --UN Escape 查看

  ```bash
  [amd@localhost tmp]$ utzip src-test.zip --su --UN Escape
  Archive contains:
    ../src/
    ../src/cli.rs
  ...
    ../src/compression/deflate.rs
    ../src/compression/mod.rs
    ../src/compression/store.rs
    ../src/zipcloak.rs
    ../src/error.rs
    #U5C0F#U9F99.txt
      Escaped Unicode:  #U5C0F#U9F99.txt
  Total 36 entries (265014 bytes)
  
  ```

  

- --db/--dc/--dd/--dg/--ds/--du/--dv

  主要选项解释：

  1. `-db`：显示已处理的字节数和剩余字节数（未压缩大小，删除和复制操作显示存储大小）
  2. `-dc`：显示已处理的文件数和剩余文件数
  3. `-dd`：每处理10MB（或指定大小）显示一个点。单个文件统计大小
  4. `-dg`：为整个归档显示点，而不是为每个文件显示。整体统计大小，与 -dd 互斥。
  5. `-ds`：设置每个点代表处理的数据量大小（0表示不显示点）
  6. `-du`：显示每个添加条目的原始未压缩大小
  7. `-dv`：以"输入磁盘>输出磁盘"格式显示卷(磁盘)号

  ```bash
  [amd@localhost tmp]$ utzip test.zip ../target/release/utzip* --db --du --dc --dv -O c.zip --dg --ds 1m 
  1>1:   0/  8 [   0/ 16M]   adding: ../target/release/utzip .... (4.5M)  (Deflated 65%)
  1>1:   1/  7 [  5M/ 11M]   adding: ../target/release/utzip.d  (1.1K)  (Deflated 83%)
  1>1:   2/  6 [  5M/ 11M]   adding: ../target/release/utzipcloak ... (3.8M)  (Deflated 65%)
  1>1:   3/  5 [  8M/  7M]   adding: ../target/release/utzipcloak.d  (1.0K)  (Deflated 82%)
  1>1:   4/  4 [  8M/  7M]   adding: ../target/release/utzipnote ... (3.7M)  (Deflated 66%)
  1>1:   5/  3 [ 12M/  4M]   adding: ../target/release/utzipnote.d  (1.0K)  (Deflated 82%)
  1>1:   6/  2 [ 12M/  4M]   adding: ../target/release/utzipsplit ... (3.8M)  (Deflated 66%)
  1>1:   7/  1 [ 16M/  1K]   adding: ../target/release/utzipsplit.d  (1.0K)  (Deflated 83%)
  
  ```

  

- --sc ：显示输入的命令和参数后退出

  ```bash
  [amd@localhost tmp]$ utzip src-test.zip ../src/* ../src/commands/add.rs ../src/utils/progress.rs ../target/release/utzipcloak  --db --du --dc --dv -O c.zip --dg --ds 1m  -u --sc
  Command line:
  utzip src-test.zip ../src/bin ../src/cli.rs ../src/commands ../src/compression ../src/encryption ../src/error.rs ../src/lib.rs ../src/main.rs ../src/utils ../src/zipcloak.rs ../src/zipnote.rs ../src/zip.rs ../src/zipsplit.rs ../src/commands/add.rs ../src/utils/progress.rs ../target/release/utzipcloak --db --du --dc --dv -O c.zip --dg --ds 1m -u --sc
  Error: zip error: Interrupted (show command line)
  
  ```

- --nw : 禁用通配符

  ```bash
  [amd@localhost tmp]$ utzip z.zip -d "*.dat" -O y.zip --nw
    deleting: *.dat 
  [amd@localhost tmp]$ utzip z.zip -d "*.dat" -O y.zip
    deleting: *.dat 
    deleting: b.dat 
  ```

- --ws: 通配符不跨目录(默认跨目录)

  ```bash
  [amd@localhost tmp]$ utzip test.zip '*.log'
    updating: src-test.log  (Deflated 72%)
    updating: test.log  (Deflated 74%)
    updating: docs/test 2.log  (Deflated 74%)
    updating: docs/test.log  (Deflated 74%)
  [amd@localhost tmp]$ utzip test.zip '*.log' --ws
    updating: src-test.log  (Deflated 72%)
    updating: test.log  (Deflated 74%)
  
  ```

  

- --UN：(只在 --sf 里面显示用了，正常来说 保存文件名和注释内容应该也要，但验证不生效) TODO

- -F / -FF :  尝试修复压缩文件

  > `zip -F`（Normal）：只用中央目录，跳过损坏条目，速度快，可靠性高。
  >
  > `zip -FF`（Full）：全盘扫描本地文件头，尽量抢救所有数据，适合中央目录丢失或严重损坏。
  >
  > 1. **zip -F（普通修复模式）**：
  >    - 原理是假设ZIP文件的中央目录结构基本完整，但可能某些本地文件头或数据损坏
  >    - 它会遍历中央目录中的每个条目，尝试验证对应的本地文件头是否有效
  >    - 如果本地文件头验证失败（通过`validate_local_header`检查），则跳过该条目
  >    - 对于验证通过的条目，直接将文件数据从原ZIP复制到新ZIP（使用`raw_copy_file`）
  >    - 适用于中央目录完整但部分本地文件损坏的场景
  > 2. **zip -FF（全盘修复模式）**：
  >    - 原理是扫描整个ZIP文件原始数据，寻找有效的本地文件头签名(PK\3\4)
  >    - 它会逐个字节扫描文件，找到所有可能的ZIP条目头
  >    - 对于每个找到的本地文件头，读取并解析其结构
  >    - 尝试匹配中央目录中的对应条目（如果有），否则基于本地文件头创建新条目
  >    - 适用于中央目录损坏或丢失，但部分本地文件数据仍可恢复的场景
  >
  > 关键区别：
  >
  > - `-F`依赖中央目录，只修复能通过验证的条目
  > - `-FF`不依赖中央目录，主动扫描整个文件寻找可恢复数据

  先模拟损坏包：
  
  ```bash
  # 破坏中央目录
  dd if=normal.zip of=broken-FF.zip bs=1 count=$(( $(stat -c%s normal.zip) - 100 ))
  # 破坏前面条目数据
  dd if=normal.zip of=broken-F.zip bs=1 skip=100
  ```
  
  
  
  实际执行效果：
  
  ```bash
  [amd@localhost tmp]$ utzip broken-F.zip --out fix-F.zip -F
  Fix archive (-F) - assume mostly intact archive
   copying: ../src/
          zip warning: bad - skipping: ../src/
   copying: ../src/bin/
          zip warning: bad - skipping: ../src/bin/
   copying: ../src/bin/zipcloak.rs
          zip warning: bad - skipping: ../src/bin/zipcloak.rs
   copying: ../src/bin/zipnote.rs
   copying: ../src/bin/zipsplit.rs
   copying: ../src/cli.rs
   copying: ../src/commands/
   copying: ../src/commands/add.rs
   copying: ../src/commands/adjust.rs
  ...
   copying: ../src/utils/progress.rs
   copying: ../src/zip.rs
   copying: ../src/zipcloak.rs
   copying: ../src/zipnote.rs
   copying: ../src/zipsplit.rs
   
  [amd@localhost tmp]$ ../target/debug/utzip broken-FF.zip --out fix-F.zip --FF -v
  Fix archive (-FF) - salvage what can
          zip warning: Missing end (EOCDR) signature - either this archive
                       is not readable or the end is damaged
  Scanning for entries...
   Local ( 1      0): copying: ../src/  (0 bytes)
   Local ( 1     46): copying: ../src/bin/  (0 bytes)
   Local ( 1     96): copying: ../src/bin/zipcloak.rs  (100 bytes)
  ...
   Local ( 1  69628): copying: ../src/zipcloak.rs  (1619 bytes)
   Local ( 1  71304): copying: ../src/zipnote.rs  (2724 bytes)
   Local ( 1  74084): copying: ../src/zipsplit.rs  (3086 bytes)
  Central Directory found...
   Cen   ( 1  77227): updating: ../src/
   Cen   ( 1  77289): updating: ../src/bin/
   Cen   ( 1  77355): updating: ../src/bin/zipcloak.rs
  ...
   Cen   ( 1  79414): updating: ../src/utils/log.rs
   Cen   ( 1  79488): updating: ../src/utils/logfile.rs
   Cen   ( 1  79566): updating: ../src/utils/mod.rs
   Cen   ( 1  79640): updating: ../src/utils/progress.rs
   Cen   ( 1  79719): updating: ../src/zip.rs
   Cen   ( 1  79787): updating: ../src/zipcloak.rs
   Cen   ( 1  79860): 
          zip warning: error reading entry:  Invalid argument
          zip warning: skipping this entry...
  ```



## zipnote 基本命令测试

 Info-ZIP 工具集中的一个实用程序，用于查看和修改 ZIP 存档中的注释（包括文件注释和全局存档注释）.

utzipnote [选项] zip文件

- 查看注释

  ```bash
  utzipnote test.zip
  ```
  
- 修改注释

  ```bash
  zipnote 存档.zip > 注释.txt
  # 编辑注释.txt
  zipnote -w 存档.zip < 注释.txt
  ```

  - 第一步将注释导出到文本文件
  - 编辑文本文件中的注释内容。条目的注释内容需要在符合格式要求。
  - 使用`-w`选项将修改写回ZIP文件
  
  ```bash
  [amd@localhost tmp]$ ../target/debug/utzipnote test2.zip -w < test.txt
  [amd@localhost tmp]$ ../target/debug/utzipnote test2.zip 
  @ ../src/commands/test.rs
  xioalong test1
  @ (comment above this line)
  @ ../src/commands/update.rs
  xioalong test1 asdfag
  @ (comment above this line)
  ...
  @ ../src/utils/log.rs
  @ (comment above this line)
  @ ../src/utils/mod.rs
  @ (comment above this line)
  @ (zip file comment below this line)
  123
  
  ```
  
  



## utzipcloak 基本命令测试

 Info-ZIP 项目中的一个实用工具，主要用于处理 ZIP 文件的加密和解密功能。以下是它的核心功能介绍：

### 主要功能

1. **加密 ZIP 文件**
   
   - 默认行为是对 ZIP 文件中所有未加密的条目进行加密
   - 使用传统 ZIP 加密算法（安全性较弱，不建议用于敏感数据）
   
   ```bash
   [amd@localhost tmp]$ utzipcloak test2.zip 
   Enter passwd: 
   Verify passwd: 
   encrypting: ../src/commands/test.rs
   encrypting: ../src/commands/update.rs
   encrypting: ../src/compression/deflate.rs
   encrypting: ../src/compression/store.rs
   encrypting: ../src/encryption/
   encrypting: ../src/encryption/mod.rs
   encrypting: ../src/encryption/zipcrypt.rs
   encrypting: ../src/error.rs
   encrypting: ../src/lib.rs
   encrypting: ../src/main.rs
   encrypting: ../src/utils/
   encrypting: ../src/utils/common.rs
   encrypting: ../src/utils/log.rs
   encrypting: ../src/utils/mod.rs
   
   ```
   
2. **解密 ZIP 文件**
   
   - 通过 `-d` 参数解密已加密的条目
   - 若密码错误会自动转为复制模式（不修改文件）
   
   ```bash
   [amd@localhost tmp]$ utzipcloak test2.zip -d
   Enter passwd: 
   Verify passwd: 
   decrypting: ../src/commands/test.rs
   decrypting: ../src/commands/update.rs
   decrypting: ../src/compression/deflate.rs
   decrypting: ../src/compression/store.rs
   decrypting: ../src/encryption/
   decrypting: ../src/encryption/mod.rs
   decrypting: ../src/encryption/zipcrypt.rs
   decrypting: ../src/error.rs
   decrypting: ../src/lib.rs
   decrypting: ../src/main.rs
   decrypting: ../src/utils/
   decrypting: ../src/utils/common.rs
   decrypting: ../src/utils/log.rs
   decrypting: ../src/utils/mod.rs
   
   ```
   
   
   
3. **其他特性**
   - 支持临时文件路径指定 (`-b` 参数)
   - 可输出到新文件而非覆盖原文件 (`-O` 参数)

## utzipsplit 基本命令测试

主要用于将一个大型 ZIP 文件分割成多个较小的部分。以下是它的核心功能介绍：

### 参数功能实现

- **-n SIZE**
  控制每个分割文件的最大大小（默认36000字节），通过 `max_size` 参数控制

  ```bash
  utzipsplit tmp/test2.zip -b tmp -n 15000
  5 zip files would be made (100% efficiency)
  creating: tmp/test21.zip
  creating: tmp/test22.zip
  creating: tmp/test23.zip
  creating: tmp/test24.zip
  creating: tmp/test25.zip
  
  
  -rw-r--r-- 1 amd amd 14987  4月27日 16:12 test21.zip
  -rw-r--r-- 1 amd amd 14907  4月27日 16:12 test22.zip
  -rw-r--r-- 1 amd amd 14978  4月27日 16:12 test23.zip
  -rw-r--r-- 1 amd amd 14868  4月27日 16:12 test24.zip
  -rw-r--r-- 1 amd amd 12877  4月27日 16:12 test25.zip
  
  ```

  

- **-i**
  创建索引文件 `zipsplit.idx`，记录分割信息：

  ```bash
  utzipsplit tmp/test2.zip -b tmp -n 15000 -i
  5 zip files would be made (100% efficiency)
  creating: tmp/zipsplit.idx
  creating: tmp/test21.zip
  creating: tmp/test22.zip
  creating: tmp/test23.zip
  creating: tmp/test24.zip
  creating: tmp/test25.zip
  
  cat zipsplit.idx 
  1 Cargo.lock
  1 Cargo.toml
  1 src/
  ...
  5 ../src/commands/add.rs
  3 ../src/compression/
  4 ../src/compression/deflate.rs
  4 ../src/compression/mod.rs
  4 ../src/compression/store.rs
  5 ../src/zipcloak.rs
  5 ../src/error.rs
  ```

  

- **-t**
  测试模式，只计算不实际分割

  ```bash
  utzipsplit tmp/test2.zip -b tmp -t
  3 zip files would be made (100% efficiency)
  
  ```

  

- **-s**
  顺序分割模式（即使会创建更多文件），默认为贪心模式

  ```bash
  utzipsplit tmp/test2.zip -b tmp -n 15000
  5 zip files would be made (100% efficiency)
  creating: tmp/test21.zip
  creating: tmp/test22.zip
  creating: tmp/test23.zip
  creating: tmp/test24.zip
  creating: tmp/test25.zip
  [amd@localhost utzip]$ utzipsplit tmp/test2.zip -b tmp -n 15000 -s
  6 zip files would be made (83% efficiency)
  creating: tmp/test21.zip
  creating: tmp/test22.zip
  creating: tmp/test23.zip
  creating: tmp/test24.zip
  creating: tmp/test25.zip
  ```

- **-p PAUSE**
  分割文件间暂停，等待用户确认

  ```bash
  utzipsplit tmp/test2.zip -b tmp -n 15000 -s -p
  6 zip files would be made (83% efficiency)
  Insert disk #1 of 6 and hit return: 
  creating: tmp/test21.zip
  Insert disk #2 of 6 and hit return: 
  creating: tmp/test22.zip
  Insert disk #3 of 6 and hit return: 
  creating: tmp/test23.zip
  Insert disk #4 of 6 and hit return: 
  creating: tmp/test24.zip
  Insert disk #5 of 6 and hit return: 
  creating: tmp/test25.zip
  Insert disk #6 of 6 and hit return: 
  creating: tmp/test26.zip
  
  ```

- **-r ROOM**
  为首个分割文件预留空间（默认0）

  ```bash
  utzipsplit tmp/test2.zip -b tmp -n 15000 -r 10000 -s
  7 zip files would be made (71% efficiency)
  creating: tmp/test21.zip
  creating: tmp/test22.zip
  creating: tmp/test23.zip
  creating: tmp/test24.zip
  creating: tmp/test25.zip
  creating: tmp/test26.zip
  creating: tmp/test27.zip
  
  -rw-r--r-- 1 amd amd  4320  4月27日 16:42 test21.zip  # 生成的首个文件，大小不超过 15000-10000
  -rw-r--r-- 1 amd amd 12646  4月27日 16:42 test22.zip
  -rw-r--r-- 1 amd amd 14876  4月27日 16:42 test23.zip
  -rw-r--r-- 1 amd amd 11279  4月27日 16:42 test24.zip
  -rw-r--r-- 1 amd amd 11129  4月27日 16:42 test25.zip
  -rw-r--r-- 1 amd amd 12576  4月27日 16:42 test26.zip
  -rw-r--r-- 1 amd amd  5835  4月27日 16:42 test27.zip
  ```


## 备注

**所有的短参数不是唯一字母时，都修改为了长参数。比如 -lf --> --lf ;  -db --> --db ...**

-MM 严格模式，当前版本实现默认就是严格模式

-so 显示可用选项。等价于 -h。效果一致



## 代办项

TODO 打包，集成测试 note和split -- done

TODO 对于大文件压缩较为耗时时，应该先打印添加的文件信息，压缩完成后追加压缩率，然后换行。-- done

TODO 分卷文件的数据读取，分卷不允许更新，只能复制  --out 输出到新的文件。--done

TODO 压缩后大小反而比源文件更大的，需要修改为存储模式 -- done

TODO 压缩级别 -1->-9 ,中间任意数都支持. --done

TODO: FileCompressionTracker 似乎不需要了，直接在run_state 里面维护状态。

TODO: update -FS 时，init信息不对。-- done

TODO: 添加文件输入通配符时， "*.log" 报错，无法添加。-- done

TODO: -FF 修复 --done (修复中央目录损坏，根据本地头重新生成)

TODO: -F 修复 （中央目录完好，本地条目数据损坏，保留有效的条目）-- done

TODO: ZipArchive 不应该加载全部数据字段，应该只记录数据的开始pos，需要时才读取。--done

TODO: utzipcloak 加密后的数据，无法解开 --done

TODO: 验证 -A -J 执行后的数据，无法正常解压，且都是一次性加载所有文件内容，需要修改，避免爆内存。--done

TODO: zip64 超过4GB文件支持。 --done

TODO: 分卷文件的读取，更新。--done



