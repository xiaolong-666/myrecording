# fcitx 配置禁用虚拟键盘

全局配置文件目录：`/usr/share/fcitx/`

用户配置目录：`~/.conf/fcitx/`



禁用组件（虚拟键盘），其它组件类似。组件的配置文件存放与全局目录中`/usr/share/fcitx/addon`

1. 通过输入法配置界面，高级选项取消选中虚拟键盘
2. 自己在`~/.conf/fcitx/addon/`目录中，编写禁用虚拟键盘组件配置`fcitx-vk.conf`

```bash
test@test-pc:~/.config/fcitx/addon$ cat fcitx-vk.conf
[Addon]
Enabled=False
```

