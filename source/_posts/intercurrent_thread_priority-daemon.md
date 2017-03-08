---
title: 并发编程（七）守护线程与优先级
date: 2016-06-22 15:32:07
tags:
	- 并发编程
---
### 什么是守护线程

守护线程又被称为精灵线程，当所有的用户线程结束后，守护线程会自动退出，程序结束。
	
``` java
	public class DaemonThread extends Thread {
		@Override
		public void run() {
	
			while(true) {
				System.out.println("HelloWorld");
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
		
		public static void main(String[] args) throws InterruptedException {                  
			Thread t = new DaemonThread();
			t.start();
			t.setDaemon(true);
			Thread.sleep(1000);
			System.out.println("MainThread end");
		}
	}
```

### 守护线程应该注意什么 <!--more-->
	
- Thread.setDaemon(true)必须在Thread.start()之前，否则会抛出java.lang.IllegalThreadStateException
- 守护线程里面所产生的线程都是守护线程
- 不是所用应用都可以使用守护线程，例如文件读写。

### 线程优先级

线程的优先级从1到10，数字越大，优先级越高，获取CPU的概率越大，但不代表优先级高就一定要比优先级低的先执行
``` java
	public class PriorityThread {
		public static class HighPriority extends Thread {
			private static int i = 0;
			@Override
			public void run() {
				while(true) {
					synchronized (LowPriority.class) {
						if(i > 10000) {
							System.out.println("HighPriority complete");
							break;
						}
						i++;
					}
				}
			}
		}
		
		public static class LowPriority extends Thread {
			private static int i = 0;
			@Override
			public void run() {
				while(true) {
					synchronized (LowPriority.class) {
						if(i > 10000) {
							System.out.println("LowPriority complete");  
							break;
						}
						i++;
					}
				}
			}
		}
		
		public static void main(String[] args) {
			Thread t1 = new HighPriority();
			Thread t2 = new LowPriority();
			t1.setPriority(Thread.MAX_PRIORITY);
			t2.setPriority(Thread.MIN_PRIORITY);
			t1.start();
			t2.start();
		}
	}
```
输出结果 ：
``` java
HighPriority complete
LowPriority complete
```
