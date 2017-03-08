---
title: 并发编程（五）线程的挂起与线程恢复
date: 2016-06-14 14:30:18
tags:
	- 并发编程
---
### 什么是线程挂起

线程挂起就是就是使线程处于不可执行状态。那不可执行状态又有哪些呢？
- Thread.sleep(1000) 在这1000ms中处于不可执行状态
- Thread.join()
- Thread.wait()

<!--more-->

``` java
	@Deprecated
    public final void suspend()
```

### 什么是线程恢复

线程恢复就是就是使挂起线程再重新处于可执行状态。
``` java
	@Deprecated
    public final void resume()
```

### 为什么挂起和恢复会被废弃

线程挂起的时候，并不会释放资源，其他任何线程想要访问被他占有的锁都会收到牵连，导致程序不能正常运行。直到对应线程上进行resume()操作，被挂起的线程才能继续。有人说这没什么啊，只是多等待一会啊。
可是最关键的是一旦在resume()操作在suspend()前执行，那么挂起的县城很难有机会再执行，严重的话，它可能导致整个系统都崩溃。

### 解决方案

- 像之前stop()一样，增加一个标志性变量
- 使用wait()和notify()代替suspend()和resume()