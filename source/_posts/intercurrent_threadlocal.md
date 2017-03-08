---
title: 并发编程(十八)ThreadLocal
date: 2016-07-28 20:35:12
tags:
	- 并发编程
---
ThreadLocal从名字就可以看出，这是个线程局部变量，也就是说只有当前线程可以访问，这也就不存在什么线程安全的问题

<!--more-->

``` java
	public class TestThreadLocal {
		private static ThreadLocal<Integer> tl = new ThreadLocal<>();
		public static class ThreadA {

			public void get() {
				int data = tl.get();
				System.out.println("A : "+ Thread.currentThread().getName() + data);
			}
		}
		
		public static class ThreadB {
			public void get() {
				int data = tl.get();
				System.out.println("B : "+ Thread.currentThread().getName() + data);
			}
		}
		
		public static void main(String[] args) {
			for(int i = 0; i < 2; i++) {
				new Thread(new Runnable() {
		
					@Override
					public void run() {
						int data = new Random().nextInt();
						System.out.println(Thread.currentThread().getName() + " put " + data);
						tl.set(data);
						new ThreadA().get();
						new ThreadB().get();
					}
				}).start();
			}
		}	
	}
```
输出结果 ：
``` java
Thread-0 put 897133741
Thread-1 put 1679577368
A : Thread-0897133741
A : Thread-11679577368
B : Thread-11679577368
B : Thread-0897133741
```


### ThreadLocal实现原理

``` java
	public class ThreadLocal<T> {
		static class ThreadLocalMap {}
		protected T initialValue() {
			return null;
		}

		public T get() {
			Thread t = Thread.currentThread();
			ThreadLocalMap map = getMap(t);
			if (map != null) {
				ThreadLocalMap.Entry e = map.getEntry(this);                          
				if (e != null) {
					@SuppressWarnings("unchecked")
					T result = (T)e.value;
					return result;
				}
			}
			return setInitialValue();
		}

		public void set(T value) {
			Thread t = Thread.currentThread();
			ThreadLocalMap map = getMap(t);
			if (map != null)
				map.set(this, value);
			else
				createMap(t, value);
		}
	}
	
```
从上面可以看出ThreadLocal.ThreadLocalMap是非常关键的存储单个数据只是存储Map的一个特例，这里要注意的是
- ThreadLocal.ThreadLocalMap类似于WeekHashMap,其中的Entry都是若引用，只要JVM检测到就会将其回收
- 使用线程池时，要注意ThreadLocal对象数据的清除
- 就单数据而言，可以用Map代替ThreadLocal
``` java
	public class TestThreadLocal {
		private static Map<Thread,Integer> map = new HashMap<>();
		public static class ThreadA {
			public void get() {
				int data = map.get(Thread.currentThread());
				System.out.println("A : "+ Thread.currentThread().getName() + " " + data);
			}
		}
		
		public static class ThreadB {
			public void get() {
				int data = map.get(Thread.currentThread());
				System.out.println("B : "+ Thread.currentThread().getName() + " " + data);
			}
		}
		
		public static void main(String[] args) {
			for(int i = 0; i < 2; i++) {
				new Thread(new Runnable() {
		
					@Override
					public void run() {
						int data = new Random().nextInt();
						System.out.println(Thread.currentThread().getName() + " put " + data);
						map.put(Thread.currentThread(), data);
						new ThreadA().get();
						new ThreadB().get();
					}
				}).start();
			}
		}
	}

```
输出结果 ：
``` java
Thread-1 put -1497402079
Thread-0 put 1606205752
A : Thread-0 1606205752
A : Thread-1 -1497402079
B : Thread-1 -1497402079
B : Thread-0 1606205752
```
