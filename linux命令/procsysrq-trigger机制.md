# /proc/sysrq-trigger机制

近期了解到，linux 系统中，存在一些 `魔法组合键` ，也就是 `Sysrq` 。是内建于 linux 内核的调试工具。

首先，内核配置选项中要使能 `CONFIG_MAGIC_SYSRQ` 选项，这样系统启动后，会生成 `/proc/sysrq-trigger` 节点用于调试。

其次，可以在/etc/sysctl.conf中设置kernel.sysrq=1默认使能sysq功能。也可以通过写/proc/sys/kernel/sysrq节点动态使能sysrq功能。写入不同的值使能不同的功能：

```
0 - disable sysrq completely
1 - enable all functions of sysrq
...
```

更多请搜索：内核帮助文档 `kernel/Documentation/sysrq.txt`



## 使用方式

### 重启

`echo 'b' > /proc/sysrq-trigger`

### 关机

`echo 'o' > /proc/sysrq-trigger`

### 故意让系统崩溃

`echo 'c' > /proc/sysrq-trigger`



## 参考链接

https://www.cnblogs.com/klb561/p/11013746.html