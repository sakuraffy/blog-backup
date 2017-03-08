---
title: 工厂模式(Simple Factory + Factory Method + Abstract Factory)
date: 2016-11-29 19:06:13
tags:
	- 设计模式
---
Java有一个很重要的特性就是多态，简单来说多态就是引用变量所指向的引用对象在编译期时无法确定，只用在运行期才能动态绑定。而工厂模式与其有着相似的特征，只是工厂模式更加复杂化。一般来说工厂模式一般有三部分组成，分别为简单工厂，工厂方法，抽象工厂，其抽象程度也依次递增

<!-- more -->

### 简单工厂(Simple Factory)

#### 简单工厂涉及的角色
- 工厂类角色：这是本模式的核心，含有一定的判断逻辑
- 抽象产品角色：它一般是具体产品继承的父类或者实现的接口
- 具体产品角色：工厂类所创建的对象就是此角色的实例

#### 简单工厂代码实现
业务逻辑 ： 小米和努比亚的手机都实现打电话的功能
##### 抽象产品
``` java
package cn.sakuraffy.factory;

public interface Callable {
	public void call();
}
```

##### 具体产品--Nubia的实现
``` java
package cn.sakuraffy.factory;

public class Nubia implements Callable {

	@Override
	public void call() {
		System.out.println("Nubia call");
	}
}
```

##### 具体产品--Mi的实现
``` java
package cn.sakuraffy.factory;

public class Mi implements Callable {

	@Override
	public void call() {
		System.out.println("Mi call");
	}
}
```

##### 工厂
``` java
package cn.sakuraffy.factory;

public class SimpleFactory {
	public static Callable getCall(String name) {
		if("Nubia".equals(name)) {
			return new Nubia();
		} else if ("Mi".equals(name)) {
			return new Mi();
		} else {
			return null;
		}
	}
}
```
##### 客户端测试
``` java
package cn.sakuraffy.factory;

public class SimpleFactoryClient {
	public static void main(String[] args) {
		Callable phone = SimpleFactory.getCall("Mi");
		phone.call();
	}
}
```
输出结果 ：
``` java
	Mi call
```

### 工厂方法(Factory Method)
#### 工厂方法涉及的角色
在简单工厂的基础上抽象工厂类，增加如下角色 ：
- 抽象工厂角色：这是工厂方法模式的核心，它与应用程序无关。是具体工厂角色必须实现的接口或者必须继承的父类
- 具体工厂角色：它含有和具体业务逻辑有关的代码。由应用程序调用以创建对应的具体产品的对象

#### 工厂方法代码实现
业务逻辑 ： 小米和努比亚都有自己的工厂生产手机
##### 抽象工厂
``` java
package cn.sakuraffy.factory;

public interface CallFactory {
	public Callable product();
}
```
##### 具体工厂--Nubia工厂
``` java
package cn.sakuraffy.factory;

public class NubiaFactory implements CallFactory {

	@Override
	public Callable product() {
		return new Nubia();
	}
}
```
##### 具体工厂--Mi工厂
``` java
package cn.sakuraffy.factory;

public class MiFactory implements CallFactory {

	@Override
	public Callable product() {
		return new Mi();
	}
}
```
##### 客户端测试
``` java
package cn.sakuraffy.factory;

public class FactoryMethodClient {
	public static void main(String[] args) {
		FactoryCall factory = new NubiaFactory();
		Callable phone = factory.product();
		phone.call();
	}
}
```
输出结果 ：
``` java
	Nubia call
```

### 抽象工厂(Abstract Factory)
在工厂方法的基础再多添加抽象产品角色及其实现。由多个抽象产品自由组合形成一个产品族

#### 抽象工厂代码实现
业务逻辑 ： 
- 手机由手机本身和其装载的软件组成
- 软件播放器有爱奇艺和Potplayer

##### 抽象产品
``` java
package cn.sakuraffy.factory;

public interface Playable {
	public void play();
}
```

##### 具体产品--QiYiPlayer的实现
``` java
package cn.sakuraffy.factory;

public class QiYiPlayer implements Playable {

	@Override
	public void play() {
		System.out.println("QiYiplayer play");
	}
}
```

##### 具体产品--PotPlayer的实现
``` java
package cn.sakuraffy.factory;

public class Potplayer implements Playable {

	@Override
	public void play() {
		System.out.println("potplayer play");
	}
}
```

##### PlayFactory
``` java
package cn.sakuraffy.factory;

public interface PlayFactory {
	public Playable product();
}
```

##### PlayFactory--PotFactory
``` java
package cn.sakuraffy.factory;

public class PotFactory implements PlayFactory{

	@Override
	public Playable product() {
		return new Potplayer();
	}
}
```

##### PlayFactory--QiYiFactory
``` java
package cn.sakuraffy.factory;

public class QiYiFactory implements PlayFactory{

	@Override
	public Playable product() {
		return new QiYiPlayer();
	}
}
```

##### 产品族--Phone
``` java
package cn.sakuraffy.factory;

public class Phone {
	protected Callable caller;
	protected Playable player;
	
	public void show() {
		System.out.println("phone info : " + this.getClass().getName());
		caller.call();
		player.play();
	}
}
```
##### 产品族--DailyPhone
``` java
package cn.sakuraffy.factory;

public class DailyPhone extends Phone {
	public DailyPhone() {
		player = new QiYiFactory().product();
		caller = new MiFactory().product();
	}
}
```
##### 产品族-SpecialPhone
``` java
package cn.sakuraffy.factory;

public class SpecialPhone extends Phone {
	public SpecialPhone() {
		caller = new NubiaFactory().product();
		player = new PotFactory().product();
	}
}
```

##### 客户端测试
``` java
package cn.sakuraffy.factory;

public class AbstractFactoryClient {
	public static void main(String[] args) {
		Phone phone = new SpecialPhone();
		phone.show();
	}
}
```
输出结果 ：
``` java
	phone info : cn.sakuraffy.factory.SpecialPhone
	Nubia call
	potplayer play

```

### 工厂模式的优缺点
至此工厂模式的实现就介绍完了，下面就谈谈工厂模式的适用情景与优缺点

#### 简单工厂优缺点
##### 优点
- 工厂类含有必要的判断逻辑，可以决定在什么时候创建哪一个产品类的实例
- 通过引入配置文件，可以在不修改任何客户端代码的情况下更换和增加新的具体产品类

##### 缺点 
- 过度依赖工厂类，一旦出问题，整个系统都将受影响
- 使用了静态工厂方法，造成工厂角色无法形成基于继承的等级结构

#### 工厂方法优缺点
##### 优点
- 增加新的产品类时无须修改现有系统
- 封装了产品对象的创建细节，系统具有良好的灵活性和可扩展性

##### 缺点 
- 增加新产品的同时需要增加新的工厂，增加系统复杂度

#### 抽象工厂优缺点
##### 优点
- 隔离了具体类的生成，
- 实现高内聚低耦合

##### 缺点 
- 在添加新的产品对象时，难以扩展抽象工厂来生产新种类的产品
- 开闭原则的倾斜性（增加新的工厂和产品族容易，增加新的产品等级结构麻烦）
