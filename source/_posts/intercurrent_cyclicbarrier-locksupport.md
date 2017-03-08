---
title: 并发编程（十三）CyclicBarrier与LockSupport
date: 2016-07-18 00:10:11
tags:
	- 并发编程
---
### CyclicBarrier（循环栅栏）

CyclicBarrier与CountDownLatch非常相似，它也可以实现线程的计数等待，但它比CountDownLatch更加的强大。它可以多次利用，也就是说当计数为0后又变成和开始一样。
下面展示一个测试案例 ：

<!--more-->

``` java
	public class TestCyclicBarrier {
		public static class BarrierThread implements Runnable {		
			private static CyclicBarrier cb = new CyclicBarrier(5);
			private boolean flag = true;
			
			@Override
			public void run() {
				try {
					doWork(flag);
					cb.await();
					flag = false;
					doWork(flag);
					cb.await();
				} catch (InterruptedException | BrokenBarrierException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			
			private void doWork(boolean flag) throws InterruptedException {
				Thread.sleep(1000 * new Random().nextInt(5));
				if (flag) {
					System.out.println(System.currentTimeMillis()/1000 + " "
							+ Thread.currentThread().getName() + " ready");	
				}else {
					System.out.println(System.currentTimeMillis()/1000 + " "
							+ Thread.currentThread().getName() + " finsh");	
				}
			}
			
		}
		
		public static void main(String[] args) {
			BarrierThread barrier = new BarrierThread();
			for(int i = 0; i < 5; i++) {
				new Thread(barrier).start();
			}
		}
	}
```
输出结果 ：
``` java
1469513719 Thread-4 ready
1469513719 Thread-3 ready
1469513719 Thread-2 ready
1469513720 Thread-1 ready
1469513720 Thread-0 ready
1469513720 Thread-3 finsh
1469513721 Thread-2 finsh
1469513722 Thread-1 finsh
1469513723 Thread-0 finsh
1469513724 Thread-4 finsh
```

注意事项
CyclicBarrier.await()会抛出两个异常，除了InterruptedException，还有就是BrokenBarrierException，抛出这个异常，说明此时的栅栏已经被破坏

### LockSupport

前面我们知道suspend()可能会使程序产生线程安全问题，那么JDK有没有更好的代替方案呢？
答案是肯定的，那么就由LockSupport出场了
与Object.wait()相比，LockSupport不需要获取对象的锁，更不会抛出InterruptedException。其内部实现使用了类似信号量的机制从而保证了线程安全
``` java
	public static void unpark(Thread thread)
	public static void park()
```
这是LockSupport两个常用的方法
下面展示一个关于中断处理的测试程序
``` java
	public class TestLockSupport {
		private static Object object = new Object();
		
		public static class ChangObjectThread extends Thread {
			public ChangObjectThread(String name) {
				super.setName(name);
			}
			
			@Override
			public void run() {
				synchronized (object) {
					System.out.println("in " + getName());
					LockSupport.park();
					if (interrupted()) {
						System.out.println(getName() + " interrupted");      
					}
					System.out.println(getName() + " exit");
				}
			}
		}
		
		public static void main(String[] args) {
			ChangObjectThread t1 = new ChangObjectThread("t1");
			ChangObjectThread t2 = new ChangObjectThread("t2");
			t1.start();
			t2.start();
			t1.interrupt();
			LockSupport.unpark(t2);
		}
	}
```
输出结果 ：
``` java
in t1
t1 interrupted
t1 exit
in t2
t2 exit
```
