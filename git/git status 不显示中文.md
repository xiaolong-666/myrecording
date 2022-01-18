# git status 不显示中文

- 现象

使用 `git status` 时，未提交的文件总只显示数字串，显示不出中文文件名，非常不方便

```bash
xiaolong-PC 福 /media/xiaolong/resource/xiaolong-git/myrecording ➤ 96085e8|main⚡
10035 ± : git status                                                                                                     [42d3h39m] ✹ ✭
位于分支 main
您的分支与上游分支 'origin/main' 一致。

尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git checkout -- <文件>..." 丢弃工作区的改动）

        修改：     "git\347\233\270\345\205\263.md"
        修改：     "python/Jenkins\344\275\277\347\224\250Pytest \345\222\214 Allure.md"
```

- 原因

在默认设置下，中文文件名在工作区状态输出，中文名不能正确显示，而是显示为八进制的字符编码。

- 解决方案

将 `git` 配置文件 `core.quotepath` 项设置为 `false` ， `quotepath` 表示引用路径

```bash
xiaolong-PC 福 /media/xiaolong/resource/xiaolong-git/myrecording ➤ 96085e8|main⚡
10036 ± : git config --global core.quotepath false  
```

效果如下：

```bash
xiaolong-PC 福 /media/xiaolong/resource/xiaolong-git/myrecording ➤ 96085e8|main⚡
10037 ± : git status                                                                                                     [42d3h40m] ✹ ✭
位于分支 main
您的分支与上游分支 'origin/main' 一致。

尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git checkout -- <文件>..." 丢弃工作区的改动）

        修改：     git相关.md
        修改：     python/Jenkins使用Pytest 和 Allure.md
```



## 参考链接

https://blog.csdn.net/u012145252/article/details/81775362