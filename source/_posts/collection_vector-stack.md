---
title: 集合系列07-Vector和Stack的原理与使用
date: 2016-10-13 10:57:11
tags:
	- Collection
---
### Vector

Vector应该说是线程安全的ArrayList，但又不全是，由于@since 1.1 那是还没有Iterator接口，所以多了Enumeration的形式来遍历集合
#### 基本属性
``` java
	public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
		protected Object[] elementData;
		protected int elementCount;
		protected int capacityIncrement;
	}
```

<!--more-->

这就是Vector类的几个属性，与ArrayList不同的是多了一个capacityIncrement，这就可以自定义增长数了。ArrayList增长默认的是原数组大小的一半，而Vector默认的则是原数组大小

这里看看Vector比ArrayList多出来的函数吧
``` java
	elementAt(int index) // get(int index)
	setElementAt(E e, int index) // set(E e, int index)
	E firstElement()
	
	//实现Enumeration形式的集合遍历
	public Enumeration<E> elements() {
        return new Enumeration<E>() {
            int count = 0;

            public boolean hasMoreElements() {
                return count < elementCount;
            }

            public E nextElement() {
                synchronized (Vector.this) {
                    if (count < elementCount) {
                        return elementData(count++);
                    }
                }
                throw new NoSuchElementException("Vector Enumeration");                       
            }
        };
    }
	
	//实质是将其后面的元素赋值为空
	public synchronized void setSize(int newSize) {
        modCount++;
        if (newSize > elementCount) {
            ensureCapacityHelper(newSize);
        } else {
            for (int i = newSize ; i < elementCount ; i++) {
                elementData[i] = null;
            }
        }
        elementCount = newSize;
    }
	
	//查看动态数组的容量大小
	public synchronized int capacity() {
        return elementData.length;
    }	
```

### Stack

Stack类其实还是比较简单的，就几个栈方法。栈在数据结构中为先进后出
``` java
	public class Stack<E> extends Vector<E> {
		//进栈其实调用的就是Vector的addElement
		public E push(E item) {
			addElement(item);

			return item;
		}
		
		//出栈
		public synchronized E pop() {
			E       obj;
			int     len = size();

			obj = peek();
			removeElementAt(len - 1);

			return obj;
		}
		
		public synchronized E peek() {
			int     len = size();

			if (len == 0)
				throw new EmptyStackException();                                     
			return elementAt(len - 1);
		}
		
		//因为是栈的原因，是从后往前找
		public synchronized int search(Object o) {
			int i = lastIndexOf(o);

			if (i >= 0) {
				return size() - i;
			}
			return -1;
		}
	}
```
