---
title: 集合系列06-LinkedList的原理与使用
date: 2016-10-10 08:34:17
tags:
	- Collection
---
前面我们了解了ArrayList，知道它是以数组为基础，每个数组都有固定的长度。通过学习数据结构，我们知道线性表还有一种就是链表。那么数组和链表各自的优缺点又是什么呢？
- 数组 ： 方便查找，增加删除繁琐
- 链表 ： 方便删除和增加，查找麻烦

<!--more-->

``` java
	public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
		transient int size = 0;
		transient Node<E> first;
		transient Node<E> last;
		
		private static class Node<E> {
			E item;
			Node<E> next;
			Node<E> prev;

			Node(Node<E> prev, E element, Node<E> next) {
				this.item = element;
				this.next = next;
				this.prev = prev;
			}
		}
	}
```
这是LinkedList主要的几个属性和一个很重要的内部类Node。size用来保存元素个数，first和last分别为首尾节点

接下来还是讨论一下方法的实现与应用
``` java
	public void addLast(E e) {
        linkLast(e);
    }
	
	/**
	 *
	 * 思路 ： 先取出last节点，再将新节点连接到last节点后面，最后判断原last节点是否为null，是就将其设为
	 * first节点，否则为其设置后继节点，size++
	 */
	void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
	
	public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
	
	/**
	 * 思路 ： 先取出last节点，获取节点元素，找到last节点的前驱并设为last，判断此时的last节点是否为空    
	 * 是就将first设为null，否则将其后驱节点设为null,size--
	 * 
	 */
	public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
	
	public boolean add(E e) {
        linkLast(e);
        return true;
    }
	
	public E remove() {
        return removeFirst();
    }
```
上面是LinkedList所独有的，只介绍了关于last的部分，至于first，与last原理相同，其它的poll()、push()、offer()等都是直接调用这些基础函数
