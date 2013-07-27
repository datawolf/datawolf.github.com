---
layout: post
title: "How to install sun jdk 6 on ubuntu linux"
date: 2013-04-03 17:12
comments: true
categories: [jdk, ubuntu, java]
---

##下载相应的版本
[下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk6u37-downloads-1859587.html)下载相应的版本，32位就x86，64位就x64，我下载的是jdk-6u37-linux-x64.bin；
##安装
1、创建安装目录
``` sh
$ sudo mkdir /usr/java
$ cd /usr/java/
```
2、拷贝下载好的jdk到安装目录
``` sh
$ sudo mv ~/jdk-6u37-linux-x64.bin ./
```
3、安装
``` sh
$ sudo ./jdk-6u37-linux-x64.bin
```
安装完后，会在`/usr/java`目录下多出一个`jdk_1.6.0_37`的目录。

##配置
1、配置环境变量

编辑`/etc/bashrc`，添加如下内容
``` plain
JAVA_HOME=/usr/java/jdk1.6.0_37
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$CLASSPATH
export JAVA_HOME PATH CLASSPATH
```
2、使环境变量生效
``` sh
$ source /etc/bashrc
```
3、替换ubuntu默认的jdk
``` sh
$ sudo update-alternatives --install /usr/bin/java  java  /usr/java/jdk1.6.0_37/bin/java 999
$ sudo update-alternatives --install /usr/bin/javac  javac  /usr/java/jdk1.6.0_37/bin/javac 999
$ sudo update-alternatives --install /usr/bin/javadoc javadoc /usr/java/jdk1.6.0_37/bin/javadoc 999

#以下三个命令会让你选择，选择刚安装好的版本就行了
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
$ sudo update-alternatives --config javadoc
```
4、检查一下，是否正确
``` sh
$ ls -lh /etc/alternatives/java*
```
结果如下：
``` plain
lrwxrwxrwx 1 root root 30 Apr  3 17:27 /etc/alternatives/java -> /usr/java/jdk1.6.0_37/bin/java
lrwxrwxrwx 1 root root 31 Apr  3 17:28 /etc/alternatives/javac -> /usr/java/jdk1.6.0_37/bin/javac
lrwxrwxrwx 1 root root 33 Apr  3 17:28 /etc/alternatives/javadoc -> /usr/java/jdk1.6.0_37/bin/javadoc
```

