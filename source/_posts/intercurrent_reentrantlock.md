---
title: 并发编程（十）重入锁
date: 2016-07-05 14:04:45
tags:
	- 并发编程
---
同步控制是并发编程的重要手段，前面讲到的一直都是使用synchronized，同样它也有一个替代品，那就是java.util.concurrent.locks.ReentrantLock
关于两者的性能，在JDK 5.0前，ReentranLock明显好于synchronized。但从JDK 6.0后对synchronized做了较大的优化，使得两者性能差不多  <!--more-->

``` java
	public class TestReentrantLock implements Runnable {

		private static int i = 0;
		private static ReentrantLock lock = new ReentrantLock();
		
		@Override
		public void run() {
			for(int k = 0; k < 10000; k++) {
				lock.lock();
				lock.lock();
				try {
					i++;
				}finally {
					lock.unlock();
					lock.unlock();
				}
			}
		}
		
		public static void main(String[] args) throws InterruptedException {                  
			Thread t1 = new Thread(new TestReentrantLock());
			Thread t2 = new Thread(new TestReentrantLock());
			t1.start();
			t2.start();
			t1.join();
			t2.join();
			System.out.println(i);
		}
	}
```
多次输出结果都是 20000 可以看出
- 可以看出ReentrantLock锁定的方式是对类对象实例和类同时锁定
- 使用ReentrantLock，由程序员手动控制何时加锁，释放锁。但一定要注意： 一定要释放锁
- ReentrantLock支持锁中锁，但一定要注意释放锁的次数

### 中断响应

除了上面所展示的，ReentrantLock还提供中断处理的能力
我们知道对于synchronized来说，如果一个线程等待锁，那么就有两种结果，要么获得，要么等待。但是ReentrantLock提供了第三种可能，那就是线程可以被中断。线程可以根据需要取消对锁的请求。下面展示一个用ReentrantLock解决死锁问题的小程序
``` java
	public class TestReentrantLock implements Runnable {
		private static ReentrantLock lock1 = new ReentrantLock();
		private static ReentrantLock lock2 = new ReentrantLock();
		
		private int lock;
		//控制加锁顺序，方便形成死锁
		public TestReentrantLock(int lock) {
			this.lock = lock;
		}
		
		@Override
		public void run() {
			try {
				if(lock == 1) {
					lock1.lockInterruptibly();
					try {
						Thread.sleep(1000);
					}catch (InterruptedException e) {
						e.printStackTrace();
						lock2.lockInterruptibly();
					}
				} else{
					lock2.lockInterruptibly();
					try {
						Thread.sleep(1000);
					}catch (InterruptedException e) {
						e.printStackTrace();
						lock1.lockInterruptibly();
					}
				}
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}finally {
				if(lock1.isHeldByCurrentThread()) {
					lock1.unlock();
				}
				if (lock2.isHeldByCurrentThread()) {
					lock2.unlock();
				}
				System.out.println(Thread.currentThread().getName() + ": exit");      
			}
		}

		public static void main(String[] args) throws InterruptedException {
			Thread t1 = new Thread(new TestReentrantLock(1));
			Thread t2 = new Thread(new TestReentrantLock(2));
			t1.start();
			t2.start();
			Thread.sleep(1000);
			t1.interrupt();
		}
	}
```
输出结果
``` java
Thread-0: exit
Thread-1: exit
```
值得注意的：
- 不要将所有的Exception都放在一个try里面，这样虽然简洁，但是捕捉到Exception就不会执行其它，掩饰了其它错误
- 这里用的是ReentrantLock.lockInterruptibly()，而不是ReentrantLock.lock()
- finnally里面一定要记得检查是否还存在锁再去释放，否则会抛java.lang.IllegalMonitorStateException

### 申请等待限时

ReentrantLock避免死锁，还有另一种方式，那就是限时等待。也就是说在规定时间内获取不到锁，就放弃
``` java
	public boolean tryLock()
	public boolean tryLock(long timeout, TimeUnit unit)
```
上面是JDK提供的两个重载方法。不带参数的，当前线程会尝试获取，成功返回true，失败，不等待，立即返回false。带参数就是在获取失败不立即返回，而是在规定时间内多次获取，当时间到了并还是失败就返回false
``` java
	public class TestReenrantLock implements Runnable {
		private static ReentrantLock lock1 = new ReentrantLock();
		private static ReentrantLock lock2 = new ReentrantLock();
		
		private int lock;
		
		public TestReenrantLock(int lock) {
			this.lock = lock;
		}
		
		@Override
		public void run() {
			if(lock == 1) {
				while(true) {
					if(lock1.tryLock()) {
						try {
							try {
								Thread.sleep(1000);
							} catch (InterruptedException e) {
								// TODO Auto-generated catch block
								e.printStackTrace();
							}
							if(lock2.tryLock()) {
								try {
									System.out.println(Thread.currentThread().getName()
											+ ": work done");
									return ;
								}finally {
									lock2.unlock();
								}
							}
						}finally {
							lock1.unlock();
						}
					}
				}
			}else {
				while(true) {
					if(lock2.tryLock()) {
						try {
							try {
								Thread.sleep(1000);
							} catch (InterruptedException e) {
								// TODO Auto-generated catch block
								e.printStackTrace();
							}
							if(lock1.tryLock()) {
								try {
									System.out.println(Thread.currentThread().getName()
											+ ": work done");
									return ;
								}finally {
									lock1.unlock();
								}
							}
						}finally {
							lock2.unlock();
						}
					}
				}
			}
		}	
					
		
		public static void main(String[] args) {
			Thread t1 = new Thread(new TestReenrantLock(1));
			Thread t2 = new Thread(new TestReenrantLock(2));
			t1.start();
			t2.start();
		}
	}

```
输出结果 ：
``` java
Thread-0: work done
Thread-1: work done
```

### 公平锁
``` java
	private static ReentrantLock lock1 = new ReentrantLock()
	public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```
在计算机中，一般公平就是指先来先服务。公平锁就是谁先申请就给谁使用
在这个弱肉强食，适者生存的社会，公平就有点显得……，再说维系公平需要太大的消耗，会降低性能。除特殊情况不要使用

