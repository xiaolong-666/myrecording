# Git hook钩子使用
和其它版本控制系统一样，Git 能在特定的重要动作发生时触发自定义脚本。 有两组这样的钩子：客户端的和服务器端的。 客户端钩子由诸如提交和合并这样的操作所调用，而服务器端钩子作用于诸如接收被推送的提交这样的联网操作。 你可以随心所欲地运用这些钩子。

## 安装钩子
钩子都被存储在 Git 目录下的 hooks 子目录中。 也即绝大部分项目中的 .git/hooks 。 当你用 git init 初始化一个新版本库时，Git 默认会在这个目录中放置一些示例脚本，如下所示：
```bash
> ls .git/hooks 
applypatch-msg.sample  fsmonitor-watchman.sample  pre-commit.sample          pre-rebase.sample
commit-msg             post-update.sample         prepare-commit-msg.sample  pre-receive.sample
commit-msg.sample      pre-applypatch.sample      pre-push.sample            update.sample
```
这些示例的名字都是以 .sample 结尾，如果你想启用它们，得先移除这个后缀。
把一个正确命名（不带扩展名）且可执行的文件放入 .git 目录下的 hooks 子目录中，即可激活该钩子脚本。 这样一来，它就能被 Git 调用。
**需要注意的是，克隆某个版本库时，它的客户端钩子并不随同复制 ！！！**

## 客户端钩子
客户端钩子分为很多种。 下面把它们分为：提交工作流钩子、电子邮件工作流钩子和其它钩子
本文只介绍提交工作流钩子，更多细节请参考文末链接。

### 提交工作流钩子
`pre-commit` 钩子在键入提交信息前运行。 它用于检查即将提交的快照，例如，检查是否有所遗漏，确保测试运行，以及核查代码。 如果该钩子以非零值退出，Git 将放弃此次提交，不过你可以用 `git commit --no-verify` 来绕过这个环节。
你可以利用该钩子，来检查代码风格是否一致（运行类似 lint 的程序）、尾随空白字符是否存在（自带的钩子就是这么做的），或新方法的文档是否适当。

**note: 此钩子经常在项目中采用，进行本地格式检查，当格式符合要求后，才允许提交！**

`prepare-commit-msg` 钩子在启动提交信息编辑器之前，默认信息被创建之后运行。 它允许你编辑提交者所看到的默认信息。
`commit-msg` 钩子接收一个参数，此参数即上文提到的，存有当前提交信息的临时文件的路径。 如果该钩子脚本以非零值退出，Git 将放弃提交，因此，可以用来在提交通过前验证项目状态或提交信息。
**该钩子经常用来检查commit的提交信息**

`post-commit` 钩子在整个提交过程完成后运行，该钩子一般用于通知之类的事情。

## 参考资料
[Git-Git钩子](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)
