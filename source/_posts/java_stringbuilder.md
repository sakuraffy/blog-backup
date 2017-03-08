---
title: 字符串（二）StringBuffer和StringBuilder
date: 2016-09-03 17:48:40
tags:
	- Java
---
前面我们提到String对象每次修改都会重新构造一个新对象，但我们知道构造对象是很消耗资源的，那有什么解决方案呢？
StringBuffer和StringBuilder就专注于字符串对象的修改
我们先来看一个接口Appendable
``` java
	public interface Appendable {
		Appendable append(CharSequence csq) throws IOException;
		Appendable append(CharSequence csq, int start, int end) throws IOException;
		Appendable append(char c) throws IOException;
	}
```
下面看一下具体实现类StringBuilder

<!--more-->

``` java
	abstract class AbstractStringBuilder implements Appendable, CharSequence {                    
		char[] value;
		int count;
		AbstractStringBuilder(int capacity) {
			value = new char[capacity];
		}
	}
	public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence {    
		@Override
		public String toString() {
			// Create a copy, don't share the array
			return new String(value, 0, count);
		}

		public StringBuilder() {
			super(16);
		}
		public StringBuilder(String str) {
			super(str.length() + 16);
			append(str);
		}

	}
```
由构造方法可以看出，StringBuilder会在String所拥有的数组的容量加上16用来增加等操作
下面看看StringBuilder的一些核心方法
``` java
	public class TestStringBuilder {
		public static void main(String[] args) {
			StringBuilder sb = new StringBuilder("sakuraffy");                            
			//字符串appen
			System.out.println(sb.append(1)); // sakuraffy1
			System.out.println(sb.append(new Object())); 
			//sakuraffy1java.lang.Object@2a139a55
			
			//字符串insert
			System.out.println(sb.insert(0, "hello "));
			//hello sakuraffy1java.lang.Object@2a139a55
			
			//字符串delete
			System.out.println(sb.delete(15, sb.length()));
			//hello sakuraffy
			
			//字符串替换
			System.out.println(sb.replace(0, 5, "nihao"));
			
			//字符串反转
			System.out.println(sb.reverse());
			//yffarukas oahin
		}
	}
```
还有一个与StringBuilder非常相似的类，那就是StringBuffer，方法几乎一模一样，不同的是StringBuffer是线程安全的，而StringBuilder则是线程不安全的，String同样是线程安全的，因为它采用的是不变模式
``` java
	public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence {
		@Override
		public synchronized char charAt(int index) { }
		@Override
		public synchronized StringBuffer append(Object obj) {}
		......
	}
```