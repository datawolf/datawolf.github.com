---
layout: post
title: "using bat file to run python program on windows"
date: 2013-04-06 15:56
comments: true
categories: [bat, windows, python]
---


##在windows下批处理文件中调用python

由于python的变量的问题，不能在批处理文件`somefile.bat`正确使用`python somefile.py`来执行python程序。


所以需要在bat文件中临时修改python环境变量。

最终的bat文件如下：
``` bat
    @echo off 
    set PYPATH=C:\Python27;C:\Python27\Lib;C:\Python27\libs;C:\Python27\Tools\Scripts; 
    set path=%PYPATH%;%path% 
    @echo on
    
    python get_all_problems.py
    python get_need_solved.py
    python sort_problems.py
    
    pause
```
