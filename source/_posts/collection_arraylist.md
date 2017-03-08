---
title: 集合系列04-ArrayList的原理与实现
date: 2016-10-07 22:48:41
tags:
	- Collection
---
前面提到List接口，那么它的一个完整实现就是ArrayList。array数组的意思，所以这个List是以动态数组为基础实现的
### 基本属性
``` java
	public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
		private int size;
		transient Object[] elementData;
	}
```
这就是ArrayList的两个属性，elementData用来装数据，而size则所装元素的个数

<!--more-->

下面就说一下方法的实现吧。对于一些见名知意简单的函数，这里就不涉及了。
``` java
	// 学会利用已有函数编程
	public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
	
	//注意null可以为任意类型，判空至关重要
	public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)                                         
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
	
	private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
	
	//与上个函数对别，add()是可以在size位置加入的
	private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
	
	//编程考虑垃圾回收(GC)
	public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
	
	//remove()和set()都是返回oldValue
	public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
	
	//类似的addAll(),reamoveAll(),retainAll()都是一个成功就返回true
	public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
```
上面的方法其实也很简单，只是多了一些注意点而已。下面就重点讨论一下数组动态增长、toArray、clone、batchRemove、writeObject和readObject这几个方法

### 数组动态增长

我们知道数组是有固定长度的，ArrayList的关键之一就是数组的动态增长
``` java
	private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
	
	private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:                             
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
	
	private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```
它上面说的比较严谨，简而言之就是当所需要minCapacity > dataElement.length,就在dataElement数组长度的基础上加上其一半

### ToArray

先来看一下一个小Demo吧
``` java
	public class Demo {
		public static void main(String[] args) {
			ArrayList<String> al = new ArrayList<String>(3);
			for(int i = 0; i < 3; i++) {
				al.add("Hello" + i);
			}
			String[] str1 = new String[]{"HH0","HH1","HH2","HH3","HH4","HH5"};
			String[] str2 = new String[1];
			str1 = al.toArray(str1);
			str2 = al.toArray(str2);
			System.out.println(Arrays.toString(str1));
			System.out.println(Arrays.toString(str2));
		}
	}
```
输出结果 ：
``` java
[Hello0, Hello1, Hello2, null, HH4, HH5]
[Hello0, Hello1, Hello2]
```

为什么str1会输出这个结果呢？看一下源码就一目了然了
``` java
	@SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```

### Clone 

clone()是实现cloneable接口必须实现的一个方法，具体的可以看[原型模式](https://sakuraffy.github.io/pattern_prototype/)
``` java
	public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }
```

### BatchRemove

``` java
	private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,                      
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```
batchRemove()主要用于retainAll()和removeAll()这两个方法。其主要的思想就是 ： 把满足条件的元素还是用这个数组从0开始保存起来，如果满足条件的个数小于原来的长度就将其多余的设为null，并减小size

### WriteObject和ReadObject

writeObject()和readObject()是实现序列化的。其中关于序列化请参考[序列化机制](https://sakuraffy.github.io/java_serializable/)
``` java
	private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()               
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
	
	private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```
其中ArryList的序列化，无论是读还是写都是先将大小处理，然后才是各个元素

上面讨论了ArrayList的方法，其实ArrayList中还有两个内部类Itr和ListItr
``` java
	public class ArrayList<E> extends AbstractList<E> {
		private class Itr implements Iterator<E> {}
		private class ListItr extends Itr implements ListIterator<E> {}
	}
```
值得一提的是Itr和ListItr中有expectedModCount和modCount来确保安全性，其余的就没什么可说的，如果有不明白的可以看一下[迭代器模式](https://sakuraffy.github.io/pattern_iterator/)
