---
title: 序列化机制
date: 2016-09-05 12:55:38
tags: 
	- Java
---
这几天看IO的时候，需要将对象保存起来，就想起了java的序列化机制，下面我就谈谈关于自己对序列化的理解。


### 什么是序列化 

Java平台允许我们在内存中创建和使用对象，当JVM运行时，这些对象才有可能存在，一旦JVM停止运行，这些对象将不复存在，可是实际应用中，我们经常需要在JVM停止运行后，还需要保存这些对象，用于以后重新读取。序列化机制就为我们解决了这个问题。简单的说，序列化就是将对象写进文本之类的地方。


### 序列化怎么实现 

对于java来说，实现序列化的方式有两种，一种是实现Serializable,另一种是实现Externalizable。

<!--more-->

#### Serializable 

Serializable只是一个标记型接口，只要一个类实现java.lang.io.Serializable,那么它就会被序列化，下面就写个类User，以后都会围绕这个类进行修改

``` java
	import java.io.Serializable;

	public class User implements Serializable{

		private static final long serialVersionUID = 1L;
		
		private int id;
		private static transient String name;
		private transient int age;
		private static String address;
		
		public User(int id, String name, int age,String address) {
			super();
			this.id = id;
			User.name = name;
			this.age = age;
			User.address = address;
		}
		
		public final int getId() {
			return id;
		}
		public final void setId(int id) {
			this.id = id;
		}
		public final String getName() {
			return name;
		}
		public final void setName(String name) {
			User.name = name;
		}
		public final int getAge() {
			return age;
		}
		public final void setAge(int age) {
			this.age = age;
		}
		public final String getAddress() {
			return address;
		}
		public final void setAddress(String address) {
			User.address = address;
		}
		@Override
		public String toString() {
			return "User [id=" + id + ", name=" + User.name + ", age=" + age + ", address="
					+ User.address +  "]";
		}
		
		
	}
```
		
TestSerial，是一个简单的序列化程序，它先将一个User对象保存到文件user.txt中，然后再从该文件中读出被存储的User对象，并打印该对象。
		
``` java
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.FileOutputStream;
	import java.io.ObjectInputStream;
	import java.io.ObjectOutputStream;


	public class TestSerial {
		public static void main(String[] args) throws Exception{                              
			File file = new File("user.txt");
			ObjectOutputStream oos = new ObjectOutputStream(
					new FileOutputStream(file));
			User user = new User(1,"sakaraffy",100,"HEB");
			
			//修改全局变量address,name   
			user.setAddress("WH");
			user.setName("cy");
			
			oos.writeObject(user);
			oos.close();
			
			ObjectInputStream ois = new ObjectInputStream(
					new FileInputStream(file));
			User newUser = (User) ois.readObject();
			ois.close();
			//被static修饰的属性每次都会去全局区内取值
			System.out.println(newUser);
		}
	}

```
		
输出结果 ：
``` java
User [id=1, name=cy, age=0, address=WH]
```
		
下面就来解析一下代码
- 如果User没有实现Serializable接口,会抛出NotSerializableException。
- User类所有的非基本数据类型属性必须实现Serializable接口，会抛出NotSerializableException。
- transient 关键字 ： 		
	当属性被transient关键词所修饰序列化机制就会忽略该字段，这就是为什么age=0的原因。
- static 关键字 : 
我们知道，被static关键词所修饰的属性，那么它就不再是每个对象所独立拥有，而是整个类所拥有，而序列化所解决的是保存对象数据，所以被static修饰无论是否再被transient所修饰，该字段都不会被序列化，这就是name和address属性产生该结果的原因				
		
#### Externalizable 

从上面，我们可以知道，实现了Serializable接口，它会默认地将所有字段都序列化，可是有时候我们只需要序列化个别字段，那么这个时候由该怎么办呢？
- 将所有不被序列化的字段加上tranisent修饰词
- 实现 Externalizable 接口
		
``` java 
	import java.io.Externalizable;
	import java.io.IOException;
	import java.io.ObjectInput;
	import java.io.ObjectOutput;

	public class User implements Externalizable{

		private static final long serialVersionUID = 1L;

		private int id;
		private static transient String name;
		private transient int age;
		private static String address;

		public User() {
		// TODO Auto-generated constructor stub
		}

		public User(int id, String name, int age,String address) {
			super();
			this.id = id;
			User.name = name;
			this.age = age;
			User.address = address;
		}

		public final int getId() {
			return id;
		}
		public final void setId(int id) {
			this.id = id;
		}
		public final String getName() {
			return name;
		}
		public final void setName(String name) {
			User.name = name;
		}
		public final int getAge() {
			return age;
		}
		public final void setAge(int age) {
			this.age = age;
		}
		public final String getAddress() {
			return address;
		}
		public final void setAddress(String address) {
			User.address = address;
		}
		@Override
		public String toString() {
			return "User [id=" + id + ", name=" + User.name + ", age=" + age + ", address="
				+ User.address +  "]";
		}

		@Override
		public void writeExternal(ObjectOutput out) throws IOException {
			// TODO Auto-generated method stub
			out.writeInt(id);
			out.writeInt(age);
			out.writeObject(name);
		}

		@Override
		public void readExternal(ObjectInput in) throws IOException,
			ClassNotFoundException {
			// TODO Auto-generated method stub
			id = in.readInt();
			age = in.readInt();
			name = (String) in.readObject();
		}
	}
```

再次运行TestSerial,输出结果：
``` java
User [id=1, name=sakaraffy, age=100, address=HEB]
```

下面就来解析一下代码
- 实现Externalizable接口，必须实现readExternal(ObjectInput in)和writeExternal(ObjectOutput out)
- 实现Externalizable接口，必须要有一个空的构造方法，否则会抛出java.io.InvalidClassException。
- 因为该对象是一个属性，一个属性保存，所以无论该属性是否被transient关键词修饰，都会被序列化。
- readExternal(ObjectInput in)和writeExternal(ObjectOutput out)操作字段的顺序一定要一致

### readResolve() 单例模式中使用

当反序列化，也就是说从保存文件读取对象的时候，如果实体里写了这个方法，读取对象时，以这个方法为主，即有新对象产生，原对象就会被吸附。

``` java
	private Object readResolve() throws ObjectStreamException{
		return new User(2,"cy",18,"Anlu");
	}
```

在User中加入以上代码，在运行TestSerial,输出结果：
``` java
User [id=2, name=cy, age=18, address=Anlu]
```

### 序列化对象存储方式 

``` java
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.FileOutputStream;
	import java.io.ObjectInputStream;
	import java.io.ObjectOutputStream;


	public class TestSerial {
	public static void main(String[] args) throws Exception{
		File file = new File("user.txt");
		ObjectOutputStream oos = new ObjectOutputStream(
				new FileOutputStream(file));
		User user = new User(1,"sakaraffy",100,"HEB");
		oos.writeObject(user);
		oos.flush();
		System.out.println(file.length());
		
		//java 在写入对象的时候，如果对象已经写入，则此次修改操作无效，还是原对象的内容      
		user.setId(0);
		oos.writeObject(user);
		oos.flush();
		System.out.println(file.length());
		
		oos.close();
		
		ObjectInputStream ois = new ObjectInputStream(
				new FileInputStream(file));
		
		User newUser1 = (User) ois.readObject();
		User newUser2 = (User) ois.readObject();
		
		ois.close();
		System.out.println(newUser1);
		System.out.println(newUser2);
		System.out.println(newUser1 == newUser2);
	}
}
```

输出结果：
``` java
48
53
User [id=1, name=sakaraffy, age=100, address=HEB]
User [id=1, name=sakaraffy, age=100, address=HEB]
true
```

下面就来解析一下代码：		  序列化机制为了节省磁盘空间，具有特定的存储规则，当写入文件的为同一对象时，并不会再将对象的内容进行存储，而只是再次存储一份引用，增加的5 字节新增引用
反序列化时，恢复引用关系，使得清单 3 中的 u1和 u2 指向唯一的对象，二者相等，输出 true。该存储规则极大的节省了存储空间。