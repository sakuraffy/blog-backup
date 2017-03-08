---
title: 并发编程（十四）线程池（一）
date: 2016-07-21 00:13:41
tags:
	- 并发编程
---
### 什么是线程池

在应用程序中增加线程个数会提高明显提高系统的吞吐量，但是滥用线程则会拖垮应用程序。我们知道在线程的使用中，线程的创建占了很大一部分时间。为了提高性能，我们是不是可以先创建一批线程出来，然后需要的时候，从中取出，用完之后，再放回去。存放这些线程的就是线程池

### JDK对线程池的支持

<!--more-->

与线程池有关的类或接口有：
 
Interface : 
``` java
	public interface Executor {
		//实现对Runnable对象的调度
		void execute(Runnable command);
	}
	public interface ExecutorService extends Executor {
		void shutdown();
		//有返回值的调度
		<T> Future<T> submit(Callable<T> task);
		Future<?> submit(Runnable task);
	}
	// 处理与时间有关的调度
	public interface ScheduledExecutorService extends ExecutorService {
		public ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);
		
		public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period,TimeUnit unit);
                                                                                          
		public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay,long delay,TimeUnit unit);
```
Class
``` java
	//线程池
	public class ThreadPoolExecutor extends AbstractExecutorService {}
	public abstract class AbstractExecutorService implements ExecutorService {}
	//线程工厂
	public class Executors {
		public static ExecutorService newFixedThreadPool(int nThreads) {}
		public static ExecutorService newSingleThreadExecutor()　{}
		public static ExecutorService newCachedThreadPool()　{}
		public static ScheduledExecutorService newSingleThreadScheduledExecutor()　{}
		public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)　{}
	}
```
下面就线程工厂各种产生线程池的方法加以说明 ：
- newFixedThreadPool() : 产生一个固定线程数量的线程池，数量始终不变
- newSingleThreadExecutor() : 产生只有一个线程的线程池，多余一个，任务会保存在任务队列中，待线程空闲，以先进先出的顺序执行队列中的任务
- newCachedThreadPool() : 产生一个线程数量根据实际情况调整的线程池，任务进入，优先使用空闲线程，没有则创造线程，该线程完成后，返回线程池进行复用
- newScheduledThreadPool() : 产生一个确定线程数量的ScheduledExecutorService
- newSingleThreadScheduledExecutor() : 产生一个线程数量为1的ScheduledExecutorService

下面展示一个测试程序 
``` java
	public class ThreadPool implements Runnable {
		@Override
		public void run() {
			System.out.println(System.currentTimeMillis()/1000 + " "                      
					+ Thread.currentThread().getName());
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		
		public static void main(String[] args) {
			ExecutorService es1 = Executors.newFixedThreadPool(2);
			ExecutorService es2 = Executors.newSingleThreadExecutor();
			ExecutorService es3 = Executors.newCachedThreadPool();
			
			ThreadPool tp = new ThreadPool();
			
			for(int i = 0; i < 3; i++) {
				es1.execute(tp);
				//es2.execute(tp);
				//es3.execute(tp);
			}
		}
	}
```
输出结果 ：
``` java
// 使用es1 ：
1469521145 pool-1-thread-2
1469521145 pool-1-thread-1
1469521146 pool-1-thread-2

// 使用es2 ：
1469521325 pool-2-thread-1
1469521326 pool-2-thread-1
1469521327 pool-2-thread-1

// 使用es3 ：
1469521363 pool-3-thread-2
1469521363 pool-3-thread-3
1469521363 pool-3-thread-1
```

下面展示一个关于时间调度的测试程序
``` java
	public class TestScheduledExecutorService {
		public static void main(String[] args) {
			ScheduledExecutorService ses = Executors.newScheduledThreadPool(5);           
			ses.scheduleWithFixedDelay(new Runnable() {
				
				@Override
				public void run() {
					System.out.println(System.currentTimeMillis()/100 + " "
							+ Thread.currentThread().getName());
					try {
						Thread.sleep(1500);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}, 0, 2, TimeUnit.SECONDS);
			
		/*ses.scheduleAtFixedRate(new Runnable() {
					
					@Override
					public void run() {
						System.out.println(System.currentTimeMillis()/100 + " "
								+ Thread.currentThread().getName());
						try {
							Thread.sleep(1500);
						} catch (InterruptedException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
					}
				}, 0, 2, TimeUnit.SECONDS);*/
		}
	}
```
 
输出结果截取部分如下 ：

``` java
// ses.scheduleWithFixedDelay() :
14695233533 pool-1-thread-1
14695233568 pool-1-thread-1
14695233603 pool-1-thread-2

// ses.scheduleAtFixedRate() :
14695234096 pool-1-thread-1
14695234116 pool-1-thread-1
14695234136 pool-1-thread-2
14695234156 pool-1-thread-1
```

### 关于scheduleAtFixedRate()和scheduleWithFixedDelay()的区别
 
- 对于scheduleAtFixedRate()来说，频率是一定的，它是以上个任务开始为起点，之后的period时间，调度下一次任务
- 而scheduleWithFixedDelay(),间隔时间是一定的，它是以上一个任务结束为起点

### 线程池的内部实现

我们通过查看源码发现无论是newSingleThreadExecutor(),Executors.newSingleThreadExecutor()还是Executors.newCachedThreadPool()都是使用调用ThreadPoolExecutor的构造方法
``` java
	public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
	public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
	public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
那么我们看一下这个最原始的构造方法
``` java
	 public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```
函数的参数含义如下 ：
- corePoolSize ：线程池中的线程数量
- maximumPoolSize ：线程池中的最大线程数量
- keepAliveTime ：线程数量超过corePoolSize，多余线程的最大存活时间
- unit ：keepAliveTime的时间单位
- workQueue ： 任务队列
- threadFactory ： 线程工厂
- handler ：拒绝策略

这里给出ThreadPoolExecutor的核心调度代码：
``` java
	public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated                         
         * and so reject the task.
         */
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```

### 线程池的拒绝策略

当队列中以排满，再也塞不下其它任务，那么我们就需要提供一套机制来处理这个问题。
JDK内置了四种拒绝策略
- AbortPolicy策略 ： 该策略直接抛出异常，阻止系统正常工作
- CallerRunsPolicy策略 ： 只要线程池未关闭，运行当前被丢弃的任务
- DiscardOledestPolicy策略 : 丢弃最老的任务
- DiscardPolicy策略 ；不做任何处理

同样的，我们也可以自己写处理方案，只需要实现RejectedExecutionHandler接口
``` java
	public interface RejectedExecutionHandler {
		void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
	}
```
下面展示一个测试程序
``` java
	public class RejectPolicy implements Runnable {
	
		@Override
		public void run() {
			System.out.println(System.currentTimeMillis()/1000 + " "
					+ Thread.currentThread().getName());
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		public static void main(String[] args) throws InterruptedException {
			RejectPolicy rp = new RejectPolicy();
			ExecutorService es = new ThreadPoolExecutor(5, 5, 
					0L, TimeUnit.SECONDS, 
					new LinkedBlockingQueue<Runnable>(10),
					Executors.defaultThreadFactory(),
					new RejectedExecutionHandler() {
						
						@Override
						public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
							System.out.println(r.toString() + "is discard");
						}
					});
			for(int i = 0; i < Integer.MAX_VALUE; i++) {
				es.submit(rp);
				Thread.sleep(100);
			}
		}
		
	}
```
截取部分输出如下 ：
``` java
1469533427 pool-1-thread-1
1469533427 pool-1-thread-2
1469533428 pool-1-thread-3
1469533428 pool-1-thread-4
1469533428 pool-1-thread-5
java.util.concurrent.FutureTask@55f96302is discard
java.util.concurrent.FutureTask@3d4eac69is discard
java.util.concurrent.FutureTask@42a57993is discard
java.util.concurrent.FutureTask@75b84c92is discard
java.util.concurrent.FutureTask@6bc7c054is discard
1469533428 pool-1-thread-1
```