---
title: 字节跳动二面题目记录
date: 2021-10-21 19:01:53
tags: ["面试"]
---

昨天下午参加了两场面试，其中快手一面很顺利就通过了，本以为字节二面也不会太难，结果被面到自闭...在此记录一下自己不会/不够清楚的内容以便复习（悲

<!--more-->

## 计算机组成原理相关

### 有了解 CPU 的缓存结构吗？

参考博文：

1. [程序优化：CPU 缓存基础知识 - 知乎](https://zhuanlan.zhihu.com/p/80672073)

1. [与程序员相关的 CPU 缓存知识 - 酷壳](https://coolshell.cn/articles/20793.html)

## 计算机网络相关

### time wait 状态过多会导致的问题？如何解决？

端口的范围是 0-65535，去掉系统和其它服务占用的部分，TCP 可用的端口其实并不多。在这种情况下，如果有大量连接停留在 TIMEWAIT 状态，会导致其它 HTTP 请求来临的时候无法使用这些端口。即：**持续的到达一定量的高并发短连接，会使服务器因端口资源不足而拒绝为一部分客户服务。**

解决方法：

1. [解决 TIME_WAIT 过多造成的问题 - 博客园](https://www.cnblogs.com/dadonggg/p/8778318.html)

2. [Linux TCP 状态 TIME_WAIT 过多的处理 - 知乎](https://zhuanlan.zhihu.com/p/45102654)

### 了解 http 长连接吗？

参考博文：

[HTTP 长连接和短链接 - 博客园](https://www.cnblogs.com/0201zcr/p/4694945.html)

### HTTP 的常见请求头？

参考博文：

[关于常用的 http 请求头以及响应头详解 - 掘金](https://juejin.cn/post/6844903745004765198)

### cookie 的写入？

服务器通过发送一个名为 Set-Cookie 的 HTTP 头来创建一个 cookie，作为 Response Headers 的一部分。每个 Set-Cookie 表示一个 cookie（如果有多个 cookie,需写多个 Set-Cookie），每个属性也是以名/值对的形式（除了 secure），属性间以分号加空格隔开。格式如下：

```
Set-Cookie: name=value[; expires=GMTDate][; domain=domain][; path=path][; secure]
```

参考博文：

[常用的本地存储——cookie 篇](https://segmentfault.com/a/1190000004743454)

## Linux 相关

## 使用 kill 命令杀死进程的原理是什么？

kill 命令会向指定进程号的进程发送信号，其可发送的信号列表如下：

> ❯ kill -l
>
> HUP INT QUIT ILL TRAP ABRT IOT BUS FPE KILL USR1 SEGV USR2 PIPE ALRM
>
> TERM STKFLT CHLD CLD CONT STOP TSTP TTIN TTOU URG XCPU XFSZ VTALRM PROF
>
> WINCH IO POLL PWR SYS RT\<N> RTMIN+\<N> RTMAX-\<N>

### 有了解网络相关的 Linux 命令吗？

参考博文：

[Linux 常用网络命令 - 博客园](https://www.cnblogs.com/klb561/p/9074278.html)

## Golang 相关

### Golang 垃圾回收从根节点开始扫描对象，根结点指的是什么？

参考自博文：[golang 垃圾回收 - 知乎](https://zhuanlan.zhihu.com/p/384900580)

> root set：根对象，垃圾回收器最先检查的对象。包括程序在编译期就能确定的那些存在于程序整个生命周期的变量、每个 goroutine 执行栈上的变量及指向堆区的指针、执行过程中指向堆内存的某些指针。

### Golang 并发控制可以使用 WaitGroup 和 Channel，两者有什么区别？

深入到底层实现，WaitGroup 的本质是信号量，对信号量的修改使用的是`atomic`内的原子操作。而 channel 本身是一个线程安全的环形队列，其内部的生产、消费过程是需要加锁的。即：**WaitGroup 无锁，channel 有锁。**（个人理解）

### Golang 的 channel 为什么会使用环形队列？

没有找到标准答案，参考自博文[图解 Go 的 channel 底层实现 - 菜刚 RyuGou 的博客](https://i6448038.github.io/2019/04/11/go-channel/)的内容：

> 至于为什么 channel 会使用循环链表作为缓存结构，我个人认为是在缓存列表在动态的 send 和 recv 过程中，定位当前 send 或者 recvx 的位置、选择 send 的和 recvx 的位置比较方便吧，只要顺着链表顺序一直旋转操作就好。

博主个人理解与其相同。

### Golang 垃圾回收是在什么时候 STW 的？

参考自博文：

[Go: How Does the Garbage Collector Mark the Memory? - Medium](https://medium.com/a-journey-with-go/go-how-does-the-garbage-collector-mark-the-memory-72cfc12c6976)

> To tackle that potential issue, an algorithm of write barrier is implemented and will allow Go to track any pointer changes. The only condition to enable write barriers is to stop the program for a short time, also called “Stop the World”:

即执行标记前需要打开写屏障，此时需要 STW。

![](https://rmt.ladydaily.com/fetch/allwens-work/storage/1_T16GKkEkxfswmCiHTNpwhQ.png?w=1280)
