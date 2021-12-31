# 编写systemd服务

本目录下，编写了一个守护脚本，定时监测系统资源。通过编写对应的service文件，使用systemd管理  
实现步骤：
1. 编写service文件，将其放入/lib/systemd/system/目录下即可。service文件各参数含义见
[参考链接](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)

2. 使用`systemctl enable name.service`即可开机启动该服务

