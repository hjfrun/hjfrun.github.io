---
layout:     post
title:      Git常用命令
subtitle:   
date:       2018-07-05 17:37:35
author:     hjfrun
header-img: 
catalog: false
tags:
    - Git
---



这里收集整理下自己常用到的Git命令，本人不常用或者没用过的命令不会出现在这里。以备后面自己查阅。

`git init`

初始化一个`repo`，并在当前文件夹下创建一个`.git`文件夹

`git clone`

克隆一个`repo`

`git status`

查看当前`branch`以及状态

`git log`

查看`commit`历史

`git add`

在提交之前，`Git`会有一个暂存区（`staging area`），可以放入新添加的改动。`commit`时提交的改动是上一次加入到暂存区的改动，而不是`disk`上所有的改动。

* `git add .`: 会递归添加当前工作目录中的所有文件

`git diff`

对比分支



`git commit`

提交已经被`add`进来的改动

* `git commit -m "the commit message"`

`git rm`

* `git rm file`: 从暂存区移除文件，同时也移除出工作目录
* `git rm --cached`:从暂存区移除文件，但保留在工作目录中

`git clean`

是从工作目录中移除没有track的文件，通常的参数是`git clean -df`其中`-d`表示同时移除目录，`-f`表示`force`。

`git stash`

把当前改动压入一个栈。

* `git stash list`会显示这个栈的`list`；
* `git stash apply`取出`stash`中的上一个项目（`stash@{0}`），并且应用于当前工作目录；
* 也可以指定别的项目`git stash apply stash@{1}`；
* `git stash pop`在应用项目的同时删除它；
* `git stash drop`删除上一个，也可以指定参数删除指定的一个；
* `git stash clear`删除所有项目；

`git branch`

用来列出分支，创建分支和删除分支。

* `git branch -v`可以看每个分支的最后一次提交
* `git branch`列出本地所有分支，当前分支会被星号表示出
* `git branch (branchname)`创建一个新的分支（用这种方式创建分支的时候，分支是基于上一次提交建立的）
* `git branch -d (branchname)`删除一个分支



`git checkout`

切换到一个分支

* `git checkout -b(branchname)` 创建并切换到新的分支，这个命令结合了`git branche newbranch` 和 `git checkout newbranch`

`git fetch`

下载所有远程分支

`git pull`

拉取最新代码

`git push`

推送代码