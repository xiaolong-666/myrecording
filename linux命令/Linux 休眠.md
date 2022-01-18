# Linux 休眠

## 休眠的类型

### suspend(suspend to RAM)

除内存以外的大部分机器部件都进入断电状态。这种休眠恢复速度特别快，但由于内存中的数据并没有保存下来，还会持续耗电，断电数据会丢失。

### hibernate(suspend to disk)

这种休眠会将内存中的系统状态写入交换空间内，当系统启动时可以从交换空间内读回系统状态。这种情况下系统可以完全断电，但由于要保存/读取系统状态到/从交换空间，因此速度会慢一点。

### hybrid(suspend to both)

结合上面两种休眠类型。一样将系统状态写入交换空间内，同时也想 `suspend` 一样不关闭电源。在电源未耗尽之前，它能很快的从休眠状态恢复。若休眠期间电源耗尽，它可以从交换空间中恢复系统状态。

## 手动调用休眠

在 `systemd` 系统上直接执行 `systemctl suspend` 就可以了。它的实际动作由 `systemd-suspend.service` 所定义。



## 参考链接

https://cloud.tencent.com/developer/article/1170392?from=15425
