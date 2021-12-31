# Cockpit 基于web的Linux服务器管理工具

## 介绍

**Cockpit** 是一个免费且开源的基于 **web** 的 **Linux** 服务器管理工具。并且在 **CentOS 8** 和 **RHEL 8** 中，**Cockpit** 更是成为其默认服务器管理工具。
通过 **Cockpit** 提供的友好的 **Web** 前端界面可以轻松地监视和管理我们的 **GNU**/**Linux** 服务器，执行诸如存储管理、网络配置、检查日志、虚拟机管理等任务。

## 功能特点

- **Cockpit** 使用 **systemd** 完成从运行守护进程到服务几乎所有的功能
- 集中式管理，通过一个会话窗口管理网络中的所有 **Linux** 服务器
- 创建和管理 **KVM**、**oVirt** 虚拟机
- 包括 **LVM** 在内的存储配置
- 基本的网络配置管理
- 用户 **user account** 管理
- 基于 **web** 的终端
- 图形化的系统性能展示
- 使用 **sosreport** 收集系统配置和诊断信息

## 安装使用

```bash
sudo apt install cockpit
```

安装完成后，在浏览器器中，输入：`http://ip:9090` 即可到登陆界面



tips: 如果 9090 端口被占用，导致服务失败，则可以手动修改 `/usr/lib/systemd/system/cockpit.socket` 中的端口号。

```bash
root@sys-tools-amd64:~# cat /usr/lib/systemd/system/cockpit.socket
[Unit]
Description=Cockpit Web Service Socket
Documentation=man:cockpit-ws(8)
Wants=cockpit-motd.service

[Socket]
ListenStream=8090	# 此处我修改为8090
ExecStartPost=-/usr/share/cockpit/motd/update-motd '' localhost
ExecStartPost=-/bin/ln -snf active.motd /run/cockpit/motd
ExecStopPost=-/bin/ln -snf /usr/share/cockpit/motd/inactive.motd /run/cockpit/motd

[Install]
WantedBy=sockets.target

```

登陆用户为系统的用户和密码，后续操作参考链接

## 参考链接

[cockpit介绍](https://zhongguo.eskere.club/%E5%BC%80%E5%A7%8B%E4%BD%BF%E7%94%A8-cockpit%EF%BC%8C%E4%B8%80%E4%B8%AA%E5%9F%BA%E4%BA%8E-web-%E7%9A%84-linux-%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AE%A1%E7%90%86%E5%B7%A5%E2%80%8B%E2%80%8B%E5%85%B7/2021-07-16/)