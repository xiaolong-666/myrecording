# lightdm问题排查

问题：当在登陆界面输入密码后，黑屏进不去桌面环境，等待几秒后又回到登陆界面



问题定位：一般对于lightdm相关的问题，产生的日志信息，存放于用户家目录`.xsession-errors`文件中，比如此处失败是因为缺少相应的launcher文件导致

```bash
test@test-PC:~$ cat .xsession-errors | grep com.deepin.dde.launcher
(process:3389): GLib-GIO-ERROR **: 09:03:40.537: Settings schema 'com.deepin.dde.launcher' is not installed
```





