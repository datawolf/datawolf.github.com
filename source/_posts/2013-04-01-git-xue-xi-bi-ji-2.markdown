---
layout: post
title: "git 学习笔记 2"
date: 2013-04-01 14:14
comments: true
categories: 
---

历史
==

显示全部历史记录
``` bash
$ git log
```
显示版本历史，以及版本间的内容差异
``` bash
$ git log -p
```
只显示最近一个提交
``` bash
$ git log -1
```
显示最近的20个提交，以及版本间的内容差异
``` bash
$ git log -20 -p
```
显示最近6个小时的提交
``` bash
$ git log --since="6 hours"
```
显示两天前的提交
``` bash
$ git log --before="2 days"
```
显示比HEAD（当前检出分支的末稍）早三个提交的那个提交
``` bash
$ git log -1 HEAD~3
$ git log -1 HEAD^^^
$ git log -1 HEAD~1^^
```
显示两个版本直接的提交
``` bash
$ git log <start point>...<end point>
```
<!-- more -->
显示历史，每一个提交显示一行，包括提交注释的第一行
``` bash
$ git log --pretty=oneline
```
显示改动行数统计
``` bash
$ git log --stat
```
显示改动文件的名称和状态
``` bash
$ git log --name-status
```
显示当前工作目录树和暂存区间的差别
``` bash
$ git diff
```
显示暂存区和版本库之间的差别
``` bash
$ git diff --cached
```
显示工作目录树和版本库间的差别
``` bash
$ git diff HEAD
```
显示工作目录树和版本库中某次提交版本之间的差别
``` bash
$ git diff <start point>
```
显示版本库中两个版本之间的差别
``` bash
$ git diff <start point> <end point>
```
显示差别的相关统计
``` bash
$ git diff --stat <start point> [<end point>]
```
显示文件中各个部分的修改者及相关提交信息
``` bash
$ git blame <some file>
```
显示文件中各部分的修改者及相关提交信息，包括在文件移动内容方面的情况
``` bash
$ git blame <some file>
$ git blame -M <some file>
```
显示历史时，显示复制和粘贴信息
``` bash
$ git log -C -C -p -1 <some point>
```

远程版本库
=====
克隆远程版本库
``` bash
$ git clone <some repository>
```
克隆版本库，但只下载其中最近200个提交的历史记录
``` bash
$ git clone --depth 200 <some repository>
```
在本地版本库中设置远程版本库的别名
``` bash
$ git remote add <remote repository> <repository url>
```
显示远程分支
``` bash
$ git branch -r
```
基于远程分支创建本地分支
``` bash
$ git branch <new branch> <remote branch>
```

基于远程标签创建本地分支
``` bash
$ git branch <new branch> <remote tag>
```
从别名为``orgin``的远程版本库中取来修改变化，但不合并到本地分支
``` bash
$ git fetch 
```
从任意的远程版本库中取来修改变化，但不合并到本地分支
``` bash
$ git fetch <remote repository>
```
从任意的远程版本库中取来修改变化，并合并到当前检出的本地分支
``` bash
$ git pull <remote repository>
```
从别名``orgin``的远程版本库中取来修改变化，并合并到当前检出的本地分支
``` bash
$ git pull
```
把修改变化从本地分支推入远程版本库
``` bash
$ git push <remote repository> <local branch>:<remote branch>
```
把修改变化从本地分支推入远程版本库中同名分支
``` bash
$ git push <remote repository> <local branch>
```
把修改变化从本地新建分支推入远程版本库
``` bash
$ git push <remote repository> <local branch>
```
把修改变化推入别名为``orgin``的远程版本库
``` bash
$ git push
```
在远程版本库中删除分支
``` bash
$ git push <remote repository>  :<remote branch>
```
在本地版本库中删除所有远程版本库中已不存在的分支
``` bash
$ git remote prune <remote repository>
```
在本地版本库中删除某个远程版本库的简称，以及该远程版本库相关的分支
``` bash
$ git remote rm <remote repository>
```

操作示例
====
查看git日志
``` bash
$ git log
$ git log 7b1558c
$ git log -10
$ git log --since="5 hours"
$ git log --since="5 hours" -1
$ git log 18f822e..0bb3dfb
$ git log 18f822e..HEAD
$ git log --pretty=format:"%h %s" 1.0..HEAD

```

查看版本之间的差异
``` bash
$ git diff 18f822e
$ git diff --stat 1.0
```

查明该向谁问责
``` bash
$ git blame hello.html
$ git blame -L 12,13 hello.html
$ git blame -L 12,+2 hello.html
$ git blame -L 12,-2 hello.html
$ git blame -: "/<\/body>/",+2 hello.html
```

复位操作，将版本库复位到``HEAD``之前的那个版本了
``` bash
$ git reset --hard HEAD^
```
显示远程分支
``` bash
$ git branch 
* master
$ git branch -r
  origin/HEAD -> origin/master
  origin/about
  origin/alternate
  origin/contacts
  origin/master
  origin/new

```

更新远程分支，但不会把远程分支上的修改合并到本地分支上。
``` bash
$ git fetch
```

取来远程分支并且合并，需要两个参数，一个是远程版本库名称，另一个需要拖入的远程分支名称
``` bash
$ git pull
```

推入改动，调用不带参数的``git push``命令时，git会推入到默认版本库origin中，并把本地版本库中当前所在分支的
变更推入到远程版本库对应的分支上。
``` bash
$ git push
```

将本地分支上``mybranch``上的提交推入远程版本库的主分支上
``` bash
$ git push origin  mybranch:master
```

添加新的远程版本库，必须有相应的权限
