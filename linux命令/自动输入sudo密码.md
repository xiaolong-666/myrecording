# 自动输入sudo密码

`SUDO_ASKPASS` 环境变量可以指定一个程序或者脚本的路径，获取其的标准输出。当 `sudo` 需要密码验证时，它就会调用这个程序（可做成GUI也可直接脚本返回密码）来获取密码，而不是直接在终端中显示密码交互输入。



导出环境变量：

```bash
export SUDO_ASKPASS=/path/to/my_askpass_program
```

使用sudo提前命令：

```bash
sudo -A command
```

这样，就不用在手动输入密码啦，特别用于自动化脚本中处理。