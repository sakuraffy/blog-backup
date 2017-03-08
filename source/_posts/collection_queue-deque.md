---
title: 集合系列05-Queue和Deque及其抽象类
date: 2016-10-08 10:20:20
tags:
	- Collection
---
前面提到了List和Set接口及其抽象类，今天我们就讨论一下关于队列Queue和Deque及其抽象类

### Queue

下来看一下Queue接口的几个方法
{% qnimg collection/queue-deque/p1.png 'class:class1 class2' normal:yes %}

<!--more-->

#### 两个添加方法: 继承Collection的add()和自身所特有的offer()
- offer()在添加时容量不足时会抛出IllegalStateException
- add()在添加时容量不足时不会抛出IllegalStateException


#### 两个删除方法：继承Collection的remove()和自身所特有的poll()
- 当队列为空时，remove()会抛出java.util.NoSuchElementException
- 当队列为空时，poll()返回null
	
#### 两个获得元素方法：都是Queue自身特有的。都是获得队列的头元素。
- 当队列为空时，element()会抛出NoSuchElementException
- 当队列为空时，peek()返回null
	
### AbstractQueue

{% qnimg collection/queue-deque/p2.png 'class:class1 class2' normal:yes %}

AbstractQueue重写了AbstractCollection的addAll()、clear()以及remove()和add()方法
``` java
	public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }
	
	public boolean add(E e) {
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
    }
```
通过上面的源码可以看出remove()和add()都是调用了offer()、poll()方法，再加上判断，抛出Exception

### Deque

Queue的数据结构满足先进先出或者说后进后出，而Deque是一个双向的队列
下面看一下Deque接口中的方法，其中从Collection和Queue继承而来，这里就不说了
``` java
	/* stack method */
	//添加元素
	void push(E e);
	//删除元素
	E pop();
	
	/* myself */
	void addFirst(E e);
	void addLast(E e);
	boolean offerFirst(E e);
	boolean offerLast(E e);
	E removeFirst();
	E removeLast();
	E pollFirst();
	E pollLast();
	E getFirst();
	E getLast();
	E peekFirst();
	E peekLast();
	boolean removeFirstOccurrence(Object o);
	boolean removeLastOccurrence(Object o);
```
其中各个方法所实现的功能就见名知意