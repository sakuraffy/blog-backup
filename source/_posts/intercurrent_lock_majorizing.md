---
title: 并发编程(十七)锁的优化
date: 2016-07-26 20:33:50
tags:
	- 并发编程
---
多线程的应用其实不一定要比单线程效率高，就单线程应用来说CPU资源主要在任务上，而多线程不仅要作用于任务本身还要作用于线程之间的调度及其它。这就是单核在并行计算上性能要低于原始的串行计算
在并发编程中，我们常常会使用锁来保证线程安全，那我们又有哪些方法来对锁进行优化呢？
### 减少锁的持有时间

我们前面说synchroinzed有两种锁定方法，一是直接放在方法前，另外一个是使用语句块。为了提高效率，我们使用语句块将需要同步的放在里面就行了,下面JDK中Patte的一个体现

<!--more-->

``` java
	public Matcher matcher(CharSequence input) {
        if (!compiled) {
            synchronized(this) {
                if (!compiled)
                    compile();
            }
        }
        Matcher m = new Matcher(this, input);
        return m;
    }
```

### 减小锁的粒度

所谓减少锁粒度就是指缩小锁定对象的范围，从而减小冲突的可能性
减小锁的力度一个很好的例子就是ConcurrentHashMap,对于ConcurrentHashMap来说，减小锁的粒度就是将其内部结构再次分成多个HashMap，成之为段(SEGMENT)

### 分离锁替换独占锁

这个不用多说，ReentrantLock与ReentratReadWriteLock就是很好的例子，我们要尽量使用读写锁替换独占锁

### 锁分离

将读写锁再往外延伸一点就是将锁分离，不同的锁干不同的事，明确分工，下面看下JDK中LinkedBlockingQueue的实现
``` java
	public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
			/** Lock held by take, poll, etc */
			private final ReentrantLock takeLock = new ReentrantLock();
			 /** Lock held by put, offer, etc */
			private final ReentrantLock putLock = new ReentrantLock();                    
			public E take() throws InterruptedException {
				E x;
				int c = -1;
				final AtomicInteger count = this.count;
				final ReentrantLock takeLock = this.takeLock;
				takeLock.lockInterruptibly();
				try {
					while (count.get() == 0) {
						notEmpty.await();
					}
					x = dequeue();
					c = count.getAndDecrement();
					if (c > 1)
						notEmpty.signal();
				} finally {
					takeLock.unlock();
				}
				if (c == capacity)
					signalNotFull();
				return x;
			}

			public E take() throws InterruptedException {
				E x;
				int c = -1;
				final AtomicInteger count = this.count;
				final ReentrantLock takeLock = this.takeLock;
				takeLock.lockInterruptibly();
				try {
					while (count.get() == 0) {
						notEmpty.await();
					}
					x = dequeue();
					c = count.getAndDecrement();
					if (c > 1)
						notEmpty.signal();
				} finally {
					takeLock.unlock();
				}
				if (c == capacity)
					signalNotFull();
				return x;
			}				
		}
```

### 锁粗化

为了提高并发效率，我们要减小持有锁的时间，然后再释放锁。但凡事都有一个度，反复对锁进行请求也会浪费资源，降低性能。如果遇到一连串连续对同一锁进行请求，那么我们就需要把所有锁请求整合成对锁的一次请求，这就是锁的粗化
``` java
	synchronized (this) {
		for(int i = 0; i < 10000; i++) {
			count++;
		}
	}
	for(int i = 0; i < 10000; i++) {
		synchronized (this) {
			count++
		}
	}
```
明显的，肯定是上一个的效率更高

### JVM对锁的优化

#### 偏向锁
偏向锁的核心思想就是：如果一个线程获得锁，那么锁就进入偏向模式，当这个线程再次请求锁时，无需做任何操作，这就节省了有关锁申请所耗费的时间

#### 轻量级锁
偏向锁失败之后，JVM并不会挂起，而是使用一种轻量级锁优化，简单地将对象头部作为指针，指向持有锁的线程堆栈内部来判断对象是否持有锁，成功则获取临界资源，失败则膨胀为重量级锁

#### 自旋锁
在线程被操作系统挂起之前，JVM还会做最后的努力，当线程无法获得锁时，它会假设在不久的将来它会这个锁，因此做几个空循环(这就是自旋的含义),在循环完之后，获得了锁就进入临界区，否则就真正挂起，在一定程度上还是提高性能了

#### 锁清除
JVM在编译的时候，通过上下文扫描，去除不可能共享资源的锁，这里主要指的是并发安全类的一些方法，例如Vector.add()如果操作只是一个局部变量，那么就没有锁定的必要了
``` java
	public String[] createStrings() {
		Vector<String> v = new Vector<>();
		for(int i = 0; i < 100; i++) {
			v.add(Integer.toString(i));
		}
		return v.toArray(new String[]{});
	}
```