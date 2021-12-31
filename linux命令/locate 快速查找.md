# locate 快速查找

## 简介

`locate` 让使用者可以很快速的查找系统内是否有指定的文件。其原理是先建立一个包括系统内所有文件的名称及路径的数据库，之后当查找时就只需要查询数据库即可，该数据库一般每日更新。

## 安装及使用

执行以下命令即可安装

```bash
sudo apt install locate
```

安装完成后，需要手动更新一下数据库

```bash
sudo updatedb
```

接着就可以使用 `locate file_name` 查找了，如下。

```
$ locate wpa_supplicant.service
/data/var/lib/systemd/deb-systemd-helper-enabled/multi-user.target.wants/wpa_supplicant.service
/data/var/lib/systemd/deb-systemd-helper-enabled/wpa_supplicant.service.dsh-also
/etc/systemd/system/multi-user.target.wants/wpa_supplicant.service
/usr/lib/systemd/system/wpa_supplicant.service
/usr/share/deepin-ab-recovery/hospice/systemd/deb-systemd-helper-enabled/multi-user.target.wants/wpa_supplicant.service
/usr/share/deepin-ab-recovery/hospice/systemd/deb-systemd-helper-enabled/wpa_supplicant.service.dsh-also
/var/lib/systemd/deb-systemd-helper-enabled/multi-user.target.wants/wpa_supplicant.service
/var/lib/systemd/deb-systemd-helper-enabled/wpa_supplicant.service.dsh-also
```

## 参考资料

[man locate](https://linux.die.net/man/1/locate)



