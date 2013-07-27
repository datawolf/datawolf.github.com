---
layout: post
title: "which 命令分析"
date: 2013-04-14 17:03
comments: true
categories: [ubuntu, linux, which, shell]
---

## 环境
Linux 3.2.0-40-generic #64-Ubuntu SMP Mon Mar 25 21:22:10 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux
### which 命令的位置
``` bash
datawolf@datawolf-ThinkPad-Edge:~$ which -a which
/usr/bin/which
/bin/which
```
### 分析两个which文件的类型
通过以下shell命令可以看出，`/usr/bin/which`是一个指向`/bin/which`的符号链接。而`/bin/which`是一个shell脚本。
``` bash
datawolf@datawolf-ThinkPad-Edge:~$ ls -al  /usr/bin/which /bin/which 
-rwxr-xr-x 1 root root 946 Mar 30  2012 /bin/which
lrwxrwxrwx 1 root root  10 Feb 21 16:23 /usr/bin/which -> /bin/which
datawolf@datawolf-ThinkPad-Edge:~$ file /usr/bin/which 
/usr/bin/which: symbolic link to `/bin/which'
datawolf@datawolf-ThinkPad-Edge:~$ file /bin/which 
/bin/which: POSIX shell script, ASCII text executable
```
<!-- more -->
### /bin/which 文件分析
**which**命令的原理：该命令在系统`$PATH`环境变量指向的目录中，寻找是否有指定的可执行的命令。
其用发如下：
``` bash
$ which -a ping # 寻找所有的ping命令在系统中的路径位置
$ which ping # 寻找第一个ping命令在系统中的路径位置
```
**which**命令的源代码如下：
``` bash
#!/bin/sh

# -e 如果命令带非0返回值，立即退出
# -f 禁止带扩展名的路径
set -ef

# $KSH_VERSION变量表示ksh的版本
if test -n "$KSH_VERSION"; then
	puts() {
		print -r -- "$*"
	}
else
	puts() {
		printf '%s\n' "$*"
	}
fi

ALLMATCHES=0

# 查找是否有选项-a
# 如果有选项-a，打印所有匹配的命令，否则打印第一个匹配的命令
while getopts a whichopts
do
        case "$whichopts" in
                a) ALLMATCHES=1 ;;
                ?) puts "Usage: $0 [-a] args"; exit 2 ;;
        esac
done

# shift n 表示将原来的位置参数左移n位。
# $OPTIND 代表下一个待处理的参数的索引
shift $(($OPTIND - 1))

# $#标表示参数的个数
if [ "$#" -eq 0 ]; then
 ALLRET=1
else
 ALLRET=0
fi
case $PATH in
	(*[!:]:) PATH="$PATH:" ;;
esac

# $@代表输入的所有的命令行参数
# $IFS表示bash内部的域分割符。默认值为空格
# [-f "filename"] 判断是否是一个文件
# [-x "filename"] 判断文件是否具有可执行权限
# [-z "str"] 判断字符串长度是否为0
#

for PROGRAM in "$@"; do
 RET=1
 IFS_SAVE="$IFS"
 IFS=:
# 该程序首先判断具有全部路径的命令是否存在，比如这样使用： which /usr/bin/ls
# 如果当使用如下时： which ls 该程序将在系统$PATH的路径下寻找相应的程序。
 case $PROGRAM in
  */*)
   if [ -f "$PROGRAM" ] && [ -x "$PROGRAM" ]; then
    puts "$PROGRAM"
    RET=0
   fi
   ;;
  *)
   for ELEMENT in $PATH; do
    if [ -z "$ELEMENT" ]; then
     ELEMENT=.
    fi
    if [ -f "$ELEMENT/$PROGRAM" ] && [ -x "$ELEMENT/$PROGRAM" ]; then
     puts "$ELEMENT/$PROGRAM"
     RET=0
     [ "$ALLMATCHES" -eq 1 ] || break
    fi
   done
   ;;
 esac
 IFS="$IFS_SAVE"
 if [ "$RET" -ne 0 ]; then
  ALLRET=1
 fi
done
 
exit "$ALLRET"

```

### CentOS 中的which命令

#### 环境

Linux 2.6.32-220.17.1.el6.x86_64 #1 SMP Wed May 16 00:01:37 BST 2012 x86_64 x86_64 x86_64 GNU/Linux

which - shows the full path of (shell) commands

Which takes one or more arguments. For each of its arguments it prints to stdout the full path of the executables that would have been executed when this argument had been entered at the shell prompt. It does this by searching for an executable or script in the directories listed in the environment variable PATH using the same algorithm as bash(1)

#### 在CentOS下，which是c语言写的
``` bash
[wanglong@wanglong ~]$ which -a which
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
	/usr/bin/which
/usr/bin/which
[wanglong@wanglong ~]$ file /usr/bin/which 
/usr/bin/which: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.18, stripped
[wanglong@wanglong ~]$ ldd /usr/bin/which 
	linux-vdso.so.1 =>  (0x00007fff74fff000)
	libc.so.6 => /lib64/libc.so.6 (0x0000003250400000)
	/lib64/ld-linux-x86-64.so.2 (0x0000003250000000)
[wanglong@wanglong ~]$ 
```





