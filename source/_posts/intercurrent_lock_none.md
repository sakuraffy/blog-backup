---
title: 并发编程(十九)无锁
date: 2016-07-29 20:37:50
tags:
	- 并发编程
---

就性格而言，有乐观也有悲观，对于并发控制而言，同样也是如此，锁就是一种悲观策略，它假设每一个临界资源的争夺都会产生冲突，因此它宁可牺牲让线程等待。而无锁就是一种乐观策略，它假设每次争取临界资源都不会冲突，那遇到冲突怎么办呢？它采用的是原子化CAS（Compare And Swap）的技术来有效地检查冲突，CAS算法的过程是这样的：它包含三个参数CAS（V,E,N）,v表示要更改的变量，e表示预期值，n表示新值，只有e和n相同才能修改成功

<!--more-->

### AtomicInteger

先展示一下示例 ：
``` java
	public class TestAtomicInteger {
		private static AtomicInteger ai = new AtomicInteger(0);
		
		public static class Add extends Thread {
			@Override
			public void run() {
				for(int k = 0; k < 10000; k++) {
					ai.incrementAndGet();
				}
			}
		}
		
		public static void main(String[] args) throws InterruptedException {                  
			Thread[] threads = new Thread[10];
			for(int i = 0; i < 10; i++) {
				Thread t = new Add();
				threads[i] = t;
				t.start();
			}
			for(int i = 0; i < 10; i++) {
				threads[i].join();
			}
			System.out.println(ai.get());
		}
	}
```
输出结果 ：
``` java
100000
```

由此可见，使用原子性的AtomicInteger是线程安全的，下面来看看看AtomicInteger的源码实现
``` java
	public class AtomicInteger extends Number implements java.io.Serializable {
		private volatile int value;
		private static final long valueOffset;
		public final int get() {
			return value;
		}
		public int intValue() {
			return get();
		}
		public final int incrementAndGet() {
			return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
		}
		public final boolean compareAndSet(int expect, int update) {}
	}
```
上面看到AtomicInteger操作都是关于Unsafe，那下面就看看它的神秘，Unsafe是java里面的指针，但是它却无法被应用程序所使用，是一个JDK专用类

### AtomicReference

上面我们讲述了基本数据类型原子性操作，那么对于引用类型，AtomicReference就可以满足，但是AtomicReference有个小Bug，就是当数据修改，然后再修改至原值，此时，它就无能为力了
下面就展示下这个Bug，假设一下业务流程 ：你的购物卡里面金额少于100时，公司会为你冲200，但就是一次
``` java
	public class TestAtomicReference {
		private static AtomicReference<Integer> ar = new AtomicReference<>(250);              
		public static class RechargeThread extends Thread {
			@Override
			public void run() {
				while(true) {
					while(true) {
						Integer value = ar.get();
						if(value < 100) {
							if (ar.compareAndSet(value, value + 200)) {
								System.out.println("recharge 200," 
								+ "remaining ：" + ar.get());
								break;	
							}
						}else {
							break;
						}
					}
				}
			}
		}
		
		public static class ConsumeThread extends Thread {
			@Override
			public void run() {
				for(int i = 0; i < 10; i++) {
					while(true) {
						Integer value = ar.get();
						if(value > 100) {
							if(ar.compareAndSet(value, value - 100)) {
								System.out.println("consume 100,"
								+ "remaining : " + ar.get());
								break;
							}
						}else {
							System.out.println("consume fail");
							break;
						}
					}
					try {
						Thread.sleep(100);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		}
		
		public static void main(String[] args) {
			for(int i = 0; i < 10; i++) {
				RechargeThread rt = new RechargeThread();
				rt.start();
			}
			ConsumeThread ct = new ConsumeThread();
			ct.start();
		}
	}
```
输出结果 ：
``` java
consume 100,remaining ： 150
consume 100,remaining ： 50
recharge 200,remaining ： 250
consume 100,remaining ： 150
consume 100,remaining ： 50
recharge 200,remaining ： 250
consume 100,remaining ： 150
consume 100,remaining ： 250
recharge 200,remaining ： 250
consume 100,remaining ： 150
```
### AtomicStampedReference

既然有这个Bug，那么JDK有没有解决的方案呢？答案是肯定的，那就是AtomicStampedReference
``` java
	public class TestAtomicStampedReference {
		private static AtomicStampedReference<Integer> asr = new AtomicStampedReference<Integer>(50, 0);
		
		public static class RechargeThread extends Thread {
			private int stamp = asr.getStamp();
			@Override
			public void run() {
				while(true) {
					while(true) {
						Integer value = asr.getReference();
						if(value < 100) {
							if (asr.compareAndSet(value, value
								+ 200,stamp,stamp + 1)) {
								System.out.println("recharge 200," +
								"remaining ：" + asr.getReference());
								break;	
							}
						}else {
							break;
						}
					}
				}
			}
		}
		
		public static class ConsumeThread extends Thread {
			@Override
			public void run() {
				for(int i = 0; i < 10; i++) {
					while(true) {
						Integer value = asr.getReference();
						int stamp = asr.getStamp();
						if(value > 100) {
							if(asr.compareAndSet(value, value 
								- 100,stamp,stamp + 1)) {
								System.out.println("consume 100," + 
								"remaining ： "  + asr.getReference());
								break;
							}
						}else {
							System.out.println("consume fail");
							break;
						}
					}
					try {
						Thread.sleep(100);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		}
		
		public static void main(String[] args) {
			for(int i = 0; i < 10; i++) {
				RechargeThread rt = new RechargeThread();
				rt.start();
			}
			ConsumeThread ct = new ConsumeThread();
			ct.start();
		}
	}
```
输出结果 ：
``` java
recharge 200,remaining ： 250
consume 100,remaining ： 150
consume 100,remaining ： 50
consume fail
consume fail
consume fail
consume fail
consume fail
consume fail
```

### AtomicIntegerArray

同样的，数组也可以原子化，下面看看JDK源码
``` java
	 public final int get(int i) {}
	 public final int getAndSet(int i, int newValue) {}
	 public final boolean compareAndSet(int i, int expect, int update) {}
	 public final int getAndIncrement(int i) {}
	 public final int getAndDecrement(int i) {}
```
下面是一个示例
``` java
	public class TestAtomicArray {
		public static void main(String[] args) throws InterruptedException {                  
			AtomicIntegerArray aia = new AtomicIntegerArray(10);
			Thread[] threads = new Thread[10];
			for(int i = 0; i < 10; i++) {
				Thread t = new Thread() {
					@Override
					public void run() {
						for(int k = 0; k < 1000; k++) {
							aia.incrementAndGet(k % 10);
						}
					}
				};
				threads[i] = t;
				t.start();
			}
			for(int i = 0; i < 10; i++) {
				threads[i].join();
			}
			System.out.println(aia);
		}
	}
```
输出结果 ：
``` java
[1000, 1000, 1000, 1000, 1000, 1000, 1000, 1000, 1000, 1000]
```

### AtomicIntegerFieldUpdater
有时候，在设计前期，由于考虑不周或后期改变，一些变量可能需要线程安全，那么就可以使用AtomicIntegerFieldUpdater，下面就展示个示例
``` java
	public class TestAtomicIntegerFieldUpdater {
		public static class User {
			volatile int score;
		}
		
		public static final AtomicIntegerFieldUpdater<User> scoreUpdater = 
				AtomicIntegerFieldUpdater.newUpdater(User.class, "score");            
		
		//用来验证AtomicIntegerFieldUpdater的正确性
		public static AtomicInteger ai = new AtomicInteger(0);
		
		public static void main(String[] args) throws InterruptedException {
			User user = new User();
			Random r = new Random();
			Thread[] threads = new Thread[100];
			for(int i = 0; i < 100; i++) {
				threads[i]  = new Thread() {
					@Override
					public void run() {
						if(r.nextInt(5) > 2) {
							ai.incrementAndGet();
							scoreUpdater.getAndIncrement(user);
						}
					}
				};
				threads[i].start();
			}
			for(int i = 0; i < 100; i++) {
				threads[i].join();
			}
			System.out.println("score : " + user.score);
			System.out.println("ai : " + ai);
		}
	}
```
输出结果 ：
``` java
score : 35
ai : 35
```

需要注意的几点 ：
- 属性是通过反射获得，所以不能用private修饰
- 为了确保读取的正确性，属性要用volatile修饰
- 由于CAS是通过对象来完成，所以属性不能用static修饰