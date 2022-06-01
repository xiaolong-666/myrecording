# Qt以root用户运行调试

在 `Linux` 系统下开发，避免不了涉及到权限问题，不建议直接使用 `root` 用户执行，但有时开发过程中，调试定位又必须涉及到。为此，查阅资料，找到了几种方式：

## 1. 直接sudo提权打开qtcreator

该方式会导致部分环境变量没有切换，无法正常使用。

## 2. 切换用户后，打开qtcreator

首先切换用户，使用 `su -` 或者 `sudo -i` ; 然后找到 `qtcreator` 执行即可。

**这种方式会把打开的项目文件所有权修改为 `root` 用户**

## 3. 或者修改配置，在最末尾添加 sudo

修改配置
**【Tool】➜ 【Options】➜ 【Environment】➜ 【System】➜ 【Terminal】**
然后在最末尾 `-e` 加上`sudo`, 接着在运行配置时，勾选在 `Run in terminal` 

该方式无法进行 `debug` 调试，可以直接执行。



## 参考链接

https://cxywk.com/ubuntu/q/dbK4Xjnd