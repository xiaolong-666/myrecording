# 撤销 git commit --amend

最近，使用 `git commit --amend` 命令时，没注意查看提交记录，将错误内容 `amend` 了到本次提交上。不想使用 `git reset HEAD^` 撤销本次提交，因为这会将当前的工作区改得面目全非，为此记录一种新的方式。

如果只 `amend` 了一次，那么直接使用 `git reset HEAD@{1}` 就可以撤销本次 `amend` 。如果使用了多次，那么就参考 `git reflog` 进行撤销。

首先执行 `git reflog` 命令查看操作记录：

```bash
% git reflog     
03ff845 (HEAD -> cmd) HEAD@{1}: commit: fix: 修复问题
...
```

可以在其中，找到任意的一次提交值，执行 `git reset HEAD@{1}` 中间的数字对应某次的提交。