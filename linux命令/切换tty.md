# 切换tty

近期碰到了，需要在命令中切换 `tty` 的方式，以前一直是通过快捷键 `CTRL+ALT+F1-7` 切换，为此查阅了相关命令，做个记录。

可以使用 `chvt` 命令，用法如下：

```bash
sudo chvt N
```

N 为终端号，如想切换到 `tty2` ，则执行：`sudo chvt 2` 即可

回到桌面，则：`sudo chvt 7`

