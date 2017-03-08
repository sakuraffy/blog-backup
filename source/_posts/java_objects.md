---
title: Objects的使用
date: 2016-09-16 11:07:04
tags:
	- Java
---
我们知道Object是所有类的直接或间接父类，有时又会看到Objects这个类，那么它又是什么呢?

Objects是java.util包下面的一个类，很明显是一个工具类，它主要处理的是与Object相关的操作
{% qnimg java/objects/p1.png 'class:class1 class2' normal:yes %}

通过上图可以看出，Objects没有公开的构造方法，所有的方法都为静态方法，那么我们调用就必须以Objects.的方式调用

<!--more-->

下面看一下几个有代表性的方法实现
``` java
	//深度比较两个对象是否equals
	public static boolean deepEquals(Object a, Object b) {
        if (a == b)
            return true;
        else if (a == null || b == null)
            return false;
        else
            return Arrays.deepEquals0(a, b);
    }
	
	public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }
	
	//采用可变参数，实际上也是调用数组的hashCode()方法                                                
	public static int hash(Object... values) {
        return Arrays.hashCode(values);
    }
	
	//判断Object对象是否为空
	public static boolean isNull(Object obj) {
        return obj == null;
    }
	
	//判断Object对象是否不为空
	public static boolean nonNull(Object obj) {
        return obj != null;
    }
	
	//要求Object对象不为空，否则抛出Exception
	public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }
```

