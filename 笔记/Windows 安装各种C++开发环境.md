# Windows 安装各种C++开发环境

使用 `vcpkg` ，它是一个 C++ 的包管理工具，它的出现就是为了解决C++项目依赖在不同系统不同开发工具相互不兼容，不通用的问题。



- 自动下载开源库源代码，从 github 下载官方源码
- 源码包的缓存管理和版本升级
- 一键轻松编译，相关的编译过程 vcpkg 都使用 powershell 脚本写好了
- 依赖关系检查（类似 apt build-deb ./）

## 安装

```bash
git clone https://github.com/microsoft/vcpkg
.\vcpkg\bootstrap-vcpkg.bat(windows)
```