---
title: 并发编程（九）并发下ArrayList和HashMap的bug
date: 2016-07-03 16:09:21
tags:
	- 并发编程
---
### ArrayList的Bug

我们知道ArrayList是线程不安全的，在并发编程中使用ArrayList可能会导致程序出错
``` java 
	public class ArrayListBug {
		private static ArrayList<Integer> al = new ArrayList<>(10);
		private static Vector<Integer> v = new Vector<>(10);

		public static class BugThread extends Thread {
			@Override
			public void run() {
				for(int i = 0; i < 100000; i++) {
					al.add(i);
					v.add(i);
				}
			}
		}
		
		public static void main(String[] args) throws InterruptedException {                  
			Thread t1 = new BugThread();
			Thread t2 = new BugThread();
			t1.start();
			t2.start();
			t1.join();
			t2.join();
			System.out.println("al.size() : " + al.size());
			System.out.println("v.size() : " + v.size());
		}
	}
```
运行上面的程序，我们可能得到三种结果

<!--more-->

- 恭喜你，得到了正确的答案
al.size() : 0
v.size() : 200000

- 也恭喜你，抛出了java.lang.ArrayIndexOutOfBoundsException
为什么也值得恭喜呢？对于程序员来说找到bug其实就是解决了问题

- 恭喜你，进入抓狂的状态，因为不报错，但结果就是不对
``` java
al.size() : 134508
v.size() : 200000
```

这是为什么呢？
因为两个线程同时对容器操作，那么它就有可能产生错误。不仅如此，当容器遍历的时候修改数据也会将数据写坏。

怎么解决呢？
你也看到了Vector则不会产生问题，在并发编程中可以用Vector代替ArrayList

### HashMap的Bug

``` java
	public class HashMapBug {
		private static HashMap<String,Integer> map = new HashMap<>();
		
		public static class BugThread extends Thread {
			private int start = 0;
			public BugThread(int start) {
				this.start = start;
			}
			
			@Override
			public void run() {
				for(int i = start; i < 100000; i += 2) {
					map.put(Integer.toString(i), i);
				}
			}
			
		}
		
		public static void main(String[] args) throws InterruptedException {                  
			Thread t1 = new BugThread(0);
			Thread t2 = new BugThread(1);
			t1.start();
			t2.start();
			t1.join();
			t2.join();
			System.out.println(map.size());
		}
	}
```
运行上面的程序，我们可能得到三种结果
- 100000
- 99867
- 程序陷入死循环

至于前两种和ArrayList一样。产生死循环的原因又是什么呢？
我们知道HashMap的键具有唯一性，那么每次加入，它都会以链表的形式遍历一次，在并发编程的时候，那么它就可能破坏链表结构，使其变成环，从而造成死循环。这里还要说的一点就是在JDK8中不存在这个问题，对其进行了优化，但是会抛出java.lang.ClassCastException

解决方案
使用线程安全的ConcurrentHashMap代替HashMap

