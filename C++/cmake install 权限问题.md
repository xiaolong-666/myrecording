# cmake install 权限问题

近期安装脚本时，发现目录下的脚本安装到指定目录后，权限均为 `644` ，导致脚本没有可执行权限。

查阅资料可知，`install` 有很多参数，其中就涉及到了权限的参数：`FILE_PERMISSIONS` 和 `DIRECTORY_PERMISSIONS` 选项指定对目标中文件和目录的权限。如果指定了 `USE_SOURCE_PERMISSIONS` 而未指定 `FILE_PERMISSIONS`，则将从源目录结构中复制文件权限。如果未指定权限，则将为文件提供在命令的 `FILES` 形式中指定的默认权限(644权限)，而目录将被赋予在命令的 `PROGRAMS` 形式中指定的默认权限(755权限)。

参考链接：https://blog.csdn.net/qq_38410730/article/details/102837401