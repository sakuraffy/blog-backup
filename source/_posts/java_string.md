---
title: 字符串（一）String
date: 2016-09-01 12:55:38
tags: 
	- Java
---
我们在编程中，免不了会与字符串打交道，那么就来JDK给我们提供的字符串架构

先来看一个接口 CharSequence
``` java
	public interface CharSequence {
		int length();
		char charAt(int index);
		CharSequence subSequence(int start, int end);
		public String toString();
	}
```
再来看看其实现类 String

<!--more-->

``` java
	public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
		private final char value[];
		private int hash; // Default to 0
	}
```
由此看来String其本质就是一个char[]
``` java
	public String() {
        this.value = "".value;
    }
	public String(String original) {}
	public String(char value[]) {}
	public String(char value[], int offset, int count) {}
	public String(int[] codePoints, int offset, int count) {}
	public String(byte bytes[], int offset, int length, Charset charset) {}
	public String(StringBuffer buffer) {}
	public String(StringBuilder builder) {}
	 /*
    * Package private constructor which shares value array for speed.
    * this constructor is always expected to be called with share==true.
    * a separate constructor is needed because we already have a public
    * String(char[]) constructor that makes a copy of the given char[].
    */
	String(char[] value, boolean share) {}
```
通过上面的构造函数可以发现 ：
byte、int和char类型的数组都可以转换为String，还可以指定其长度和字符集

下面看一下重头戏 ： String类方法的使用
``` java
	public class TestString {
		public static void main(String[] args) throws UnsupportedEncodingException {
			String str = new String("Sakuraffy");
			// implements CharSequence
			System.out.println(str.length()); // 9
			System.out.println(str.charAt(0)); // S
			System.out.println(str); //默认调用str.toString() Sakuraffy
			System.out.println(str.substring(0, 6)); // Sakura
			
			// implements Comparable
			System.out.println(str.compareTo("Sakuraffy"));  // 0
			
			// others
			System.out.println(str.isEmpty()); // false
			System.out.println(str.codePointAt(0)); // 打印对应字符的Unicode编码  83        
			System.out.println(str.codePointBefore(1)); //与上面等价 83
			//Returns the number of Unicode code points in the specified text
			System.out.println(str.codePointCount(0, str.length()));  // 9
			
			//返回Unicode编码下String，对应字符的编码byte数组
			byte[] b = new byte[10];
			b = str.getBytes();
			System.out.println(Arrays.toString(b)); 
			//[83, 97, 107, 117, 114, 97, 102, 102, 121]
			String s = new String("再见");
			byte[] bb = new byte[10];
			bb = s.getBytes("GBK");
			System.out.println(Arrays.toString(bb));
			//[-44, -39, -68, -5]
		
			//截取字符串到指定数组中
			char[] c = new char[10];
			//getChars(int srcBegin, int srcEnd, char dst[], int dstBegin)
			str.getChars(0, 6, c, 1);
			System.out.println(Arrays.toString(c));
			//[ ,S,a,k,u,r,a, , , ]
			
			//与CharSequence子类比较equals
			StringBuffer sb = new StringBuffer("Sakuraffy");
			System.out.println(str.equalsIgnoreCase("SAKURAFFY")); // true
			System.out.println(str.equals(sb)); // false
			System.out.println(str.contentEquals(sb)); // true
			
			//判断是否以某字符串开头或结尾
			System.out.println(str.startsWith("Sakura")); // true
			System.out.println(str.endsWith("ffy")); // true
			
			//索引定位
			System.out.println(str.indexOf("a"));  // 1
			System.out.println(str.lastIndexOf("a")); // 5
			System.out.println(str.indexOf("Sakura")); // 0
			
			//字符串链接
			System.out.println(str.concat(" hello")); // Sakuraffy hello
			
			//字符串替换
			System.out.println(str.replace('a', 'A')); // SAkurAffy
			System.out.println(str.replaceAll("Sa", "su")); // sukuraffy
			
			//字符串分割
			System.out.println(Arrays.toString(str.split("a"))); // [S, kur, ffy]
			
			//字符串大小写转换
			System.out.println(str.toUpperCase()); // SAKURAFFY
			System.out.println(str.toLowerCase()); // sakuraffy
			
			//其它数据类型转换为String类型
			System.out.println(String.valueOf(12));  // 12
			
			//去除字符串前后空格
			System.out.println(" hello ".trim()); // hello
			
		}
	}
```

> String类的实例对象从创建就是final不能被修改，所以每次修改后都是另外一个对象

``` java
	public class TestString {
		public static void main(String[] args) {
			String str1 = new String("Sakuraffy");
			String str2 = str1.replace('a','b');
			System.out.println(str1 == str2); // false
		}
	}
```
