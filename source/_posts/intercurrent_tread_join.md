---
title: 并发编程（二）线程的结束与谦让
date: 2016-06-05 22:10:00
tags:
	- 并发编程
---

### 线程结束

有时候，一个线程的输入需要依赖另外一个或多个线程的输出，这时候我们就很容易想到Thread.join()
``` java
	 public final void join() throws InterruptedException;
	 public final synchronized void join(long millis, int nanos) throws InterruptedException
```

<!--more-->

这是java为我们提供两个重载的join()。有人可能说join不是加入的意思，怎么又会变成结束呢？join()就是将自己合并到另外一个线程中，对于自己而言，可不就是就结束了吗。
再说说这两个方法吧。前者会一直等待，阻塞当前进程。而后者在等待时间内还未完成，则不管了，继续向下执行
``` java
public class JoinThread extends Thread {
	private volatile static int i = 0;
	@Override
	public void run() {
		for(; i < 100; i++) ;
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		for(; i < 200; i++) ;
	}

	public static void main(String[] args) throws InterruptedException {                  
		JoinThread jt = new JoinThread();
		jt.start();
		//jt.join();   
		jt.join(100);
		System.out.println(i);
	}
}

```
输出结果 ： 
``` java
jt.join() 200
jt.join(100) 100
```

其实从本质说join()是调用线程的wait()在当前线程对象上，下面是JDK中join()实现的核心代码片段：
``` java
	 while (isAlive()) {
        wait(0);
     }
```
可以看到，它让调用线程在当前线程对象上等待，当线程执行完成后，被等待的线程在退出前调用notifyAll()通知所有的等待线程继续执行。因此，我们要注意的是，在应用程序中尽量不要在Thread对象实例上使用类似wait()和notify()。		

### 线程谦让

Thread.yield的定义如下：
``` java
	public static native void yield();
```
它是个静态方法，一旦执行就会是当前线程让出CPU，但也不代表让出后，下一个执行的不是自己，更不代表以后就不会执行
``` java 
	public class YieldThread implements Runnable {
		@Override
		public void run() {
			while(true) {
				 System.out.println("HelloWorld");
				 Thread.yield();
			}
		}
		
		public static void main(String[] args) throws InterruptedException {
			Thread t = new Thread(new YieldThread());
			t.start();
			while(true) {
				System.out.println("nihao");
			}
		}
	}
```
截取部分输出 ：
``` java
nihao  
HelloWorld
HelloWorld
nihao
```
