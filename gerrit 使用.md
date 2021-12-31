# gerrit使用

# 提交模板

在 ~/ 目录下新建文件，并命名为`.gitcommit_template`

将以下内容写入文件当中：
```
# commit type :fix（问题修复）、feat（功能开发）、build（构建修改）、docs（文档）、chore（其他) + 简单描述. 默认fix,根据情况修改
fix: 

# Describe you bug/feature,or other things
Description: 

# 
Log: 

# 关联pms上的bug号，提交后，则会自动在pms对应bug信息页面添加备注，关联本次提交。若本次提交为修复bug相关，则请取消注释
#Bug: 

# 尚不清楚干啥的 
#Issue: 

# 关联pms上的任务号，提交后，则会自动在pms对应任务信息页面添加备注，关联本次提交。若本次提交为任务相关，则请取消注释
#Task: 

```
在命令行执行:
```bash
git config —global commit.template ~/.gitcommit_template
```

## 安装

```bash
sudo apt install git-review
```

## 下载对应仓库

1. 登录`gerrit.uniontech.com`，使用ldap账户登录
2. 找到自己的仓库地址，比如定制工具`https://gerrit.uniontech.com/admin/repos/isotailor`
3. 复制带有commit-msg的仓库地址：

4. 打开终端，在指定的目录，执行上述复制的内容即可

## 推送

`git review branch(当前分支) -r origin`

若不想每次提交都加上`-r`选项，执行以下命令：

```bash
git config --global gitreview.remote origin 
```

提交代码，直接使用 `git review branch<目标分支>`，例如`git review master`
