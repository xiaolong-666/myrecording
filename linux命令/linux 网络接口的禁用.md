# linux 网络接口的禁用

## 背景

在某些场景下，需要支持从底层禁用网络功能。为实现该功能，可以采用`udev` 规则来解除绑定的方式。

`udev` 现在是 `systemd` 的一部分，默认已经安装，故只要是 `systemd` 启动的系统，都可采用该方式。

## 规则

`udev` 规则以管理员身份编写，并保存在 `/etc/udev/rules.d` 目录，其文件名必须以 `.rules` 结尾。各种软件包提供的规则文件位于 `/lib/udev/rules.d/` 。如果两个目录下，有同名文件，则 `/etc` 中的文件优先。

## 编写网络禁用的规则

因不同机型的网络接口名不一致，故写了一个脚本，遍历 `/sys/class/net/` 目录下的所有网络接口，调用其接口的 `unbind` 属性即可，解除网络接口的绑定，从而达到从底层禁用网络的目的。

```bash
netdownrulesfile=/etc/udev/rules.d/90_disable_net.rules
for interface_path in /sys/class/net/* ;
do
    interface_name=$(basename ${interface_path})
    if [ -d ${interface_path}/device/driver ]; then
        for device_item in ${interface_path}/device/driver/* ;
        do
            item_name=$(basename ${device_item})
            if [ -d "${device_item}" ] && [ -L "${device_item}" ] && [ "${item_name}" != "module" ]; then
                cat >> "${netdownrulesfile}" <<EOF
ACTION=="add", SUBSYSTEM=="net", ENV{ID_NET_NAME_PATH}=="${interface_name}",RUN+="/bin/bash -c 'echo ${item_name} > ${interface_path}/device/driver/unbind'"
EOF
            fi
        done
    fi
done

```

