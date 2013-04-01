---
layout: post
title: "git 学习笔记 1"
date: 2013-04-01 09:37
comments: true
categories: 
---

安装和初始化
======

配置全局用户名和电子邮件地址
``` bash 
$ git config --global user.name "yourname"
$ git config --global user.email "yourname@mail.com"
```

为特定版本库配置用户名和电子邮件
``` bash 
$ cd /path/to/repo
$ git config  user.name "yourname"
$ git config  user.email "yourname@mail.com"
```


查看配置
``` bash
$ git config --global --list
user.name=yourname
user.email=yourname@mail.com
```


在命令行中使用不同的颜色显示不同类型的内容，需要设置``color.ui``的值为``auto``或者``always``
``` bash
$ git config --global color.ui "auto"
```

初始化新版本库
``` bash 
$ mkdir /path/to/repo
$ cd /path/to/repo
$ git init
$ git add .
$ git commit -m "initial import"
```

克隆版本库
``` bash
$ git clone <repository url>
```

将目录中的内容纳入到git版本控制
``` bash
$ cd /path/to/existing/directory
$ git init
$ git add .
$ git commit -m "initial import of some project"
```

<!-- more -->

在本地版本库中设置远程版本库的别名
``` bash
$ git remote add <remote repository>  <repository url>

```
日常操作
====
添加新文件或暂存已有文件上的改动，然后提交
``` bash
$ git add <some file>
$ git commit -m "<some message>"
```
替换已有文件上的部分修改
``` bash
$ git add -p [<some file> [<some file> [and so on]]]
```

使用交互式添加文件
``` bash
$ git add -i
```

暂存已纳入git版本控制之下的文件的修改
``` bash
$ git add -u [<some file> [<some file>]]
```

提交已纳入git版本控制之下的文件的所有修改
``` bash
$ git commit -m "<some message>" -a
```

清除工作目录树中的修改
``` bash
$ git checkout HEAD <some file> [<some file>]
```

取消已暂存但尚未提交的修改的暂存标识
``` bash
$ git reset HEAD <some file> [<some file>]
```

修复上一次提交中的问题，改动相关文件并暂存……
``` bash
$ git commit -m "<some message>" --amend
```

修复上一次提交中的内容，并复用上次的提交注释
``` bash
$ git commit -C HEAD --amend
```

分支
==

列出本地分支
``` bash
$ git  branch
```
列出远程分支
``` bash
$ git branch -r
```
列出所有分支
``` bash
$ git branch -a
```
基于当前分支（的末稍）创建新分支
``` bash
$ git branch <new branch>
```
检出另一条分支
``` bash
$ git checkout <new branch>
```
基于当前分支创建新分支，同时检出该分支
``` bash
$ git checkout -b <new branch>
```
基于另一个起点，创建新分支
``` bash
$ git branch <some existing branch> <start point>
```
创建同名新分支，覆盖已有分支
``` bash
$ git branch -f <some existing branch> [<start point>]
```
移动或重命名分支
1、只有当<new branch name>不存在时
``` bash
$ git checkout -m <existing branch name> <new branch name>
```
2、如果<new branch name>以存在，就覆盖她
``` bash
$ git checkout -M <existing branch name> <new branch name>
```

把另一条分支合并到当前分支
``` bash
$ git merge <some branch>
```

合并但不提交
``` bash
$ git merge --no-commit <some branch>
```
拣选合并，并且提交
``` bash
$ git cherry-pick  <commit name>
```
拣选合并，但不提交
``` bash
$ git cherry-pick -n <commit name>
```
把一条分支上的内容压和到另一条分支（上的一个提交）
``` bash
$ git merge --squash <some branch>
```

删除分支

1、进当欲删除分支已经合并到当前分支时
``` bash
$ git branch -d <branch to delete>
```
2、不论欲删除分支是否已合并到当前分支上
``` bash
$ git branch -D <branch to delete>
```

举例说明
====


添加一个文件index.html，使用``git add``命令将该文件添加到版本库的索引
``` bash
$ git add index.html
```

使用``git commit``命令提交，即创建一个``提交记录``
``` bash
$ git commit -m "add in hello world HTML"
```

查看提交的相关信息
``` bash
$ git log
```

在项目中工作，修改了index.html，显示工作树的状态
``` bash
$ git status
$ git add index.html
$ git status
```
Git可以接受任意多次的留言的输入，每一次另起一段
``` bash
$ git commit -m "add xxx to xxx" \
	-m "this is the second commit document"
```

提交所有的修改
``` bash 
$ git commit -a
```

Git 输出一个提交条目或者多个提交条目，使用``-num``参数
``` bash
$ git log -1
$ git log -2
$ git log -5
```

创建一个分支，需要两个参数，第一个是新分支名称，第二个是父分支名称
``` bash
$ git branch RB_1.0  master
```

切换到分支RB_1.0
``` bash
$ git checkout RB_1.0
```

处理发布时，打标签命令。需要两个参数，第一个是标签的名称，第二个是分支的末稍
``` bash
$ git tag 1.0 RB_1.0
```

查看版本库中的标签列表
``` bash
$ git tag
```

分支合并，使用变基命令，``git rebase`` 跟一个参数，指的是变基到哪一条分支的末稍。
将master分支变基到RB_1.0分支的末稍
``` bash
$ git checkout master
$ git rebase RB_1.0
```
删除分支RB_1.0
``` bash
$ git branch -d RB_1.0
```
删除分支后，只是删除了分支的名称，不会删除分支上的任何实际内容，可以使用标签来找到从标签
到版本树起点的一连串的提交记录

打补丁时创建分支的过程
``` bash
$ git branch RB_1.0.1 1.0
$ git checkout RB_1.0.1
```
修改完后在合并到分支1.0中。

快速产看历史记录
``` bash
$ git log --pretty=oneline
```

为代码发布创建归档文件
``` bash
$ git archive --format=tar \
              --prefix=mysite-1.0/ 1.0 \
              | gzip > mysite-1.0.tar.gz
```
或者
``` bash
$ git archive --format=zip \
              --prefix=mysite-1.0/ 1.0 \
              > mysite-1.0.zip
```

克隆远程版本库
``` bash
$ git clone http://github.com/tswicegood/mysite.git mysite-remote
```

提交修改有三种方法

1、先暂存，在提交
```bash
$ git add some-file
$ git commit -m "changes to some-file"
```

2、直接提交，提交工作目录树中的所有修改。
``` bash
$ git commit -m "changes to some-file"  -a
```

3、直接修改，提交工作目录树中指定的修改。
``` bash
$ git commit -m "changes to some-file" some-file
```

将命令``git commit``简写为``git ci``:
``` bash
$ git config --global alias.ci "commit"
```
查看工作目录树中所有的改动
``` bash
$ git status
```

比较工作目录树与暂存区之间的区别
``` bash
$ git diff
```

比较暂存区和版本库直接的区别
```
$ git diff --cached
```

比较工作目录树（包括暂存和未暂存的修改）与版本库中的区别

```
$ git diff HEAD
```
``HEAD``指的是当前所在分支末稍的最新提交。

文件重命名与移动
```
$ git mv index.html hello.html
$ git status
$ git commit -m "rename to more appropriate name"
```

忽略文件

1、可以在工作目录树下设置文件``.gitignore``来设置版本库级别的忽略。

2、可以通过编辑``.git/info/exclude``来设置本地级的忽略。

修改分支的名称,``-m``不会覆盖已有的分支名称，所以新分支必须是唯一的。
``` bash
$ git branch -m master mymaster
```

创建新分支
``` bash
$ git branch new
$ git branch
```
检出刚才创建的分支
``` bash
$ git checkout new
```
用快捷方法创建新分支``alternate``
``` bash
$ git checkout -b alternate master
```

合并分支间的修改

1、直接合并

``` bash
$ git branch
* alternate
  master
$ touch about.html
$ git add about.html
$ git commit -m "add the skeleton of an about page"
$ git checkout master
$ git merge alternate
```

2、压合合并
``` bash
$ git checkout -b contact master
$ git add contact.html
$ git commit -m "add contact file with email"
#修改contact.html文件
$ git commit -m "add secondary email" -a
$ git checkout master
$ git merge --squash contact
$ git status
$ git commit -m "add contact" \
	-m "has primary and secondary email."
```

3、拣选合并``git cherry-pick``
``` bash
$ git cherry-pick 9826311
$ git cherry-pick -n 9826311
```

删除分支,``-d``删除合并了的分支，``-D``删除未合并的分支
``` bash
$ git branch -d about2
$ git branch -D about
```
