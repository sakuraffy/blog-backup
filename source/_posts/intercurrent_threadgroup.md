---
title: 并发编程（八）线程组
date: 2016-06-28 15:01:52
tags:
	- 并发编程
---
线程组顾名思义就是将相同功能的线程放在一起。
### 构造函数
``` java 
	public ThreadGroup(String name)
	public ThreadGroup(ThreadGroup parent, String name)

	private ThreadGroup() {     // called from C code
        this.name = "system";
        this.maxPriority = Thread.MAX_PRIORITY;
        this.parent = null;
    }
```
	
从上面可以看出不添加线程组的名称，则会默认看作system  <!--more-->
``` java 
	public class GroupThread {
	
		public static class HelloWorldPrint implements Runnable {
			@Override
			public void run() {
				synchronized (this) {
					System.out.println("HelloWorld");
					System.out.println(Thread.currentThread().getThreadGroup().getName() + " "
							+ Thread.currentThread().getName());
				}
			}
		}
		
		public static class NiHaoPrint implements Runnable {
			@Override
			public void run() {
				synchronized (this) {
					System.out.println("Nihao");
					System.out.println(Thread.currentThread().getThreadGroup().getName() + " "
							+ Thread.currentThread().getName());
				}
			}
		}
		
		public static void main(String[] args) {
			ThreadGroup tg= new ThreadGroup("print");
			Thread t1 = new Thread(tg,new HelloWorldPrint(),"helloWorld");
			Thread t2 = new Thread(tg,new NiHaoPrint(),"nihao");
			t1.start();
			t2.start();
		}
	}
```
输出结果 ：
``` java
Nihao
print nihao
HelloWorld
print helloWorld
```
