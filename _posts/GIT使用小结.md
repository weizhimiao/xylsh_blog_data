---
title: GIT使用小结
date: 2016-09-21 23:30:00
ctime: 2016-09-21 23:30:00
utime: 2016-09-23 23:30:00
modif_times: 2
tags:
- git
- 版本控制
categories:
- 软件工程
---

## 概念
在使用之前先明确两个概念。
- 工作区（working directory）
> 我们创建的文件夹

- 版本库（Repository）
> 一个工作区中隐藏的目录（.git）这个目录不算工作区
版本库
>   - stage，暂存区
>   - master，分支
>
> 日常我们进行git add操作，是将文件修改添加到了暂存区，而进行git commit操作，则是将暂存区中的修改提交至当前分支。

<!-- more -->

## 常用操作
1. 创建项目文件夹
```bash
mkdir demo
```
2. 进入项目目录
```
cd demo
git init（将该目录变成git可以管理的仓库（repository））
```
初始化后，该目录下会产生一个.git 的隐藏文件夹。

3. 添加文件到仓库
```
git add 文件
```
添加一个文件到仓库。
其实，该操作作用是将文件添加至Stage暂存区。

  常用操作
```bash
git add .   #将所有文件添加至 stage
git add -u  #将所有文件添加至 stage ，同时将工作区中删除的文件也从仓库中删除。
```
4. git commit  提交到版本库
```
git commit -m "write readme file"
```
-m 为对本次版本提交的说明

5. git status 查看当前版本库状态
```bash
$ mkdir git_test
$ cd git_test
$ git init
# Initialized empty Git repository in /Users/zhimiao/WWW/hexo/git_test/.git/
$ ll -a
# total 0
# drwxr-xr-x   3 zhimiao  staff   102B  9 23 10:25 .
# drwxr-xr-x   6 zhimiao  staff   204B  9 23 10:25 ..
# drwxr-xr-x  10 zhimiao  staff   340B  9 23 10:25 .git
$ git status
# On branch master
# Initial commit
# nothing to commit (create/copy files and use "git add" to track)
```
  - git将所有的文件分为3类：已追踪的，被忽略的，和未追踪的。
    - 已追踪的（tracked）
 > 已追踪的文件是指已经在版本库中的文件，或者是已暂存到索引中的文件。即，一个文件通过执行git add，就会被添加到暂存区，该文件就变成一个已追踪的文件。
    - 被忽略的（ignored）
  > 被忽略的文件是指在版本库中被明确声明为不可见或者被忽略。一般通过在工作区新建 `.gitignore` 文件来来声明。
  > ```
> $ cat .gitignore
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
> ```
    - 未追踪的（untracked）
  > 未追踪的文件是指那些不在前两类中的文件。git把工作目录下的所有文件当成一个集合，减去已经追踪的文件和忽略的文件，剩下的部分就作为未追踪的文件。

  ```
$ echo "git test" > readme.md
$ git status
# On branch master
# Initial commit
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
# 	readme.md
# nothing added to commit but untracked files present (use "git add" to track)
$ git add readme.md
$ git status
# On branch master
# Initial commit
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
# 	new file:   readme.md
$ git commit -m "add readme file"
# [master (root-commit) 81c90a0] add readme file
#  1 file changed, 1 insertion(+)
#  create mode 100644 readme.md
$ git status
#  On branch master
#  nothing to commit, working directory clean
```

6. git diff 显示当前尚未缓存的改动记录
```
$ echo "Second Line " >> readme.md
$ git diff
# diff --git a/readme.md b/readme.md
# index f6edd6e..a1e649c 100644
# --- a/readme.md
# +++ b/readme.md
# @@ -1 +1,2 @@
#  git test
# +add new Line
```
在开头，原始文件被『--』符号标记起来，新文件被用『+++』标记。@@ 之间表示两个不同文件版本的上下文行，以减号（-）开始的行表示从原始文件删除该行以得到新文件。相反，以加号（+）开始的行表示从原始文件中添加该行以产生新文件，而以空格开始的行则表示两个版本都有的行。

7. git log 记录每次commit的信息
```
$ git add readme.md
$ git commit -m "update"
$ echo "third Lines;" >> readme.md
$ git add readme.md
$ git commit -m "update third"
$ git log
# commit 2ba4ebfe3ccef91f0d34646f5a0e50e339ee6f7a
# Author: weizhimiao <532615323@qq.com>
# Date:   Fri Sep 23 11:21:24 2016 +0800
#
#     update third
#
# commit ed0fe412477c8ac828e450cbc319c06ac8db0445
# Author: weizhimiao <532615323@qq.com>
# Date:   Fri Sep 23 11:19:34 2016 +0800
#
#     update
#
# commit 81c90a01f972ef803bd16272e7983aa1b3e9fa9c
# Author: weizhimiao <532615323@qq.com>
# Date:   Fri Sep 23 10:42:58 2016 +0800
#
#     add readme file
```
8. git reset 修改命令
```
git reset HEAD    废除本次修改，回到上次提交的状态
git reset -hard [commit id]
```
```
$ git reset --hard HEAD^
# HEAD is now at 7563423 update
$ git log   
# commit ed0fe412477c8ac828e450cbc319c06ac8db0445
# Author: weizhimiao <532615323@qq.com>
# Date:   Fri Sep 23 11:19:34 2016 +0800
#
#     update
#
# commit 81c90a01f972ef803bd16272e7983aa1b3e9fa9c
# Author: weizhimiao <532615323@qq.com>
# Date:   Fri Sep 23 10:42:58 2016 +0800
#
#     add readme file
#
$ cat readme.md
# git test
# Second Lines;
```
然后我们就回到了上一次提交的版本。

9. git rm 删除所有版本库记录（慎用）


## 从远程库克隆
git clone 克隆一个本地库
```
$ git clone git@github.com:weizhimiao/git_test.git
# Cloning into 'git_test'...
# remote: Counting objects: 3, done.
# remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0
# Receiving objects: 100% (3/3), done.
# Checking connectivity... done.
$ ll
# drwxr-xr-x  4 zhimiao  staff   136B  9 23 21:53 git_test
$ cd git_test
$ ll
# -rw-r--r--  1 zhimiao  staff    12B  9 23 21:53 README.md
$ git status
# On branch master
# Your branch is up-to-date with 'origin/master'.
# nothing to commit, working directory clean
```
## 关联远程库
> 本地仓库名：git_test
> 远程仓库名：git_test
> 在本地git_test仓库下执行

```
$ git remote add origin git@github.com:weizhimiao/git_test.git
```
> weizhimiao 是github账户名
> origin 为远程仓库的名字，git的默认叫法

将本地所有的内容推送到远程库上
```
git push -u origin master 把本地master分支推送到远程库
-u 为把本地master 分支和远程master分支关联起来，之后就可以通过以下命令进行推送了
git push origin master
```

## 分享与更新项目
1. git push origin dev  提交到远程dev分支


2. git pull origin dev  拉取远程dev分支到本地并和本地dev分支合并

3. git remote add origin git@github.com:weizhimiao/git_test.git  将本地仓库推送至名为test的仓库里

## 分支管理
创建：
```
$ git branch dev
$ git branch
#   dev
# * master
$ git checkout dev
# Switched to branch 'dev'
#
$ git branch
# * dev
#   master
```
> git branch dev 创建dev分支
> git checkout dev  切换当前分支

等价于
```
$ git checkout -b dev 创建并切换到dev分支
```
```
# git branch 查看当前分支
# git branch -a 查看本地和远程所有分支
# git branch -r 常看远程分支
# git branch -d 删除本地分支
# git checkout master 用于dev分支完成工作后，切换回master 分支
#
# git merge 分支合并
# 如当前分支是master，本地另一个分支是dev，用下面命令将dev合并到master分支
# git merge dev
```
## 版本回退
```
# git log 查看历史纪录
#
# 回退到上一个版本
# git reset -hard HEAD^
# 或
# git reset --hard [commit id]回退至指定版本号的版本
#
#
#
# git中
# HEAD表示当前版本
# HEAD^表示上一个版本
# HEAD^^ 上上一个版本
# HEAD~100 上100个版本
#
# git reflog 查看命令历史
# 一般通过这个命令查看之前版本号
# 例如：（前7个字符就是版本号的缩写）
$ git reflog
bb862b6 HEAD@{0}: merge dev: Fast-forward
3223509 HEAD@{1}: checkout: moving from dev to master
bb862b6 HEAD@{2}: commit: dev branch commint
3223509 HEAD@{3}: checkout: moving from master to dev
3223509 HEAD@{4}: checkout: moving from master to master
3223509 HEAD@{5}: checkout: moving from dev to master
3223509 HEAD@{6}: checkout: moving from master to dev
3223509 HEAD@{7}: commit: add readme.md
54906f2 HEAD@{8}: pull origin master: Merge made by the 'recursive' strategy.
7563423 HEAD@{9}: reset: moving to HEAD^
7865e6f HEAD@{10}: commit: update third
7563423 HEAD@{11}: commit: update
81c90a0 HEAD@{12}: reset: moving to HEAD^
ed0fe41 HEAD@{13}: reset: moving to HEAD^
2ba4ebf HEAD@{14}: commit: update third
ed0fe41 HEAD@{15}: commit: update
81c90a0 HEAD@{16}: commit (initial): add readme file
```
## 管理修改
> git diff HEAD -- README.md

查看工作区和版本库里最新版本的区别
```
$ git diff HEAD -- README.md
# diff --git a/README.md b/README.md
# index 9235721..62c0eaa 100644
# --- a/README.md
# +++ b/README.md
# @@ -1 +1 @@
# -First Line!
# +branch dev line
```

## 撤销修改
1. 修改了工作区，想直接丢弃
```
$ git checkout -- filename
```
2. 修改了工作区内容，同事添加到了暂存区
```
$ git reset HEAD filename
$ git checkout -- filename
```

## 删除文件
```
$ git rm filename
$ git commit 提交到版本库
```
