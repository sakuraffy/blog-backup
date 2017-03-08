---
title: 并发编程（三）线程的等待与通知
date: 2016-06-08 21:45:25
tags:
	- 并发编程
---
为了支持多线程之间的通讯，JDK提供了wait()和notify()两个重要的方法。但请注意这两个方法都不是Thread类所拥有的，而是Object类所提供的。这就意味着所有的对象都可以调用这个方法。
``` java 
	public final void wait() throws InterruptedException
	public final native void notify()
```

<!--more-->

当对象调用wait()后，当前线程则会停止继续执行，转换为等待状态，直到有对象将其唤醒，此线程才继续向下执行
当对象调用notify()后，它就会进入Object等待队列随机唤醒一个线程，注意这里不是唤醒优先级高的，更不是唤醒先来的。
notify()相似的方法notifyAll()，它是将Object等待队列中所有的线程全部唤醒。

Object.wait()和Object.notify()并不是所有地方都可以调用，它必须包含在对应的synchronized语句中，否则会抛出java.lang.IllegalMonitorStateException。

当notify()将其它线程唤醒，被唤醒的线程也不能立即向下执行，它必须获取Object的监视器才能向下执行。下面我就给出一个小小的示例 ：
``` java
	public class ThreadWN {
		private static Object obj = new Object();
		
		public static class WaitThread extends Thread {
			@Override
			public void run() {
				synchronized (obj) {
					System.out.println(System.currentTimeMillis() +
					" WaitThread start");
					try {
						obj.wait();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					System.out.println(System.currentTimeMillis() +               
					" WaitThread end");
				}
			}
		}
		
		public static class NotifyThread extends Thread {
			@Override
			public void run() {
				synchronized (obj) {
					System.out.println(System.currentTimeMillis() + 
					" NotifyThread start");
					obj.notify();
					System.out.println(System.currentTimeMillis() + 
					" NotifyThread end");
					try {
						Thread.sleep(2000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		}
		
		public static void main(String[] args) {
			WaitThread wt = new WaitThread();
			wt.start();
			NotifyThread nt = new NotifyThread();
			nt.start();
		}	
	}
```
输出结果如下 ：
``` java
1469168734181 WaitThread start
1469168734181 NotifyThread start
1469168734181 NotifyThread end
1469168736183 WaitThread end
```
