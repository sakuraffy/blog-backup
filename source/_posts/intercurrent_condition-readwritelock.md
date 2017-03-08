---
title: 并发编程（十一）Condition与ReadWriteLock
date: 2016-07-15 16:20:44
tags:
	- 并发编程
---

### Condition

我们前面说到ReentrantLock可以替代synchronized，那么JDK提供了wait()和notify()的替代方式吗？
答案是肯定的。那就是Condition，但是值得一说的一点是Condition依赖于ReentrantLock
下面我们看一下Condition
``` java
	public interface Condition {
		void await() throws InterruptedException;
		void awaitUninterruptibly();
		boolean await(long time, TimeUnit unit) throws InterruptedException;
		void signal();
		void signalAll();
	}
```

<!--more-->

- await() 与 wait() 对应
- signal() 与 notify() 对应
- signalAll() 与 notifyAll() 对应
那么还有几个方法是干什么用的呢？
- awaitUninterruptibly() 不响应等待中的中断
- boolean await(long time, TimeUnit unit) 等待规定时间，无论是否被唤醒都继续执行

``` java
	public class TestCondition implements Runnable {
		private static ReentrantLock lock = new ReentrantLock();
		private static Condition condition = lock.newCondition();
		
		@Override
		public void run() {
			try {
				lock.lock();
				condition.await();
				System.out.println("thread is going on");
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}finally {
				lock.unlock();
			}
		}
		
		public static void main(String[] args) throws InterruptedException {                  
			Thread t = new Thread(new TestCondition());
			t.start();
			Thread.sleep(1000);
			lock.lock();
			condition.signal();
			lock.unlock();
		}
	}
```
输出结果 ：
``` java
thread is going on
```

### ReadWriteLock(读写锁)

为了提高系统性能，从JDK5后提供ReadWriteLock将数据读写分离。读与写的阻塞状态如下 ：
- 读与读不互斥，读读之间不阻塞
- 读与写互斥，读写互相堵塞
- 写与写互斥，写写阻塞
