---
title: 并发编程（二十一）生产者-消费者模式
date: 2016-08-03 18:15:10
tags:
	- 并发编程
---
生产者与消费者是一个很典型的多线程问题，下面就看看生产者-消费者模式的基本角色
- 任务数据 ：生产者和消费者所操作的数据结构
- 生产者 ：用于提交用户请求，提取用户任务，并装入内存缓冲区
- 消费者 ：在内存缓冲区中提取并处理任务
- 内存缓冲区 ：缓存生产者任务，供消费者使用
- Client ：使用客户端

<!--more-->

下面展示一个实例
任务数据类型定义 ：
``` java
	public static class Data {
		private static int count = 0;
		private int id;
		public Data() {
			id = count++ ;
		}
		public final int getId() {
			return id;
		}
		public final void setId(int id) {
			this.id = id;
		}
		
	}
```
生产者定义 ：
``` java
	public static class Producer implements Runnable {
		private BlockingQueue<Data> bq;
		
		public Producer(BlockingQueue<Data> bq) {
			super();
			this.bq = bq;
		}

		@Override
		public void run() {
			Random r = new Random();
			while(true) {
				try {
					Thread.sleep(r.nextInt(1000));
					Data data = new Data();
					if(!bq.offer(data, 1, TimeUnit.SECONDS)) {
						System.out.println("produce fail");
					}else {
						System.out.println(Thread.currentThread().getName() + 
								" produce " + data.getId());
					}
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
					Thread.currentThread().interrupt();
				}
			}
		}
	}
```
消费者定义 ：
``` java
	public static class Consumer implements Runnable {
		private BlockingQueue<Data> bq;
		
		public Consumer(BlockingQueue<Data> bq) {
			super();
			this.bq = bq;
		}

		@Override
		public void run() {
			Random r = new Random();
			while(true) {
				try {
					Thread.sleep(r.nextInt(1000));
					Data data = bq.take();
					if (data != null) {
						System.out.println(Thread.currentThread().getName() + " consume "
								+ data.getId());
					}
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
	}
```
Client :
``` java
	public static void main(String[] args) {
		BlockingQueue<Data> bq = new ArrayBlockingQueue<>(10);
		ExecutorService es = Executors.newCachedThreadPool();
		for(int i = 0; i < 5; i++) {
			es.execute(new Producer(bq));
			es.execute(new Consumer(bq));
		}
	}
```
输出结果 ：
``` java
pool-1-thread-3 produce 0
pool-1-thread-7 produce 1
pool-1-thread-3 produce 2
pool-1-thread-7 produce 3
pool-1-thread-8 consume 0
pool-1-thread-2 consume 1
pool-1-thread-7 produce 4
pool-1-thread-6 consume 2
pool-1-thread-6 consume 3
pool-1-thread-3 produce 5
pool-1-thread-1 produce 6
pool-1-thread-3 produce 7
pool-1-thread-2 consume 4
pool-1-thread-5 produce 8
```
