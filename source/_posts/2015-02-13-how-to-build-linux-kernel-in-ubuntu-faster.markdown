---
layout: post
title: "how to build linux kernel in ubuntu faster?"
date: 2015-02-13 22:39
comments: true
categories: 
---

##传统编译方式

通常，我们要需要编译linux内核，大概要经历以下几个步骤：
### 配置内核

make menuconfig

### 编译内核和模块

依次执行如下命令：

make
make modules
make modules_install
make install

## 使用make-kpkg

如果是ubuntu的用户，可以使用make-kpkg简化这个流程，而且还能带来额外的好处。
在ubuntu下，安装kernel-package这个包之后，就可以使用make-kpkg了。

sudo apt-get install kernel-package

使用make-kpkg编译内核，第一个步骤“配置内核是必不可少的”，在这里，我比较建议在发行版默认的config的基础上再进行配置，这样配置出来的内核和发行版本身具有更好的相容性。比如ubuntu 14.10,可以在运行“make menuconfig”之前执行命令“cp /boot/config-3.13.0-44-generic .config”。

配置完内核后，接下来执行真正的编译过程。通常我们可以执行如下命令：

make-kpkg  --initrd -j 8 --append-to-version -20150213001 kernel_image

- --initrd选项会让make-kpkg自动帮助我们生成initramfs
- --revisin会让生成的deb文件加上一个版本信息，这个参数只影响到文件名。
- --append-to-version 也是一种版本信息，它不仅会出现在deb安装包的名称中，也会影响到kernel的名称。
- kernel_image表示生成内核和默认模块的安装包，另外您也可以加上kernel_headers，这样也会生成一个内核头文件的安装包。

编译过程执行完毕之后，会在上层目录里生成一个deb安装包。

使用make-kpkg来编译内核，还有其他好处。因为我们是通过包管理器来安装新的内核，当不再需要这个内核时，就可以简单的通过dpkg命令、新立得软件包管理器或者Ubuntu软件中心来完全卸载，而不需要一个个手动删除修改。

如果需要详细了解make-kpkg的用法，可以查阅manual:
man make-kpkg

## TIPS

默认的config会编译很多模块，为了尽可能少的编译模块，在配置内核时，可以使用如下命令精简内核模块：

make localmodconfig
