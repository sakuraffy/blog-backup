---
title: 迭代器模式(Iterator)
date: 2016-11-08 11:12:46
tags:	
	- 设计模式
---
### 什么是Iterator模式

所谓Iterator模式就是提供一种方法顺序访问一个聚合(集合)对象中的各个元素，而不是暴露其内部的表示。在开发中，我们有可能需要以不同的方式来遍历整个整合对象，但是我们不希望在聚合对象的抽象接口层中充斥着各种不同的便利操作。这个时候我们就需要这样一种东西，它应该具备如下三个功能：

- 能够便利一个聚合对象。
- 我们不需要了解聚合对象的内部结构。
- 能够提供多种不同的遍历方式。

<!--more-->

这三个功能就是迭代器模式需要解决的问题。作为一个功能强大的模式，迭代器模式把在元素之间游走的责任交给迭代器，而不是聚合对象。这样做就简化了聚合的接口和实现，也可以让聚合更专注在它所应该专注的事情上，这样做就更加符合单一责任原则。

### Iterator所涉及的角色 

抽象容器：一般是一个接口，提供一个iterator()方法，例如java中的Collection接口，List接口，Set接口等

具体容器：就是抽象容器的具体实现类，比如List接口的有序列表实现ArrayList，List接口的链表实现LinkList，Set接口的哈希列表的实现HashSet等

抽象迭代器：定义遍历元素所需要的方法，一般来说会有这么三个方法：取得第一个元素的方法first()，取得下一个元素的方法next()，判断是否遍历结束的方法isDone()（或者叫hasNext()），移出当前对象的方法remove()

迭代器实现：实现迭代器接口中定义的方法，完成集合的迭代

### 代码实现

#### Iterator接口
``` java
	package cn.sakuraffy.iterator;

	public interface Iterator<T> {
		boolean hasNext();
		T next();
	}
```
#### 抽象容器接口
``` java	
	package cn.sakuraffy.iterator;

	public interface Aggregate<T> {
		void add(T obj);
		Iterator<T> itreator();
	}
```
#### 具体容器实现
``` java
	package cn.sakuraffy.iterator;

	import java.util.NoSuchElementException;

	public class ConcreteAggregate<T> implements Aggregate<T>{                                    
		private Object[] data = new Object[10];
		private int size;
		
		@Override
		public void add(T obj) {
			if (size < data.length) {
				data[size++] = obj;
			}
		}

		@Override
		public Iterator<T> itreator() {
			return new ConcreteIterator();
		}

		private class ConcreteIterator implements Iterator<T>{
			private int cursor;
			
			@Override
			public boolean hasNext() {
				return cursor < size;
			}

			@SuppressWarnings("unchecked")
			@Override
			public T next() {
				if (cursor < size) {
					return (T) data[cursor++];
				} else {
					throw new NoSuchElementException();
				}
			}
			
		}
	}
```
#### Client端测试
``` java
	package cn.sakuraffy.iterator;

	public class Client {
		public static void main(String[] args) {
			Aggregate<Integer> ag = new ConcreteAggregate<>();
			ag.add(1);
			ag.add(2);
			ag.add(2);
			ag.add(2);
			Iterator<Integer> it = ag.itreator();
			while(it.hasNext()){
				System.out.println(it.next());
			}
		}
	}
```
输出结果 ：
``` java
	1
	2
	2
	2
```

### ArrayList中Iterator的使用

``` java
	private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;
	}
```
这就是ArrayList.Itr的三个属性，至于expectedModCount和modCount是干什么用的，我们先看一个小Demo
``` java 
	package cn.sakuraffy.iterator;

	import java.util.ArrayList;
	import java.util.Iterator;

	public class ALI {
		public static void main(String[] args)  {
			ArrayList<Integer> list = new ArrayList<Integer>();
			list.add(2);
			Iterator<Integer> iterator = list.iterator();
			while(iterator.hasNext()) {
				Integer i = iterator.next();
				if (i == 2) {
					list.remove(i);
					//iterator.remove();
				}
			}
		}
	}
```
运行程序就会抛出java.util.ConcurrentModificationException,但是使用iterator.remove()则会正常运行，这是为什么呢？
``` java
	private class Itr implements Iterator<E> {
		public void remove() {
			if (lastRet < 0)
				throw new IllegalStateException();
			checkForComodification();

			try {
				ArrayList.this.remove(lastRet);
				cursor = lastRet;
				lastRet = -1;
				expectedModCount = modCount;
			} catch (IndexOutOfBoundsException ex) {
				throw new ConcurrentModificationException();
			}
		}
		
		final void checkForComodification() {
			if (modCount != expectedModCount)
				throw new ConcurrentModificationException();
		}
	}
```
看到这是不是就有些明白了，expectedModCount和modCount就是为了解决集合遍历时删除的问题(也仅仅只是单线程，至于多线程后面再说)
