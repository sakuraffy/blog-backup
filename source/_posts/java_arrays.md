---
title: Arrays的使用
date: 2016-09-06 10:59:22
tags:
	- Java
---
在Java基础中，数据类型除了八大基本数据类型，最常用的就是数组了。那对于数组中元素排序，查找又包装在哪呢？
那就是接下来我们要说的java.util.Arrays这个类

同样的，Arrays这个类没有公开的构造方法，其中所有的方法都是静态的

### Sort

``` java
	public static void sort(int[] a) {
        DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
    }
	public static void sort(int[] a, int fromIndex, int toIndex) {
        rangeCheck(a.length, fromIndex, toIndex);
        DualPivotQuicksort.sort(a, fromIndex, toIndex - 1, null, 0, 0);
    }
	
	public static void parallelSort(int[] a) {}
	public static <T> void parallelSort(T[] a, Comparator<? super T> cmp) {} 	 
```
从上面的源码可以看出Arrays可以解决各种数组，甚至是引用类型的泛型。至于怎么实现的，那就是算法的问题，在这里，就不讨论了。值得一提的引用类型的数组只能使用parallelSort。而且还可以使用比较器

<!--more-->

### Search

``` java
	public static int binarySearch(long[] a, long key) {
        return binarySearch0(a, 0, a.length, key);
    }
```
Arrays采用的是二分法查找，使用这种查找必须先对数组进行排序

### Equals

``` java
	public static boolean equals(Object[] a, Object[] a2) {
        if (a==a2)
            return true;
        if (a==null || a2==null)
            return false;

        int length = a.length;
        if (a2.length != length)
            return false;

        for (int i=0; i<length; i++) {
            Object o1 = a[i];
            Object o2 = a2[i];
            if (!(o1==null ? o2==null : o1.equals(o2)))
                return false;
        }

        return true;
    }
```
比较两个数组是否equals()

### Fill

``` java
	public static void fill(long[] a, long val) {
        for (int i = 0, len = a.length; i < len; i++)
            a[i] = val;
    }
	
	public static void fill(long[] a, int fromIndex, int toIndex, long val) {
        rangeCheck(a.length, fromIndex, toIndex);
        for (int i = fromIndex; i < toIndex; i++)
            a[i] = val;
    }
```
对数组进行填充

### Copy

``` java
	public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }

	public static <T,U> T[] copyOfRange(U[] original, int from, int to, Class<? extends T[]> newType) {
        int newLength = to - from;
        if (newLength < 0)
            throw new IllegalArgumentException(from + " > " + to);
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, from, copy, 0,
                         Math.min(original.length - from, newLength));
        return copy;
    }
```
上面的就是Arrays的copyOf()实现的源码，初一看，哇，这都是什么啊？下面就展示一个小实例
``` java
	public class TestArrays {
		public static void main(String[] args) {
			int[] arr = new int[] {1,2,3,4,5,6,7,8,9};
			System.out.println(Arrays.toString(arr));
			
			//将arr数组的容量扩展至15，并将原数组内容复制至此，其余的用默认值填充
			arr = Arrays.copyOf(arr, 15);
			System.out.println(arr.length);
			System.out.println(Arrays.toString(arr));
			
			//将arr数组的容量缩至5，并将原数组前5个复制至此
			arr = Arrays.copyOf(arr, 5);
			System.out.println(arr.length);
			System.out.println(Arrays.toString(arr));			
		}
	}
```
输出结果 ：
``` java
[1, 2, 3, 4, 5, 6, 7, 8, 9]
15
[1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 0, 0, 0, 0, 0]
5
[1, 2, 3, 4, 5]
```

### ToString

``` java
	public static String toString(int[] a) {
        if (a == null)
            return "null";
        int iMax = a.length - 1;
        if (iMax == -1)
            return "[]";

        StringBuilder b = new StringBuilder();
        b.append('[');
        for (int i = 0; ; i++) {
            b.append(a[i]);
            if (i == iMax)
                return b.append(']').toString();
            b.append(", ");
        }
    }
	
	public static String deepToString(Object[] a) {                                               
        if (a == null)
            return "null";

        int bufLen = 20 * a.length;
        if (a.length != 0 && bufLen <= 0)
            bufLen = Integer.MAX_VALUE;
        StringBuilder buf = new StringBuilder(bufLen);
        deepToString(a, buf, new HashSet<Object[]>());
        return buf.toString();
    }
```
将数组以字符串的形式展现出来，其中toString()对应的是一维，而deepToString()对应的则是多维
