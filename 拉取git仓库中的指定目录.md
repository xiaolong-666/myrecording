# 拉取git仓库中的指定目录
有的项目目录太多，不需要全部拉取，只需要其中一部分，如果clone全部，会浪费很多空间。所以，局部子目录下载，显得很有必要。

## Git拉取子目录
1. 在本地新建一个temp目录，然后执行`git init`实例化项目
2. 然后修改配置`git config core.sparsecheckout true`，开启简介模式
3. 将远程项目目录下，想要下载的文件目录写入到配置文件中 `echo "test/*" >> .git/info/sparse-checkout`
	- 其中 `test/*`目录就是我们想要下载的目录
4. 然后为这个项目添加一个远程仓库`git remote add origin git@***:***.git`
5. 拉取指定分支的文件`git pull origin 分支名`

检查当前目录，已经拉取下来了

## 参考
https://blog.csdn.net/u013948858/article/details/115859862