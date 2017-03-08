---
title: 并发编程（十二）Semaphore与CountDownLatch
date: 2016-07-17 16:23:17
tags:
	- 并发编程
---
### Semaphore（信号量）

通过前面，我们知道无论是ReentrantLock还是Synchronized，每一次都只允许一个线程访问一个资源，而有时候我们需要多个线程访问同一资源，这时候我们就可以使用信号量。
``` java
	public Semaphore(int permits)
	public Semaphore(int permits, boolean fair)
```

<!--more-->

通过上面的构造函数我们知道每次构造一个信号量必须指明其大小，也就是最多几个线程同时访问一个资源
关于Semaphore常用函数
``` java
	public void acquire() throws InterruptedException
	public void acquireUninterruptibly()
	public boolean tryAcquire()
	public boolean tryAcquire(long timeout, TimeUnit unit)
	public void release()
```
命名和前面重入锁差不多。acquire()获取准入，失败就线程等待。acquireUninterruptibly()不响应中断。tryAcquire()尝试获取准入，失败返回false。tryAcquire(long timeout, TimeUnit unit)，在规定时间内，尝试获取准入，失败就再次尝试直至时间结束且还未成功则返回false

下面演示一个简单的示例 ：
``` java
	public class TestSemaphone implements Runnable {
		private final Semaphore semaphore = new Semaphore(3);
		
		@Override
		public void run() {
			try {
				semaphore.acquire();
				Thread.sleep(2000);
				System.out.println(System.currentTimeMillis()/1000
						+ " " + Thread.currentThread().getName());            
				semaphore.release();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		
		public static void main(String[] args) {
			TestSemaphone test = new TestSemaphone();
			for(int i = 0; i < 20; i++) {
				new Thread(test).start();
			} 
		}
	}
```
- 输出结果
``` java
1469447531 Thread-0
1469447531 Thread-1
1469447531 Thread-2
1469447533 Thread-6
1469447533 Thread-4
1469447533 Thread-5
1469447535 Thread-10
1469447535 Thread-9
1469447535 Thread-8
1469447537 Thread-11
1469447537 Thread-14
1469447537 Thread-13
1469447539 Thread-3
1469447539 Thread-16
1469447539 Thread-15
1469447541 Thread-7
1469447541 Thread-12
1469447541 Thread-19
1469447543 Thread-18
1469447543 Thread-17
```

- 注意
	- 开启线程的方式要是实现Runnable接口，因为Thread这种方式targer = null
	- 多个线程的Runnable对象必须是同一个

### CountDownLatch（倒计时器）

与上面的信号量有点相似，信号量是指最多有多少个线程访问资源，倒计时器是指最少有多少个线程都满足时，才执行下一步，就像火箭发射一样，只有所有都检查就位时才能发射
CountDownLatch常用方法
``` java
	public void await() throws InterruptedException
	public boolean await(long timeout, TimeUnit unit)
	public void countDown()
	public long getCount()
```
- await() : 阻塞式等待
- await(long timeout, TimeUnit unit) : 过了规定时间，无论是否都准备完毕都继续向下执行
- countDown() 计数减少一个
- getCount() 距离最小个数还差多少

下面演示一个简单的示例 ：
``` java
	public class TestCountDownLatch implements Runnable {
		private static CountDownLatch cdl = new CountDownLatch(10);
		@Override
		public void run() {
			try {
				Thread.sleep(1000 * new Random().nextInt(5));
				System.out.println(System.currentTimeMillis() /1000 + " "            
					+ Thread.currentThread().getName() + " complete");
				cdl.countDown();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		
		public static void main(String[] args) throws InterruptedException {      
			TestCountDownLatch  test = new TestCountDownLatch();
			for(int i = 0; i < 10; i++) {
				new Thread(test).start();
			}
			cdl.await();
			System.out.println("fire");
		}
	}
```
输出结果 ：
``` java
1469451326 Thread-4 complete
1469451326 Thread-3 complete
1469451326 Thread-8 complete
1469451327 Thread-9 complete
1469451327 Thread-6 complete
1469451327 Thread-7 complete
1469451329 Thread-1 complete
1469451330 Thread-5 complete
1469451330 Thread-2 complete
1469451330 Thread-0 complete
fire
```