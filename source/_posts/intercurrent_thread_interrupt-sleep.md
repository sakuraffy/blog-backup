---
title: 并发编程（四）线程的休眠与中断
date: 2016-06-11 22:58:12
tags:
	- 并发编程
---

### 线程休眠

``` java
	public static native void sleep(long millis) throws InterruptedException;
```

### 线程中断

前面我们说到了线程销毁stop(),但由于会释放所有资源，引起线程安全问题而被废弃。那么在JDK中是有一套强大的支持能避免这个问题呢？答案是肯定的，那就是线程中断。
严格地说，线程中断并没有直接退出，而是给目标线程发送通知，至于目标线程怎么处理，则是目标线程的事了。
java中与线程中断有关的主要是以下三个方法 ：

<!--more-->

``` java
	public void interrupt()  //设置中断标志位
	public boolean isInterrupted() //检查是否中断
	public static boolean interrupted() //检查是否中断，并清除终端状态
```

下面的代码对t1进行中断，那么中断后，t1就真的会停止吗？

``` java
	public class InterruptThread {
		public static void main(String[] args) throws InterruptedException {
			Thread t = new Thread() {
				@Override
				public void run() {
					while(true) {
						try {
							Thread.sleep(200);
						} catch (InterruptedException e) {
							System.out.println("Interrupted when sleeping");
						}
						Thread.yield();
					}
				}
			};
			
			t.start();
			Thread.sleep(200);
			t.interrupt();
		}
	}
```
答案是否定的。虽然t1设置了中断位置，却没有什么作用。那么应该怎么做呢？
有两种实现方法。
- 捕获到异常时直接退出。
- 如下程序

``` java
	public class InterruptThread {
		public static void main(String[] args) throws InterruptedException {
			Thread t1 = new Thread() {
				@Override
				public void run() {
					while(true) {
						if(Thread.currentThread().isInterrupted()) {
							System.out.println("Interrupted !");
							break;
						}
						try {
							Thread.sleep(200);
						} catch (InterruptedException e) {
							System.out.println("Interrupted when sleeping");
							Thread.currentThread().interrupt();
						}
						Thread.yield();
					}
				}
			};
			
			t1.start();
			Thread.sleep(200);
			t1.interrupt();
		}
	}

```
Thread.sleep()因中断而抛出异常，异常会清楚中断标志位，所以必须重新设置中断标志位。