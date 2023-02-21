# Kickstart 安装器

`Kickstart` 安装提供了一种部分或全部自动化安装过程的方法。`Kickstart` 文件包含安装程序通常提出的所有问题的答案，例如您希望系统使用哪个时区、驱动器应如何分区或应安装哪些软件包。因此，在安装开始时提供准备好的 `Kickstart` 文件允许您自动执行安装，而无需用户的任何干预。这在同时在大量系统上部署 `Fedora` 时特别有用

所有 `Kickstart` 脚本及其执行的日志文件都存储在该 `/tmp` 目录中，以帮助调试安装问题。



## 如何执行 `Kickstart` 安装

- 创建一个 `Kickstart` 文件
- 创建启动媒体
- 使 `Kickstart` 文件在可移动媒体、硬盘驱动器或网络位置上可用
- 通过引导安装程序并使用引导选项，告诉安装程序在哪里找到 `Kickstart` 文件，并开始 `Kictstart` 安装

## 创建 `Kickstart` 文件

[Kickstart 文件本身是一个纯文本文件，包含Kickstart 语法参考](https://docs.fedoraproject.org/en-US/fedora/rawhide/install-guide/appendixes/Kickstart_Syntax_Reference/#appe-kickstart-syntax-reference) 中列出的关键字，用作安装说明。任何能够将文件保存为 ASCII 文本的文本编辑器都可用于创建和编辑 `Kickstart` 文件。

创建 `Kickstart `文件时，请记住以下几点：

- 以井号 ( ) 开头的行`#`被视为注释并被忽略。
- 部分必须按**顺序**指定。除非另有说明，否则这些部分中的项目不必按特定顺序排列。正确的部分顺序是：
  - 命令部分包含 [Kickstart 语法参考 ](https://docs.fedoraproject.org/en-US/fedora/rawhide/install-guide/appendixes/Kickstart_Syntax_Reference/#appe-kickstart-syntax-reference)中列出的实际 `Kickstart` 命令和选项。请注意，某些命令（例如install）是必需的，但大多数命令是可选的。
  - %packages部分包含要安装的包和包组的列表。有关详细信息，请参阅[%packages (required) - Package Selection](https://docs.fedoraproject.org/en-US/fedora/rawhide/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-packages)。
  - %pre和%post部分，包含安装前和安装后脚本。这两个部分可以按任何顺序排列，不是强制性的。有关详细信息，请参阅[%pre（可选）- 安装前脚本](https://docs.fedoraproject.org/en-US/fedora/rawhide/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-preinstall)和[%post（可选）- 安装后脚本](https://docs.fedoraproject.org/en-US/fedora/rawhide/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-postinstall)。

## 验证 `Kickstart` 文件

安装此软件包，请执行以下命令：

```
# dnf install pykickstart
```

安装软件包后，您可以使用以下命令验证 `Kickstart` 文件：

```
$ ksvalidator /path/to/kickstart.ks
```

将 */path/to/kickstart.ks* 替换为您要验证的 `Kickstart` 文件的路径。

## 参考链接

https://docs.fedoraproject.org/en-US/fedora/rawhide/install-guide/advanced/Kickstart_Installations/#chap-kickstart-installations