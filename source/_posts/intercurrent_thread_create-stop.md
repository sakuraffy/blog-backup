---
title: 并发编程（一）线程的创建与销毁
date: 2016-06-02 22:38:43
tags: 
	- 并发编程
---

### 线程创建

#### 继承java.lang.Thread

``` java
public class CreateThread extends Thread{
	public void run() {
		while(true) {
			System.out.println("HelloWorld");
		}
	}
	
	public static void main(String[] args) {
		new CreateThread().start();
	}
}
```

<!--more-->

#### 实现java.lang.Runnable

``` java
public class CreateThread implements Runnable{
	public void run() {
		while(true) {
			System.out.println("HelloWorld");
		}
	}
	
	public static void main(String[] args) {
		new Thread(new CreateThread()).start();
	}
}
```

java.lang.Thread 为我们提供了一个Thread(Runnable targe)的构造方法，而查看源码会发现Thread的run()的实现如下 :

``` java 
@Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```
也就是说start0()中对这两种不同实现方式的run()的调用也是不一样的

``` java
	public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }
	 public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
	private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize)
```
extends Thread这种方式target = null

#### 两种创建方法的比较
extends Thread : 简单，易理解
implements Runnable ： java单继承，节约继承资源，更符合面向对象的思想

#### 关于run()和start()的比较

run() : 在当前线程调用对象方法
start() : 重新开启一个线程调用对象方法，

### 线程销毁

####stop() 已经被废弃

stop()是粗暴的将线程线程终止，这就有可能引起线程安全问题。

``` java
public class StopThread {
	
	private static User user = new User();
	
	public static class User {
		private String name;
		private int id;
		
		public User() {
			id = 0;
			name = "0";
		}
		
		public final String getName() {
			return name;
		}
		public final void setName(String name) {
			this.name = name;
		}
		public final int getId() {
			return id;
		}
		public final void setId(int id) {
			this.id = id;
		}
		
		@Override
		public String toString() {
			return "User [name=" + name + ", id=" + id + "]";
		}
	
	}
	
	public static class WriteThread extends Thread {
		
		@Override
		public void run() {
			while(true) {	
				synchronized (user) {
					int num = (int) (System.currentTimeMillis()/1000);
					user.setId(num);
					try {
						Thread.sleep(100);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					user.setName(String.valueOf(num));
				}
				Thread.yield();
			}
		}
	}
	
	public static class ReadThread extends Thread {
		@Override
		public void run() {
			while(true) {
				synchronized (user) {
					while(user.getId() != Integer.parseInt(user.getName())) {
						System.out.println(user);
						break;
					}
				}
				Thread.yield();
			}
		}
	}
	
	public static void main(String[] args) {
		System.out.println(System.currentTimeMillis()/1000);
		new ReadThread().start();
		while(true) {
			WriteThread write = new WriteThread();
			write.start();
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			write.stop();
		}
	}
}
```

截取部分结果如下
``` java
User [name=1468850323, id=1468850324]
User [name=1468850324, id=1468850325]
```

产生原因 ：
Thread.stop()会在线程结束时，直接终止线程，并释放所有锁，而这些锁恰恰是维持对象一致性的。如果此时写线程写数据写到一半时，被强行终止，那么对象就会被写坏。同时，由于锁被释放，读线程顺理成章的读到不一致的数据。

解决方法 ： 
我们设置一个标记变量来表示线程是否需要退出。
``` java
public static class WriteThread extends Thread {
	private boolean stop = false;
	
	public void stopMe() {
		stop = true;
	}
	
	@Override
	public void run() {
		while(true) {
			
			if(stop) {
				System.out.println("exit");
				break;
			}
			
			synchronized (user) {
				int num = (int) (System.currentTimeMillis()/1000);           
				user.setId(num);
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				user.setName(String.valueOf(num));
			}
			Thread.yield();
		}
	}
}
```