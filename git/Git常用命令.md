# Git常用命令

[Pro Git 中文版](https://gitee.com/progit/index.html)

## 初始化仓库

初始化当前工作目录

```bash
git init
```

从已有的GitHub仓库克隆

```bash
git clone <url>
```

## 忽略文件

编辑`.gitignore`文件

```bash
# 忽略类型为a的文件
*.a
# 但是lib.a除外
!lib.a
# 忽略build目录下的所有文件
build/
# 忽略doc目录下的所有txt文件，但是不包括子目录下的
doc/*.txt
```

## 基础命令

查看当前状态

```bash
git status
```

跟踪文件到暂存区

```bash
git add <file_name>
```

`add .`和`add *`的区别：`add *`会忽略.gitignore而跟踪所有文件

查看变更内容

```bash
git diff #尚未暂存的文件和已经暂存的文件
git diff --staged #上次提交的快照和已经暂存的文件
git diff <pointer> #某次提交的快照和已经暂存的文件
```

提交更新

```bash
git commit #启动文本编辑器输入本次提交说明
git commit -m "annotation" #简短的说明
git commit -a #自动暂存已经被追踪的文件，从而跳过使用git add
git commit --amend #修改上次提交的说明
```

**HEAD指向最近的一次提交**

移除文件

```bash
git rm <file_name> #移除文件
git rm --cached <file_name> #停止跟踪文件但不删除
```

查看提交历史

```bash
git log #当前状态为终点的历史
git reflog #操作历史
```

回溯修改

```bash
git reset --hard <pointer> #将当前仓库的HEAD,暂存区和工作树回溯到指定状态
```

## 分支

创建分支

```bash
git branch <branch_name>
```

删除分支

```bash
git branch -d <branch_name>
```

更换分支

```bash
git checkout <branch_name>
git checkout -b <branch_name> #创建并更换分支
```

合并分支

```bash
git merge <branch_name> #合并当前分支和指定分支
```

衍合分支

```bash
git rebase <branch_name>
```

## 远程

添加远程仓库

```bash
git remote add origin <url>
```

推送到远程仓库

```bash
git push -u origin <branch_name>
```

获取远程仓库更新

```bash
git pull origin <branch_name>
```

获取远程的分支

```bash
$ git checkout -b <branch_name> origin/<branch_name>
```