# runuser命令使用

`runuser` 命令使用一个替代的用户或组运行一个 `shell` 。这个命令仅在 `root` 用户时可以使用。

核心选项：

>
>
>-l: 让shell成为登陆shell
>
>-c：切换用户后执行的命令

主要用途：

可以在root用户下，执行某些普通用户权限下的命令，使用方式：

```bash
runuser -l xiaolong -c 'ls'
```

列出的结果为指定用户的结果。