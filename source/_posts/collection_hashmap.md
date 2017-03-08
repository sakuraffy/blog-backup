---
title: 集合系列09-HashMap的原理与使用
date: 2016-10-20 11:30:12
tags:
	- Collection
---

在说到HashMap的实现与使用之前，先来看一下HashMap的数据结构，HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体
{% qnimg collection/hashmap/p2.jpg 'class:class1 class2' normal:yes %}

### 骨架实现和属性

``` java 
	public class HashMap<K,V> extends AbstractMap<K,V>
		implements Map<K,V>, Cloneable, Serializable {
		// 节点table数组
		transient Node<K,V>[] table;
		// 数组的增长因子
		final float loadFactor;
		//HashMap的实体集
		transient Set<Map.Entry<K,V>> entrySet;
		//HashMap中键的个数
		transient int size;
		// 保证遍历时操作的安全性
		transient int modCount;
		//table使用的最大限制
		int threshold;
	}
```

<!--more-->

### 构造函数

``` java
	public HashMap()
	public HashMap(int initialCapacity)
	public HashMap(int initialCapacity, float loadFactor);
```
这两个参数就是衡量HashMap性能好坏的关键因素，initialCapacity就是table数组的长度，而loadFactor为散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。对于使用链表法的散列表来说，负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费

### 扩容实现(resize)

``` java
final Node<K,V>[] resize() {
	Node<K,V>[] oldTab = table;
	//获取原table的长度
	int oldCap = (oldTab == null) ? 0 : oldTab.length;
	int oldThr = threshold;
	int newCap, newThr = 0;
	if (oldCap > 0) {
		//MAXIMUM_CAPACITY = 1 << 30
		if (oldCap >= MAXIMUM_CAPACITY) {
			threshold = Integer.MAX_VALUE;
			return oldTab;
		}
		//扩大为原来的2倍
		else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
				 oldCap >= DEFAULT_INITIAL_CAPACITY)
			newThr = oldThr << 1; // double threshold
	}
	else if (oldThr > 0) // initial capacity was placed in threshold
		newCap = oldThr;
	else {               // zero initial threshold signifies using defaults               
		//DEFAULT_INITIAL_CAPACITY = 1 << 4
		newCap = DEFAULT_INITIAL_CAPACITY;
		newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
	}
	//计算threshold
	if (newThr == 0) {
		float ft = (float)newCap * loadFactor;
		newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
				  (int)ft : Integer.MAX_VALUE);
	}
	threshold = newThr;
	@SuppressWarnings({"rawtypes","unchecked"})
		Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
	table = newTab;
	//将oldTable元素复制到table
	if (oldTab != null) {
		for (int j = 0; j < oldCap; ++j) {
			Node<K,V> e;
			if ((e = oldTab[j]) != null) {
				oldTab[j] = null;
				//链表只有一个元素
				if (e.next == null)
					newTab[e.hash & (newCap - 1)] = e;
				//链表的元素过多，使用红黑树来表示
				else if (e instanceof TreeNode)
					((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
				//依次替换链表
				else { // preserve order
					Node<K,V> loHead = null, loTail = null;
					Node<K,V> hiHead = null, hiTail = null;
					Node<K,V> next;
					do {
						next = e.next;
						if ((e.hash & oldCap) == 0) {
							if (loTail == null)
								loHead = e;
							else
								loTail.next = e;
							loTail = e;
						}
						else {
							if (hiTail == null)
								hiHead = e;
							else
								hiTail.next = e;
							hiTail = e;
						}
					} while ((e = next) != null);
					if (loTail != null) {
						loTail.next = null;
						newTab[j] = loHead;
					}
					if (hiTail != null) {
						hiTail.next = null;
						newTab[j + oldCap] = hiHead;
					}
				}
			}
		}
	}
	return newTab;
}
```
看到这你有可能会问：我自己初始化table大小，怎么能保证它一定是2的n次方呢？而链表移动的时候，全部移动，效率该有多低啊

其实不然，JDK也考虑到了这个，他用|和>>>来确保2的n次方
``` java
	public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
	static final int tableSizeFor(int cap) {
		int n = cap - 1;
		n |= n >>> 1;
		n |= n >>> 2;
		n |= n >>> 4;
		n |= n >>> 8;
		n |= n >>> 16;
		return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;             
	}
```

### 存储实现(put)

``` java
	final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
		//table为null或者长度为0，重置大小
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
		//table索引链表位置为null
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);	
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
			//若链表过长，使用红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);               
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

```

### 读取实现(get)

``` java
	public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
	//先通过hash找到table索引，再遍历比较
	final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))                   
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```