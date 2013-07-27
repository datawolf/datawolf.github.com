---
layout: post
title: "Linux世界里的时间"
date: 2013-05-26 20:33
comments: true
categories: [Linux, Time, Clock]
---

通常，操作系统可以使用三种方法来表示系统的当前时间与日期：

1. 简单的一种方法就是直接用一个64位的计数器来对**时钟滴答**进行计数。
2. 第二种方法就是用一个32位计数器来对**秒**进行计数，同时还用一个32位的辅助计数器对时钟滴答计数，直至累积到一秒为止。因为23^2超过136年，因此这种方法直至22世纪都可以让系统工作得很好。
3. 第三种方法也是按**时钟滴答**进行计数，但是是相对于系统启动以来的滴答次数，而不是相对于某个确定的外部时刻；当读外部后备时钟（如RTC）或用户输入实际时间时，根据当前的滴答次数计算系统当前时间。

## 基本概念 ##
首先，有必要明确一些Linux内核时钟驱动中的基本概念。
###  时钟周期的频率 ###
时钟周期（clock cycle）的频率：`8253／8254 PIT`的本质就是对由晶体振荡器产生的
时钟周期进行计数，**晶体振荡器在1秒时间内产生的时钟脉冲个数就是时钟周期的频率**。

Linux用宏`CLOCK_TICK_RATE`来表示`8254 PIT`的输入时钟脉冲的频率,该宏定义在`arch/x86/include/asm/timex.h`头文件中：
``` c
    ./arch/x86/include/asm/timex.h:8:#define CLOCK_TICK_RATE  PIT_TICK_RATE
    ./include/linux/timex.h:153:#define PIT_TICK_RATE 1193182ul
```
### 时钟滴答 ###
时钟滴答（clock tick）：当PIT通道0的计数器减到0值时，它就在`IRQ0`上产生一次时钟中断，也即一次**时钟滴答**。PIT通道0的计数器的初始值决定了要过多少时钟周期才产生一次时钟中断，因此也就决定了一次时钟滴答的时间间隔长度。 
### 时钟滴答的频率 ###
时钟滴答的频率（HZ）：即**1秒时间内PIT所产生的时钟滴答次数**。类似地，这个值也是由PIT通道0的计数器初值决定的（反过来说，确定了时钟滴答的频率值后也就可以确定8254 PIT通道0的计数器初值）。

Linux内核用宏`HZ`来表示时钟滴答的频率，而且在不同的平台上`HZ`有不同的定义值。对于ALPHA和IA62平台HZ的值是1024，对于SPARC、MIPS、ARM和i386等平台`HZ`的值都是100。该宏在i386平台上的定义如下: 
``` c
    #ifndef HZ 
    #define HZ 100 
    #endif 
```
根据`HZ`的值，我们也可以知道一次时钟滴答的具体时间间隔应该是（1000ms／HZ）＝10ms。
<!-- more -->
### 时钟滴答的时间间隔 ###
时钟滴答的时间间隔：Linux用全局变量`tick`来表示时钟滴答的时间间隔长度，该变量定义在`kernel/timer.`c文件中，如下： 
``` c
	long tick = (1000000 + HZ/2) / HZ; /* timer interrupt period */ 
```
`tick`变量的单位是**微妙**（μs），由于在不同平台上宏HZ的值会有所不同，因此方程式tick＝1000000÷HZ的结果可能会是个小数，因此将其进行四舍五入成一个整数，所以Linux将`tick`定义成（1000000＋HZ／2）／HZ，其中被除数表达式中的HZ／2的作用就是用来将`tick`值向上圆整成一个整型数。 
 
另外，Linux还用宏`TICK_SIZE`来作为tick变量的引用别名（alias），其定义如下（`arch／i386/kernel/time.c`）： 
``` c
	#define TICK_SIZE tick 
``` 
### 宏LATCH ###
宏LATCH：Linux用宏`LATCH`来定义**要写到PIT通道0的计数器中的值**，它表示PIT将每隔多少个时钟周期产生一次时钟中断。显然LATCH应该由下列公式计算： 

	LATCH＝（1秒之内的时钟周期个数）÷（1秒之内的时钟中断次数）＝（CLOCK_TICK_RATE）÷（HZ） 

类似地，上述公式的结果可能会是个小数，应该对其进行四舍五入。所以，Linux将`LATCH`定义为（`/include/linux/jiffies.h`）： 
``` c
	./include/linux/jiffies.h:54:/* LATCH is used in the interval timer and ftape setup. */
	./include/linux/jiffies.h:55:#define LATCH ((CLOCK_TICK_RATE + HZ/2) / HZ)	/* For divider */
```
类似地，被除数表达式中的HZ／2也是用来将`LATCH`向上圆整成一个整数。 

## 时间的内核数据结构 ##
作为一种UNIX类操作系统，Linux内核显然采用本文开始所述的第三种方法来表示系统的当前时间。Linux内核在表示系统当前时间时用到了三个重要的数据结构：
### 全局变量jiffies ###
**全局变量jiffies**：这是一个32位的无符号整数，用来表示自内核上一次启动以来的时钟滴答次数。每发生一次时钟滴答，内核的时钟中断处理函数`timer_interrupt()`都要将该全局变量`jiffies`加1。

该变量定义在`include/linux/jiffies.h`源文件中，如下所示：
``` c
	  /*
	   * The 64-bit value is not atomic - you MUST NOT read it
	   * without sampling the sequence number in jiffies_lock.
	   * get_jiffies_64() will do this for you as appropriate.
	   */
	  extern u64 __jiffy_data jiffies_64;
	  extern unsigned long volatile __jiffy_data jiffies;
```
C语言限定符`volatile`表示`jiffies`是一个易该变的变量，因此编译器将使对该变量的访问从不通过CPU内部cache来进行。

`__jiffy_data`的定义如下：
``` c
      /* some arch's have a small-data section that can be accessed register-relative
       * but that can only take up to, say, 4-byte variables. jiffies being part of
       * an 8-byte variable may not be correctly accessed unless we force the issue
       */
      #define __jiffy_data  __attribute__((section(".data")))
``` 
### 全局变量xtime ###
**全局变量xtime**：它是一个`timeval`结构类型的变量，用来表示当前时间距UNIX时间基准
1970－01－01 00：00：00的**相对秒数值**。结构`timeval`是Linux内核表示时间的一种格式
（Linux内核对时间的表示有多种格式，每种格式都有不同的时间精度），其时间精度是**微秒**。
该结构是内核表示时间时最常用的一种格式，它定义在头文件`include/uapi/linux/time.h`中，
如下所示：
``` c
    struct timeval {
      		__kernel_time_t    tv_sec; /* seconds */
      		__kernel_suseconds_t    tv_usec;/* microseconds */
      };
```
`__kernel_time_t`的定义如下：
``` c
    typedef __kernel_long_t __kernel_time_t;
    typedef long long __kernel_long_t;
```

`__kernel_suseconds_t`的定义如下：
``` c
	#ifndef __kernel_suseconds_t
	  typedef __kernel_long_t         __kernel_suseconds_t;
	#endif
	typedef long long __kernel_long_t;
```
其中，成员`tv_sec`表示当前时间距UNIX时间基准的秒数值，而成员`tv_usec`则表示一秒之内的
微秒值，且`1000000>tv_usec>＝0`。Linux内核通过`timeval`结构类型的全局变量`xtime`来维持当前时间，该变量定义在`kernel/timer.c`文件中，如下所示： 
``` c
	/* The current time */ 
	volatile struct timeval xtime __attribute__ ((aligned (16))); 
```
### 全局变量sys_tz ###

**全局变量sys_tz**：它是一个`timezone`结构类型的全局变量，表示系统当前的时区信息。结构类型`timezone`定义在`include/uapi/linux/time.h`头文件中，如下所示:
``` c
	  struct timezone {
	          int     tz_minuteswest; /* minutes west of Greenwich */
	          int     tz_dsttime;     /* type of dst correction */
	  };
```
基于上述结构，Linux在`kernel/time.c`文件中定义了全局变量`sys_tz`表示系统当前所处的时区信息，如下所示：
``` c
	  /*
	   * The timezone where the local system is located.  Used as a default by some
	   * programs who obtain this value by using gettimeofday.
	   */
	  struct timezone sys_tz;
````
