---
layout: post
title: "Linux 时钟和定时器"
date: 2013-06-01 19:38
comments: true
categories: [linux, time, timer]
---

## 时钟 ##
时钟这个东西，实际上是作为一种工具而存在，内核通过时钟来感知、管理时间。这里的时钟，更主要的还是**软件上的概念**，系统通过维护软件时钟来追踪时间。
### 概念 ###
1. 	**时钟中断**：由硬件产生的电信号，一切的缘起。该中断产生时，内核通过特殊的中断处理程序进行处理。
2. 	**节拍率**（tick rate）：系统以某种频率（可编程）自行触发（hitting、popping）时钟中断（即系统定时器的频率）。
3. 	**节拍**（tick）：由于节拍率已知，系统当然也知道两次时钟中断之间所间隔的时间，这个时间就是时钟节拍。

### 再说节拍率：HZ ###
**节拍率**，即系统定时器的频率，在内核中通过`HZ`这个宏进行定义。在进行内核编程的时候，切记不要假设`HZ`不会发生变化，事实上，大多数体系结构的HZ都是可调的。

`HZ`的理想取值：从2.5内核开始，这个取值在i386体系结构中就改为了`1000`（2.6.13版本后的内核，加入了250这个取值）。改变HZ的取值，对于操作系统而言，意味着改变时钟中断的频率：

 **增大HZ**：提高时钟中断的频率，这带来的好处是，提高了时间驱动事件的解析度与精确度,内核定时器具有更高的频度与精确度（依赖内核定时器的系统调用也有了更精确的执行度，比如select、epoll等，这会带来很大的性能提升），时间相关的测量会更准确，内核抢占更准确，进程调度的响应更及时。

当然也会有负面影响：更高的中断频率，必然会导致系统消耗更多的资源来处理时钟中断（当然，就目前的主机来说，`1000`还是一个不错的取值）。

2.6的内核允许在编译的时候，选择不同的HZ取值，用户空间的`USER_HZ`，是根据内核的`HZ`进行了相应的转换。

最后顺便说一下，OS也是可以采取无节拍的实现的，但系统的开销会非常大。

### jiffies ###
变量类型为`unsigned long volatile`，该变量记录了系统启动以来，产生的`tick`总数，`系统运行时间 = jiffies/HZ` 。

下面是几个典型运用：
``` c
	unsigned long timestamp = jiffies
	unsigned long nextTick = jiffies + 1
	unsigned long 5sLater = jiffies + 5*HZ
```
<!-- more -->
作为`unsigned long`，在64位机上是不会溢出的。而在32位机上，当`HZ=1000`时，49.7天将发生溢出，会进行所谓的回绕，下面是几个处理回绕的宏(`include/linux/jiffies.h`)：
``` c
    /*
     *  These inlines deal with timer wrapping correctly. You are
     *  strongly encouraged to use them
     *  1. Because people otherwise forget
     *  2. Because if the timer wrap changes in future you won't have to
     * alter your driver code.
     *
     * time_after(a,b) returns true if the time a is after time b.
     *
     * Do this with "<0" and ">=0" to only test the sign of the result. A
     * good compiler would generate better code (and a really good compiler
     * wouldn't care). Gcc is currently neither.
     */
    #define time_after(a,b) \
    	(typecheck(unsigned long, a) && \
    	 typecheck(unsigned long, b) && \
    	 ((long)(b) - (long)(a) < 0))
    #define time_before(a,b)time_after(b,a)
    
    #define time_after_eq(a,b)  \
    	(typecheck(unsigned long, a) && \
    	 typecheck(unsigned long, b) && \
    	 ((long)(a) - (long)(b) >= 0))
    #define time_before_eq(a,b) time_after_eq(b,a)

    /*
     * Calculate whether a is in the range of [b, c].
     */
    #define time_in_range(a,b,c) \
    	(time_after_eq(a,b) && \
     	time_before_eq(a,c))
    
    /*
     * Calculate whether a is in the range of [b, c).
     */
    #define time_in_range_open(a,b,c) \
    	(time_after_eq(a,b) && \
    	 time_before(a,c))
    
    /* Same as above, but does so with platform independent 64bit types.
     * These must be used when utilizing jiffies_64 (i.e. return value of
     * get_jiffies_64() */
    #define time_after64(a,b)   \
    	(typecheck(__u64, a) && \
     	typecheck(__u64, b) && \
     	((__s64)(b) - (__s64)(a) < 0))
    #define time_before64(a,b)  time_after64(b,a)
    
    #define time_after_eq64(a,b)\
    	(typecheck(__u64, a) && \
    	 typecheck(__u64, b) && \
     	((__s64)(a) - (__s64)(b) >= 0))
    #define time_before_eq64(a,b)   time_after_eq64(b,a)
    
    /*
     * These four macros compare jiffies and 'a' for convenience.
     */
    
    /* time_is_before_jiffies(a) return true if a is before jiffies */
    #define time_is_before_jiffies(a) time_after(jiffies, a)
    
    /* time_is_after_jiffies(a) return true if a is after jiffies */
    #define time_is_after_jiffies(a) time_before(jiffies, a)
    
    /* time_is_before_eq_jiffies(a) return true if a is before or equal to jiffies*/
    #define time_is_before_eq_jiffies(a) time_after_eq(jiffies, a)
    
    /* time_is_after_eq_jiffies(a) return true if a is after or equal to jiffies*/
    #define time_is_after_eq_jiffies(a) time_before_eq(jiffies, a)
```

## 时间 ##
讲Linux系统编程的书，都要讲时间以及相关接口，因为这个概念是不可或缺的，许多功能都需要时间进行支撑。

### 几个时间 ###
1. **墙上时间**：墙上挂钟的时间，即真是时间。用户级进程一般使用该时间。
2. **进程时间**：进程消耗的时间，包括用户空间的和内核空间的，一般在对程序进行分析、统计的时候使用。
3. **单调时间**：保持稳定线性递增的时间，其重要性不在于绝对值是多少，而在于通过多次采样并计算相对时间。单调时间不会因为墙上时间的改变而受到影响。

 
系统衡量这几个时间的方式有两个：**绝对时间**，**相对时间**
### POSIX时钟 ###
`POSIX`定义了若干种时钟，这些`POSIX`时钟其实是指的**实现和表示时间源的标准**，Linux支持以下四种：

`MONOTONIC`：不可被设置的，单调增长的时钟

`PROCESS_CPUTIME_ID`：系统为每进程提供的高精度时钟

`REALTIME`：墙上时钟，设置需要root权限  （必须实现的）

`THREAD_CPUTIME_ID`：系统为每线程提供的高精度时钟
``` c
	/*
	 * The IDs of the various system clocks (for POSIX.1b interval timers):
	 */
	#define CLOCK_REALTIME                  0
	#define CLOCK_MONOTONIC                 1
	#define CLOCK_PROCESS_CPUTIME_ID        2
	#define CLOCK_THREAD_CPUTIME_ID         3
	#define CLOCK_MONOTONIC_RAW             4
	#define CLOCK_REALTIME_COARSE           5
	#define CLOCK_MONOTONIC_COARSE          6
	#define CLOCK_BOOTTIME                  7
	#define CLOCK_REALTIME_ALARM            8
	#define CLOCK_BOOTTIME_ALARM            9

```
## 延迟执行 ##
Linux系统提供了许多种让任务延后执行的方法，我们最为熟悉的自然是定时器了，但在细说定时器之前，我们也有必要了解其他的延迟机制。
### 忙等待 ###
**忙等待**的原理最简单：通过不断的循环，直到经过了设定的时钟节拍。

下面是其典型示例：
``` c
	unsigned long  delay  =  jiffies + 100;
	While (time_before (jiffies , delay ));
```
忙等待唯一的优点就是实现简单了，但是缺点却有一箩筐`精度低、等待时占用CPU资源等。因此我们一般不会在正式的代码中发现它的身影。

### 短延迟 ###
内核提供一种**微秒级**的循环等待机制，其原理是：**内核知道系统每秒能够执行多少次循环，然后通过执行特定次数的循环来达到延时的目的**

接口：`udelay() / mdelay()`  入参均为微秒/毫秒数，并不使用`jiffies`

这里有一个相关变量：`BogoMIPS` ，该变量记录的是，处理器在给定时间内忙循环执行的次数，主要就是被上述2个接口使用。

### 在等待队列上睡眠 ###
在学习进程的时候，我们已经知道当进程等待某个事件的发生时，是能够将自己挂起的，即将自己加入等待队列中。这就是系统提供的第三种延迟机制。

在学习进程时，我们知道了schedule()，这里对应的会有schedule_timeout()，顾名思义，该调用与时间有关：作用是在等待队列上睡眠指定的时间（精度为节拍）。使用这个机制时，需要首先设置进程状态为`TASK_INTERRUPTIBLE`(处理信号 )/ `TASK_UNINTERRUPTIBLE`(不处理信号)，否则进程不会睡眠

## Linux定时器 ##

### Linux提供的睡眠系统调用——临时定时器 ###
Linux提供了几个睡眠的系统调用
```
	sleep()				// s
	usleep()			// us
	nanosleep()			// ns
	clock_nanosleep()	// POSIX规范接口，更高级，提供各种类型的定时器
```
另外，应该知道的是，select调用也是能够实现微秒级的定时器的 ，而且移植性较好。

同时在调用这些接口时，需要处理返回值或者errno，以提高程序的健壮性，比如睡眠被信号打断这类的事件。

### 进程唯一的实时定时器 ###
对于进程唯一的实时定时器的使用，系统提供了两类接口：`alarm/ualarm`，以及 `getitimer/setitimer`

前者(`alarm/ualarm`)是简单的小闹钟，结合信号量使用，精确度在秒级。

后者(`getitimer/setitimer`)则更为高级，其提供了3种工作模式，能够较好的过滤掉诸如上下文切换带来的统计误差：
```
	REAL：记录墙上时间的流逝（SIGALARM）
	VIRTUAL：只记录进程空间的时间流逝（SIGVTALARM）
	PROF：在进程执行以及内核为进程服务时，记录时间的流逝（SIGPROF）
```
### POSIX实时扩展定时器 ###
如果需要更高的时间精度，或者需要多个定时器，那么每个进程一个的实时间隔定时器就无能为力了，这个时候我们可以选择`POSIX实时扩展中的定时器`：
``` c
    int timer_create(clockid_t clockid, 
				struct sigevent *restrict evp,
				timer_t *restrict timerid);
    int timer_getoverrun(timer_t timerid);
    int timer_gettime(timer_t timerid, struct itimerspec *value);
    int timer_settime(timer_t timerid, int flags,
       			const struct itimerspec *restrict value,
       			struct itimerspec *restrict ovalue);
```
它实际上就是**进程间隔定时器**的增强版，除了可以定制时钟源（nanosleep也存在能定制时钟源的版本：clock_nanosleep）和时间精度提高到纳秒外，它还能通过将`evp->sigev_notify`设定为如下值来定制定时器到期后的行为：



1. **SIGEV_SIGNAL**：发送由`evp->sigev_sino`指定的信号到调用进程，`evp->sigev_value`的值将被作为`siginfo_t`结构体中`si_value`的值。
2. **SIGEV_NONE**：什么都不做，只提供通过`timer_gettime`和`timer_getoverrun`查询超时信息。
3. **SIGEV_THREAD**：以`evp->sigev_notification_attributes`为线程属性创建一个线程，在新建的线程内部以`evp->sigev_value`为参数调用`evp->sigev_notification_function`。
4. **SIGEV_THREAD_ID**：和`SIGEV_SIGNAL`类似，不过它只将信号发送到线程号为`evp->sigev_notify_thread_id`的线程，注意：这里的线程号不一定是`POSIX`线程号，而是线程调用`gettid`返回的实际线程号,并且这个线程必须实际存在且属于当前的调用进程。


###  其它问题 ###


#### 更高的时间精度 ####
从前面的描述我们可以发现，Linux定时器很多都是基于时钟节拍的，那么当`HZ`较低比如100时，其节拍精度仅为10毫秒，那么显然对于一个想要在1毫秒级精度进行睡眠的进程而言，这个精度是无法容忍的，我们可以考虑适当提高`H`Z取值。

另外，Linux的API暗示它能够提供**纳秒级**的时间精度，但是，由于种种不确定因素，它实际上并不能提供纳秒级的精度，比较脆弱。如果你需要更高强度的实时性，请考虑采用软实时系统、硬实时系统、专有系统，甚至是专业硬件。

#### 传统信号的不可靠性 ####
传统UNIX信号是不可靠的，也就是说如果当前的信号没有被处理，那么后续的同类信号将被丢失，而不是被排队，而实时信号则没有这个问题，它是被排队的。联系到当前应用，如果信号丢失，则是因为任务消耗了过多的处理器时间，而这个不确定性是那个任务带来的，需要改进的应该是那个任务。
#### 系统负载过高 ####
如果系统的负载过高，使得我们的程序因为不能得到及时的调度导致时间精度降低，我们不妨通过`nice`提高当前程序的优先级，必要时可以通过`sched_setscheduler`将当前进程切换成优先级最高的实时进程以确保得到及时调度。
#### 硬件相关的问题 ####
硬件配置也极大的影响着定时器的精度，有的比较老的遗留系统可能没有比较精确的硬件定时器，那样的话我们就无法期待它能提供多高的时钟精度了。相反，如果系统的配置比较高，比如说对称多处理系统，那么即使有的处理器负载比较高，我们也能通过将一个处理器单独分配出来处理定时器来提高定时器的精度。
