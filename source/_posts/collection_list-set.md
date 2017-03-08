---
title: 集合系列03-List和Set及其抽象类
date: 2016-10-05 09:25:03
tags:
	 - Collection
---
前面提到了Collection接口和它的抽象类，而在集合架构中提到Collection是将List、Set和Queue高度抽象而来

### List

List是有序的集合，那么理所当然的会在List接口加入关于索引操作

<!--more-->

``` java
	public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {     
		//extends Collection
		int size();
		boolean isEmpty();
		boolean contains(Object obj);
		boolean add(E e);
		boolean remove(Object obj);
		boolean equals(Object obj);
		int hashCode();
		void clear();
		
		boolean addAll(SCollection<? extends E> sc);
		boolean containAll(SCollection<? extends E> sc);
		boolean removeAll(SCollection<? extends E> sc);
		boolean retainAll(SCollection<? extends E> sc);
		
		SIterator<E> iterator();
		
		E[] toArray(E[] e);
		Object[] toArray();
		
		//自己所独有的
		void add(int index, E e);
		E get(int index);
		E set(int index, E e);
		E remove(int index);
		E indexOf(Object obj);
		E lastIndexOf(Object obj);
		boolean addAll(int index,Collection<? extends E> c);
		ListIterator<E> listIterator();
		List<E> subList(int fromIndex, int toIndex);
	}
```
### AbstractList

``` java
	public boolean add(E e) {
		add(size(), e);
		return true;
	}
	public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }
	
	public Iterator<E> iterator() {
        return new Itr();
    }
	
	public ListIterator<E> listIterator() {
        return listIterator(0);
    }
	
	private class Itr implements Iterator<E> {}
	private class ListItr extends Itr implements ListIterator<E> {}
```
Iterator和ListIterator都是采用的内部类。get(),set()和remove()都是采用和add()一样的方法抛出UnsupportedOperationException

### Set

Set为无序不可重复的集合，Set接口和Collection接口完全一样，没有自己所独有的方法。但为什么要抽象出这个接口，这是为了整体架构，方便以后对Set的拓展

### AbstractSet

AbstractSet重写了equals()和hashCode()方法
``` java
	public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {       
		public boolean equals(Object o) {
			if (o == this)
				return true;

			if (!(o instanceof Set))
				return false;
			Collection<?> c = (Collection<?>) o;
			if (c.size() != size())
				return false;
			try {
				return containsAll(c);
			} catch (ClassCastException unused)   {
				return false;
			} catch (NullPointerException unused) {
				return false;
			}
		}
    }
	
	public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;

        if (size() > c.size()) {
            for (Iterator<?> i = c.iterator(); i.hasNext(); )
                modified |= remove(i.next());
        } else {
            for (Iterator<?> i = iterator(); i.hasNext(); ) {
                if (c.contains(i.next())) {
                    i.remove();
                    modified = true;
                }
            }
        }
        return modified;
    }
	
```