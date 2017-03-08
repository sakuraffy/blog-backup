---
title: 并发编程（二十）不变模式
date: 2016-08-01 18:14:47
tags:
	- 并发编程
---
### 什么是不变模式

使用一种不可以改变的对象，依靠对象的不可变性，可以确保其在没有同步操作的多线程环境中依然始终保持内部状态的一致性和正确性，这就是不变模式

### 不变模式的思想

不变模式的核心思想是：一个对象一旦被创建，则它的内部状态就永远不会发生改变，所以没有一个线程可以修改其内部状态和数据，同时内部状态也不会自发发生改变。基于这些特性，对不变对象的多线程操作不需要同步
不变模式和只读属性也是有不同的。只读属性有可能会自发地改变，例如对象的存活时间，虽然只读，但它会随着时间的改变而变化

<!--more-->

### 不变思想的实例

JDK中一些不变对象 ： String，Integer，Double……
``` java
	public final class String {}
	public final class Integer extends Number implements Comparable<Integer> {}
	public final class Double extends Number implements Comparable<Double> {}
```
下面给出自己写的示例
``` java
	public final class UnModifiable {
		private final int id;
		private final String name;
		
		public UnModifiable(int id, String name) {
			super();
			this.id = id;
			this.name = name;
		}

		public final int getId() {
			return id;
		}

		public final String getName() {
			return name;
		}
	}
```
不变模式应该注意的几点 ：
- 所有的属性都应该用final修饰
- 属性没有setter方法
- 必须要用一个全部属性的构造方法
- 确保没有子类可以修改它的行为（可以用final修饰class）