---
title: 并发编程(十六) 并发容器
date: 2016-07-24 12:21:59
tags:
	- 并发编程
---
编程过程中，我们会经常使用容器，但我们知道ArrayList，LinkedList等都是线程不安全的。那么在并发编程中，我们又怎么解决这个问题呢？
使用Collections工具类将上述数据结构包装成安全的，这里以ArrayList举例

<!--more-->

``` java
	static class SynchronizedList<E> extends SynchronizedCollection<E> implements List<E> {
		private static final long serialVersionUID = -7754090372962971524L;
		final List<E> list;
		public void add(int index, E element) {
			synchronized (mutex) {list.add(index, element);}
		}
	}
```
关于List的所有操作都会使用这个mutex进行同步，从而保证线程安全，但它仅仅只用于并发级别不高的

使用线程安全的数据结构
### ConcurrentLinkedQueue

我们先来看看它的具体实现
``` java
	public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {
			private static class Node<E> {
				volatile E item;
				volatile Node<E> next;
				boolean casItem(E cmp, E val){}
				void lazySetNext(Node<E> val) {}
				boolean casNext(Node<E> cmp, Node<E> val) {}
			}
	}
```
我们可以看到Node中的方法都没有加锁，线程安全全部由CAS操作来保证，关于CAS操作，后面会说

### CopyOnWriteArrayList

为了更好的提高读写性能，JDK提供了CopyOnWriteArrayList类，对它来说，读写是不需要加锁的。而写数据的时候，它并不在原来的数据上修改，而是将原来的数据Copy一份修改好后，再将原来的替换
下面就展示有关读取的实现
``` java
	private transient volatile Object[] array;
	final Object[] getArray() {
        return array;
    }
	private E get(Object[] a, int index) {
        return (E) a[index];
    }
	public E get(int index) {
        return get(getArray(), index);
    }
```
相对于读，写就更麻烦一点
``` java
	public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

### BlockingQueue

BlockingQueue主要作为数据共享的通道，它会让服务线程在队列为空时进行等待，当有新的消息进入队列后自动将线程唤醒, 值得一说的是BlockingQueue是个接口，它的主要实现有基于数组的ArrayBlockingQueue和基于链表的LinkedBlockingQueue
下面看看JDK源码的实现
``` java
	final Object[] items;
	/** Condition for waiting takes */
	private final Condition notEmpty;

	/** Condition for waiting puts */
	private final Condition notFull;
	public E take() throws InterruptedException {                                                 
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
	public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
```
 
