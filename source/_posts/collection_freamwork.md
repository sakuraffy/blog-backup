---
title: 集合系列01-集合框架
date: 2016-10-01 09:59:29
tags:
	- Collection
---
一个集合（collection）就是一个存储一组对象的容器，一般将这些对象称为集合的元素（element）。Java 集合中一般分为这四块 ： Collection接口及其实现类、Map接口及其实现类、Iterator接口及实现类 和工具类
先来看看整体的其框架的UML类图 ：
{% qnimg collection/freamwork/p1.png 'class:class1 class2' normal:yes %}

### Collection

Collection是List、Set、Queue等集合高度抽象出来的接口，它包含了这些集合的基本操作.实现关系层次图：

<!--more-->

{% qnimg collection/freamwork/p2.png 'class:class1 class2' normal:yes %}

### Map

Map提供key到value的映射。一个Map中不能包含相同的key，每个key只能映射一个value.实现关系层次图：　
{% qnimg collection/freamwork/p3.png 'class:class1 class2' normal:yes %}

### Iterator

Iterator用来对于容器的遍历，请参考[迭代器模式(Iterator)](https://sakuraffy.github.io/pattern_iterator/)
``` java
	public interface Iterator<E> {
		boolean hasNext();
		E next();
	}
	public interface ListIterator<E> extends Iterator<E> {
		 boolean hasPrevious();
		 E previous();
		 int nextIndex();
		 int previousIndex();
		 void remove();
		 void set(E e);
		 void add(E e);
	}
```

### 工具类

关于Collection的工具类Collections一般主要处理容器的排序，查找等操作