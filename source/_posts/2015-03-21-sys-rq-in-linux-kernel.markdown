---
layout: post
title: "Sys Rq in linux kernel"
date: 2015-03-21 16:57
comments: true
categories: 
---
曾几何时，笔者在使用ubuntu系统时，内核卡死，然后就束手无策了。只能去把电源来重启系统，但这是非常危险的，有可能会造成数据的丢失或者磁盘的损害。

幸好，使用sysrq组合键可以让系统安全的进行重启，并能收集一些系统的信息，方便问题的定位。

在终端上同时按Alt, SysRq和命令键则会执行SysRq命令, SysRq键就是"Print Screen"健. 比如Alt+SysRq+b则重启机器。ALT+SysRQ+X（此处X代表命令参数）这是一组“魔术组合键”，只要内核没有被完全锁住，不管内核在做什么事情，使用这些组合键能即时打印出内核的信息。

使用sysrq组合键是了解系统目前运行情况的最佳方式。如果系统出现挂起（例如：死机等）的情况或在诊断一些和内核相关比较难解决的问题的时候，使用sysrq键是个比较好的方式。

在有些系统中，默认SysRq组合键是关闭的。 打开这个功能，运行:

	# echo 1 > /proc/sys/kernel/sysrq

关闭这个功能:

	# echo 0 > /proc/sys/kernel/sysrq

如果想让此功能一直生效，在/etc/sysctl.conf里面设置kernel.sysrq的值为1. 重新启动以后，此功能将会自动打开。

	kernel.sysrq = 1

因为打开sysrq键的功能以后，有终端访问权限的用户将会拥有一些特别的功能。因此，除非是要调试，解决问题，一般情况下，不要打开此功能。如果一定要打开，请确保你的终端访问的安全性。

有几种方式能触发sysrq事件。在带有AT键盘的一般系统上，在终端上输入一下组合键:

	Alt+PrintScreen+[CommandKey]

如果你在机器上有root权限，你能把commandkey字符写入到 `/proc/sysrq-trigger` 文件。这能帮助你通过脚本或你不在系统终端上的时候触发sysrq事件。

	# echo 'm' > /proc/sysrq-trigger

哪些类型的sysrq事件能被触发？请参见文档：https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/plain/Documentation/sysrq.txt



## 下面具体介绍一些如何安全的重启卡死的linux系统

#### 使用 SysRq 重启计算机的方法：

台式机键盘或者全尺寸键盘： `Alt + SysRq + [R-E-I-S-U-B] `

部分笔记本键盘： `Fn + Alt + SysRq + [R-E-I-S-U-B] `


解释：括号内的英文字母需要依次顺序按下，而且每次按下字母后需要间隔 5-10s 再执行下一个动作。（如 `alter +SysRq + R`，间隔10s 后再按 alter+ SysRq +E，以此类推）切记不可快速按下 `R-E-I-S-U-B` ，否则后果和 扣电池拔电源线无异！

#### 下面详细讲解一下各个序列： 

* `unRaw` – 把键盘设置为 ASCII 模式，使按键可以穿透 x server 捕捉传递给内核 
* `tErminate` – 向除 init 外进程发送 SIGTERM 信号，让其自行结束 
* `kIll` - 向除 init 以外所有进程发送 SIGKILL 信号，强制结束进程 
* `Sync` – 同步缓冲区数据到硬盘，避免数据丢失 
* `Unmount` – 将所有已经挂载的文件系统 重新挂载为只读 
* `reBoot` - 立即重启计算机 


