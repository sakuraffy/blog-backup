---
title: 并发编程（六）volatile与synchronized
date: 2016-06-17 15:32:07
tags:
	- 并发编程
---
### volatile

当32位机器多线程多long类型数操作时，会出现“写坏”，但是long前加上volatile关键字则没有问题。可是volatile是万能的吗？
``` java
	public class Volatile extends Thread {
		private static volatile int i = 0;

		@Override
		public void run() {
			for(int k = 0; k < 10000; k++) {
				i++;
			}
		}
			
		public static void main(String[] args) throws InterruptedException {                  
			Thread[] threads = new Thread[10];
			for(int i = 0; i < 10; i++) {
				threads[i] = new Volatile();
				threads[i].start();
			}
			for(int i = 0; i < 10; i++) {
				threads[i].join();
			}
			System.out.println(i);
		}
	}
```

输出结果 ： 	<!--more-->

``` java
37989
```

为什么会这样呢？volatile不是原子性的吗？
volatile不能代替synchronized，它也无法保证符合操作的原子性。

### synchronized

synchronized可以有效的保证线程安全
``` java
	public class Synchronized implements Runnable {
		private static int i = 0;
		
		public synchronized void increase() {
			for(int k = 0; k < 10000; k++) {
				i++;
			}
		}
		
		public void run() {
			increase();
		}
		
		public static void main(String[] args) throws InterruptedException {                  
			Thread t1 = new Thread(new Synchronized());
			Thread t2 = new Thread(new Synchronized());
			t1.start();
			t2.start();
			t1.join();
			t2.join();
			System.out.println(i);
		}
	}
```
输出结果 ：
``` java
10821
```

为什么不是20000呢？
这里我们需要注意的synchronized有两种出现方式
- 直接加在函数前面
- 使用synchronized语句块

> 锁定的对象，也就是获取谁的监视器

- 获取当前类的实例对象的监视器
``` java 
	public synchronized void increase() {
		i++;
	}
```
与下面的等价
``` java
	Synchronized s = new Synchronized();
	public void increase() {
		synchronized(s) {
			i++;
		}
	}
```
也与下面的等价
``` java
	public void increase() {
		synchronized(this) {
			i++;
		}
	}
```

- 获取当前类的监视器
``` java
	public static synchronized void increase() {
		i++;
	}
```
与下面的等价
``` java
	public void increase() {
		synchronized(Synchronized.class) {
			i++;
		}
	}
```