# go.mod 文件

`Go.mod` 是 `Golang1.11` 版本新引入的官方包管理工具，用于记录依赖包具体版本，方便依赖包的管理。

`Go.mod` 其实是一个 `Modules`

`Modules` 和传统的 `GOPATH` 不同，不需要包含例如 `src`，`bin` 这样的子目录，一个源代码目录甚至是空目录都可以作为 `Modules`，只要其中包含有`go.mod`文件。



## 基本命令

`go.mod` 提供了 `module、require、replace和exclude` 四个命令

module语句指定包的名字（路径）

require语句指定的依赖项模块

replace语句可以替换依赖项模块

exclude语句可以忽略依赖项模块

## 初始化

在当前目录下，命令行运行 `go mod init 模块名称`

```go
go mod init hello
```

运行完成后，会在当前目录下生成一个 `go.mod` 文件，内容如下：

```go
module hello

go 1.18
```

接着执行 `go mod tidy`，更新 `go.mod` 文件，并自动下载依赖，更新后如下

```
module hello

go 1.18

require github.com/gin-gonic/gin v1.9.0

require (
	github.com/bytedance/sonic v1.8.0 // indirect
	...
	gopkg.in/yaml.v3 v3.0.1 // indirect
)

```



## 参考链接

https://zhuanlan.zhihu.com/p/126561786

