# Fcitx 虚拟键盘的关闭开启

某些场景下，需要关闭 `fcitx` 的虚拟键盘功能。默认是开启该功能，关闭时按照以下方式。

## 方式一、界面勾选方式

在输入法的配置界面，附加组件选项-->选中高级-->取消底部的虚拟键盘组件--> 重启输入法。

## 方式二、配置文件方式

在用户的家目录下的 `.config/fcitx/addon` 中编写配置，写入以下信息，重启输入法。

```bash
xiaolong@xiaolong-PC:~$ cat $HOME/.config/fcitx/addon/fcitx-vk.conf
[Addon]
Enabled=False
```