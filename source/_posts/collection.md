---
title: 集合系列02-Collection和AbstractCollection
date: 2016-10-04 14:31:02
tags:
	- Collection
---
前面提到了Java集合的架构，接下来我们说说Collection接口和AbstractCollection抽象类

<!--more-->

### Collection

``` java
	public interface Collection<E> extends Iterable<E> {
		public int size();
		public boolean isEmpty();
		public boolean contains(Object obj);
		public boolean add(E e);
		public boolean remove(Object obj);
		public boolean equals(Object obj);
		public int hashCode();
		public void clear();
		
		public boolean addAll(Collection<? extends E> c);
		public boolean containAll(Collection<? extends E> c);
		public boolean removeAll(Collection<? extends E> c);
		public boolean retainAll(Collection<? extends E> c);
		
		public Iterator<E> iterator();
		
		public E[] toArray(E[] e);
		public Object[] toArray();
	}
```
一些见名知意的方法就不说了，这里提一下的就是hashCode()和equals()是一起的，重写equals()方法就必须重写hashCode()方法。retainAll(Collection<? extends E> c)就是仅保留此 collection 中那些也包含在指定 collection 的元素。toArray(E[] e)是需要为数组分配内存空间的，而toArray()则不需要

### AbstractCollection

SAbstractCollection<E>是一个抽象类，它的主要作用是将Collection的一些方法进行实现，主要是利用iterator()、size()和一些单操作对多操作进行处理
``` java
	public abstract class AbstractCollection<E> implements Collection<E> {
		//其中抽象方法
		public abstract Iterator<E> iterator();
		public abstract int size();
		
		//非抽象方法，但相当于抽象方法
		public boolean remove(Object o) {}
		public boolean add(E e) {
			throw new UnsupportedOperationException();
		}
		
		//非抽象方法
		public boolean isEmpty() {}
		public boolean contains(Object o) {}
		public boolean containsAll(Collection<?> c) {}
		public boolean addAll(Collection<? extends E> c) {}                                   
		public boolean removeAll(Collection<?> c) {}
		public boolean retainAll(Collection<?> c) {}
		public void clear() {}
		public Object[] toArray() {}
		public <T> T[] toArray(T[] a) {}
		public String toString() {
			// [a,b,c]这种格式
		}
	}
```
这里值得一提的就是removeAll()、retainAll()和addAll()这三个方法只要元素成功即返回true

接下来就看一下 toArray()的具体实现
``` java
	public <T> T[] toArray(T[] a) {
        // Estimate size of array; be prepared to see more or fewer elements
        int size = size();
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);                         
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // fewer elements than expected
                if (a == r) {
                    r[i] = null; // null-terminate
                } else if (a.length < i) {
                    return Arrays.copyOf(r, i);
                } else {
                    System.arraycopy(r, 0, a, 0, i);
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }
        // more elements than expected
        return it.hasNext() ? finishToArray(r, it) : r;
    }
	
	private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
        int i = r.length;
        while (it.hasNext()) {
            int cap = r.length;
            if (i == cap) {
                int newCap = cap + (cap >> 1) + 1;
                // overflow-conscious code
                if (newCap - MAX_ARRAY_SIZE > 0)
                    newCap = hugeCapacity(cap + 1);
                r = Arrays.copyOf(r, newCap);
            }
            r[i++] = (T)it.next();
        }
        // trim if overallocated
        return (i == r.length) ? r : Arrays.copyOf(r, i);
    }
```