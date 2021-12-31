# 关于git的日常使用

## 日常使用
1. 添加文件  
>添加到Git仓库，分两步：  
使用命令git add <file>，注意，可反复多次使用，添加多个文件；  
使用命令git commit -m <message>，完成。

2. 要随时掌握工作区的状态，使用git status命令。  
如果git status告诉你有文件被修改过，用git diff可以查看修改内容。

3. 版本回退  
>HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令git reset --hard commit_id。  
穿梭前，用git log可以查看提交历史，以便确定要回退到哪个版本。  
要重返未来，用git reflog查看命令历史，以便确定要回到未来的哪个版本。  

4. 提交后，用git diff HEAD — <file>命令可以查看工作区和版本库里面最新版本的区别：

5. 命令git checkout -- readme.txt
>意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：  
一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；  
一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。  
总之，就是让这个文件回到最近一次git commit或git add时的状态。

6. 用命令git reset HEAD <file>可以把暂存区的修改撤销掉（unstage），重新放回工作区

7. git checkout — <file>其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

8. 要关联一个远程库，使用命令  
git remote add origin git@server-name:path/repo-name.git；  
关联后，使用命令git push -u origin master第一次推送master分支的所有内容；  
此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改；  

9. Git鼓励大量使用分支：  
>查看分支：git branch  
创建分支：git branch <name>  
切换分支：git checkout <name>或者git switch <name>  
创建+切换分支：git checkout -b <name>或者git switch -c <name>  
合并某分支到当前分支：git merge <name>  
删除分支：git branch -d <name>  

10. 当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug  
修复后，再git stash pop，回到工作现场；



## 打path
1. 使用git format-patch生成所需要的patch:  
>当前分支所有超前master的提交：  
git format-patch -M master  
某次提交以后的所有patch:  
git format-patch 4e16 --4e16指的是commit名  
从根到指定提交的所有patch:  
git format-patch --root 4e16  
某两次提交之间的所有patch:  
git format-patch 365a..4e16 --365a和4e16分别对应两次提交的名称  
某次提交（含）之前的几次提交：  
git format-patch –n 07fe --n指patch数，07fe对应提交的名称  
故，单次提交即为：  
git format-patch -1 07fe

`git format-patch HEAD^ `　　　　　　　　　　#生成最近的1次commit的patch  
`git format-patch HEAD^^`　　　　　　　　　　#生成最近的2次commit的patch

>git format-patch生成的补丁文件默认从1开始顺序编号，并使用对应提交信息中的第一行作为文
件名。如果使用了-- numbered-files选项，则文件名只有编号，不包含提交信息；
如果指定了--stdout选项，可指定输出位置，如当所有patch输出到一个文件；可指
定-o <dir>指定patch的存放目录；


2. 应用patch：
    先检查patch文件：git apply --stat newpatch.patch  
    检查能否应用成功：git apply --check newpatch.patch  
    打补丁：git am --signoff  newpatch.patch  

如果使用`am`打补丁失败，提示有冲突，调用：`git apply --reject newpatch.path`。将能够打上的全部打上，不能的生成 `.rej`文件，查看冲突部分，自己手动解决，然后删除`.rej`文件，将修改后的手动`add`，执行`git am --resolved`，即可成功应用。

## 变基操作(git rebase)  
>1. git checkout dev2.0		#切换到需要合并的分支
2. git rebase master	        #变基目的分支
3. git checkout master		# 切换到目的分支
4. git merge dev2.0		       #合并dev2.0上所有的信息到master上，即变基

[参考链接](https://git-scm.com/book/zh/v2/Git-分支-变基)

## 拉取一个仓库代码，上传到另一个仓库

参考链接：[git基础-远程仓库的使用](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%E7%9A%84%E4%BD%BF%E7%94%A8)
1. 先拉取代码，以gerrit上的dde为例
```bash
git clone "ssh://******@gerrit.******.com:29418/dde" && scp -p -P 29418 ******@gerrit.******.com:hooks/commit-msg "dde/.git/hooks/"
```
2. 新建分支, 并切换
```bash
git checkout -b chengdu
```
3. 查看远程仓库地址
```bash
dde [chengdu●] git remote -v
origin  ssh://******@gerrit.******.com:29418/dde (fetch)
origin  ssh://******@gerrit.******.com:29418/dde (push)
```
4. 增加远程仓库地址，并取名
```bash
dde [chengdu●] git remote add cd git@******.******.com:deviceos/dde.git
dde [chengdu●] git remote -v
cd      git@******.******.com:deviceos/dde.git (fetch)
cd      git@******.******.com:deviceos/dde.git (push)
origin  ssh://******@gerrit.******.com:29418/dde (fetch)
origin  ssh://******@gerrit.******.com:29418/dde (push)

```
5. 推送的新的仓库里面
```bash
dde [chengdu] git push cd chengdu 
枚举对象: 73, 完成.
对象计数中: 100% (73/73), 完成.
使用 16 个线程进行压缩
压缩对象中: 100% (28/28), 完成.
写入对象中: 100% (73/73), 11.48 KiB | 11.48 MiB/s, 完成.
总共 73 （差异 32），复用 61 （差异 26）
To ******.******.com:deviceos/dde.git
 * [new branch]      chengdu -> chengdu

```

## git pull时产生'Merge branch 'master' of...问题
在进行 pull 操作的同时，其实就是 fetch+merge 的一个过程。我们从 remote 分支中拉取新的更新，然后再合并到本地分支中去。

- 在执行`git pull`的时候加上`--rebase`参数。这参数的意思就是在合并代码之前，先执行变基操作，成功后在进行真正的merge操作。(如果有冲突需要手动解决)
- 在你的git bash里执行`git config --global pull.rebase true`。这个配置就是告诉git在每次pull前先进行rebase操作。这种方法和方法1原理一样，只不过方法1是每次pull前都要手动操作。

[参考资料](https://blog.csdn.net/xiaojie_570/article/details/96486325)