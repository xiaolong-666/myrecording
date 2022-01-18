# 清除git所有历史记录

- 创建新分支（--orphan)，没有任何提交记录的分支

```bash
9:46 xiaolong@xiaolong-PC /media/xiaolong/resource/xiaolong-git/myrecording
% git checkout --orphan new_branch        
切换到一个新分支 'new_branch'
```

- 添加所有文件

```bash
git add ./
```

- 提交当前所有文件

```bash
git commit -m "提交说明"
```

- 删除原来的主分支

```bash
git branch -D main
```

- 把当前分支重命名为main

```bash
git branch -m main
```

- 最后把代码推送到远程仓库

```bash
git push -f origin main
```

