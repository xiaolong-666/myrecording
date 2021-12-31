# sudo 免密

编辑 `/etc/sudoers` 文件，找到 `root ALL=(ALL) ALL` 行，在下面添加以下任意一条格式即可：

```bash
youuser            ALL=(ALL)                ALL
%youuser           ALL=(ALL)                ALL
youuser            ALL=(ALL)                NOPASSWD: ALL
%youuser           ALL=(ALL)                NOPASSWD: ALL

第一行：允许用户youuser执行sudo命令(需要输入密码)。
第二行：允许用户组youuser里面的用户执行sudo命令(需要输入密码)。
第三行：允许用户youuser执行sudo命令,并且在执行的时候不输入密码。
第四行：允许用户组youuser里面的用户执行sudo命令,并且在执行的时候不输入密码。
```

强制保存退出：w!