# Executor框架

java并发处理基本上在java.util.concurrent包中

因为直接创建thread比较耗时，所有jdk提供了Executors用来先创建线程池再使用

Java并发包提供了一套“异步任务执行服务”机制，将“需要执行的异步任务”和“任务的执行”相分离。

## Executor的框架结构由三大部分构成

- **需要执行的异步任务**

   需要实现接口：Runnable接口或Callable接口。Runnable没有返回结果，而Callable有，Runnable不会抛出异常，而Callable会。

- **任务的执行**

   是通过线程池中的线程来执行异步任务，是通过Executor接口以及继承自Executor的ExecutorService接口。

  Executors类是一个工厂类，提供了一系列静态工厂方法来创建不同的线程池，返回一个ExecutorService，开启异步任务，从这个类开始。

- **异步任务结果**

  接口Future，Future提供了三种功能

  - 判断任务是否完成
  - 能够中断任务
  - 能够获取任务执行结果

  FutureTask类

  ​	是 Callable or Runnable + Future 感觉没有多大用处

  CompletableFuture类（是Future的改进版本，异步任务的结果通过CallBack的方式通知，有点像js的异步任务用法）

  

## Executor的框架类图

![preview](.\java-Executor.assets\executer_class.png)



## 一个简单例子

```java
package com.javakk;

import java.util.concurrent.*;

public class FutureTest {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newCachedThreadPool(); // 1.创建线程池
        Future<String> future = executor.submit(() ->{ // 2. 创建Callable
            Thread.sleep(200); // 模拟异步任务，耗时200ms
            return "hello world";
        });
        // 在输出下面异步结果时主线程可以不阻塞的做其他事情
        // 3. TODO 其他业务逻辑
	    // 4. 主线程获取异步结果,3秒内还没获取到结果，会抛出TimeoutException异常。
        System.out.println("异步结果:"+future.get(3, TimeUnit.SECONDS)); 
        // 或者通过下面轮询的方式
        /* 
        while(!future.isDone()) {
        	Thread.sleep(1000); // sleep 1秒
    	}
    	 System.out.println("异步结果:"+future.get()); 
    	*/
    }
}

```

## 

## Callable

和 Runnable相比，Callable拥有以下功能

- call 方法可以有返回值。返回值的类型即是 Callable 接口传递进来的V类型。
- call 方法可以声明抛出异常

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

## 

## Executor

表示最简单的执行服务，定义了一个接收Runnable对象的方法execute

```java
public interface Executor {
    void execute(Runnable command);
}
```



## ExecutorService

扩展了Executor，定义了终止任务、提交任务、跟踪任务返回结果等方法。
一个ExecutorService是可以关闭的，关闭之后它将不能再接收任何任务。

对于不再使用的ExecutorService，应该将其关闭以释放资源。

ExecutorService是在线程池上运行任务的。

```java
public interface ExecutorService extends Executor { 
    // 对于Callable，任务最终有个返回值
    <T> Future<T> submit(Callable<T> task);
    // Runnable是没有返回值的，但可以提供一个结果，在异步任务结束时返回
    <T> Future<T> submit(Runnable task, T result);
    // 最终返回null
    Future<?> submit(Runnable task);
    // 不再接受新任务，但已提交的任务会继续执行，即使任务还未执行
    void shutdown();
    // 不接口新任务，且会终止已提交但未执行的任务，对于正在执行的任务，一般会调用线程的interrupt方法尝试中断
    // 返回已提交但尚未执行的任务列表  
    List<Runnable> shutdownNow();   

    // showdown和showdownNow不会阻塞，它们返回后不代表所有任务都已结束，但isShutdown会返回ture
    boolean isShutdown();

    // 除非首先调用shutdown或shutdownNow，否则isTerminated永不为true。
    // 当调用shutdown()方法后，并且所有提交的任务完成后返回为true;
    // 当调用shutdownNow()方法后，成功停止后返回为true;
    boolean isTerminated(); 

    // 等待所有任务结束，可以限定等待的时间 
    // 如果超时前所有任务都结束了，则返回true，否则返回false
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;

    // 等待所有任务完成，返回Future列表 
    // 每个Future的isDone都返回true，不过isDone不代表任务执行成功了，可能是被取消了 
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) 
                    throws InterruptedException; 
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                long timeout, TimeUnit unit) throws InterruptedException;
    // 只要有一个任务返回成功了，就会返回该任务的结果，其他任务会被取消 
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException; 
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;     
}
```



## ThreadPoolExecutor

线程池的核心实现类，用来执行被提交的任务。使用工厂类Executor来创建。Executor可以创建三种类型的ThreadPoolExcutor。

### ThreadPoolExecutor的构造器

```java
 /**
 * Creates a new {@code ThreadPoolExecutor} with the given initial
 * parameters.
 *
 * @param corePoolSize the number of threads to keep in the pool, even
 *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
 * @param maximumPoolSize the maximum number of threads to allow in the
 *        pool
 * @param keepAliveTime when the number of threads is greater than
 *        the core, this is the maximum time that excess idle threads
 *        will wait for new tasks before terminating.
 * @param unit the time unit for the {@code keepAliveTime} argument
 * @param workQueue the queue to use for holding tasks before they are
 *        executed.  This queue will hold only the {@code Runnable}
 *        tasks submitted by the {@code execute} method.
 * @param threadFactory the factory to use when the executor
 *        creates a new thread
 * @param handler the handler to use when execution is blocked
 *        because the thread bounds and queue capacities are reached
 * @throws IllegalArgumentException if one of the following holds:<br>
 *         {@code corePoolSize < 0}<br>
 *         {@code keepAliveTime < 0}<br>
 *         {@code maximumPoolSize <= 0}<br>
 *         {@code maximumPoolSize < corePoolSize}
 * @throws NullPointerException if {@code workQueue}
 *         or {@code threadFactory} or {@code handler} is null
 */   
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

**构造器的各个参数说明**：

- **corePoolSize**：核心线程数，核心线程会一直存活，即使没有任务需要处理。但如果设置了`allowCoreThreadTimeOut` `为 true 则核心线程也会超时退出。

- **maximumPoolSize**：最大线程数，线程池中可允许创建的最大线程数。

- **keepAliveTime**：当线程池中的线程数大于核心线程数，那些多余的线程空闲时间达到keepAliveTime后就会退出，直到线程数量等于corePoolSize。如果设置了`allowCoreThreadTimeout`设置为true，则所有线程均会退出直到线程数量为0。

- **unit**：keepAliveTime参数的时间单位

- **workQueue**：在任务执行前用来保存任务的 **阻塞**队列。这个队列只会保存通过execute方法提交到线程池的Runnable任务。

  存放线程的任务队列有以下几种：

  - ArrayBlockingQueue:基于数组结构的有界阻塞队列，遵循FIFO。
  - LinkedBlockingQueue：基于链表结构的无界阻塞队列，遵循FIFO。吞吐量比ArrayBlockingQueue高。
  - SynchronousQueue: 不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态。吞吐量比LinkedBlockingQueue高。
  - PriorityBlockingQueue：一个具有优先级的无阻塞队列

- **threadFactory**：线程池创建新线程时使用的factory。默认使用defaultThreadFactory创建线程。

- **handle** ：饱和策略。当线程池的线程数已达到最大，并且任务队列已满时来处理被拒绝任务的策略。默认使用ThreadPoolExecutor.AbortPolicy，任务被拒绝时将抛出RejectExecutorException

  饱和策略有以下几种：

  - AbortPolicy:直接抛出异常。
  - CallerRunsPolicy: 使用调用者所在线程来运行任务。
  - DiscardOldestPolicy: 丢弃队列里最近的一个任务，并执行当前任务。
  - DiscardPolicy: 不处理，丢弃掉。

除此之外，ThreadPoolExecutor还有两个个常用的参数设置：

- **allowCoreThreadTimeout**：是否允许核心线程空闲退出，默认值为false。
- **queueCapacity**：任务队列的容量。

ThreadPoolExecutor线程池的逻辑结构图

![image-20220629152538220](D:\github\knowhow\java\并发\java-Executor.assets\image-20220629152538220.png)

### 线程池执行任务的流程

1. 当线程数小于核心线程数时，创建线程。

2. 当线程数大于等于核心线程数，且任务队列未满时，将任务放入任务队列。

3. 当线程数大于等于核心线程数，且任务队列已满

   - 若线程数小于最大线程数，创建线程

   - 若线程数等于最大线程数，抛出异常，拒绝任务。

## ScheduledExecutorService

ScheduledExecutorService是一个线程池，用来在指定延时之后执行或者以固定的频率周期性的执行提交的任务。它包含了4个方法。

```java
public interface ScheduledExecutorService extends ExecutorService {
    /**
     * 在指定delay（延时）之后，执行提交Runnable的任务，返回一个ScheduledFuture，
     * 任务执行完成后ScheduledFuture的get()方法返回为null，ScheduledFuture的作用是可以cancel任务
     */
    public ScheduledFuture<?> schedule(Runnable command,
                                       long delay, TimeUnit unit);

    /**
     * 在指定delay（延时）之后，执行提交Callable的任务，返回一个ScheduledFuture
     */
    public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                           long delay, TimeUnit unit);

    /**
     * 提交一个Runnable任务延迟了initialDelay时间后，开始周期性的执行该任务，每period时间执行一次
     * 如果任务异常则退出。如果取消任务或者关闭线程池，任务也会退出。
     * 如果任务执行一次的时间大于周期时间，则任务执行将会延后执行，而不会并发执行
     */
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);

    /**
     * 提交一个Runnable任务延迟了initialDelay时间后，开始周期性的执行该任务，以后
       每两次任务执行的间隔是delay
     * 如果任务异常则退出。如果取消任务或者关闭线程池，任务也会退出。
     */
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);
}
```



## Executors

Executors类是一个工厂类，提供了一系列静态工厂方法来创建不同的ExecutorService或 ScheduledExecutorService实例。

**`注意`**: 如果线程池中的一个线程执行一个任务失败，并导致线程结束，系统会创建一个新的线程去执行队列里后续的任务，不会因为前面的任务有异常导致后面无辜的任务无法执行。

1. #### newSingleThreadExecutor

   它的特点在于工作线程数目被限制为 1，操作一个无界的工作队列，所以它保证了所有任务的都是被顺序执行，最多会有一个任务处于活动状态，并且不允许使用者改动线程池实例，因此可以避免其改变线程数目。。

   **适用于需要保证顺序地执行各个任务，并且在任意时间点不会有多个线程的应用场景。**

2. #### newFixedThreadPool

   创建一个可重用的固定线程数量的线程池。即corePoolSize=线程池中的线程数= maximumPoolSize。

   重用指定数目（nThreads）的线程，其背后使用的是无界的工作队列，任何时候最多有 nThreads 个工作线程是活动的。这意味着，如果任务数量超过了活动队列数目，将在工作队列中等待空闲线程出现；如果有工作线程退出，将会有新的工作线程被创建，以补足指定的数目 nThreads。

   **适用于为了满足资源管理的需求，而需要限制当前线程数量的应用场景，它适用于负载比较重的服务器。**

3. #### newCachedThreadPool

   创建一个不限制线程数量的动态线程池。

   它是一种用来处理大量短时间工作任务的线程池，具有几个鲜明特点：它会试图缓存线程并重用，当无缓存线程可用时，就会创建新的工作线程；如果线程闲置的时间超过 60 秒，则被终止并移出缓存；长时间闲置时，这种线程池，不会消耗什么资源。其内部使用 SynchronousQueue 作为工作队列。

   **适用于执行很多短期异步任务的小程序或者是负载较轻的服务器。**

4. #### newSingleThreadScheduledExecutor

   指定延时之后执行或者以固定的频率周期性的执行提交的任务。

5. #### newScheduledThreadPool

   指定延时之后执行或者以固定的频率周期性的执行提交的任务。

   

## Future

Future主要用来对具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过`get()`方法获取执行结果，`get()`方法会阻塞直到任务返回结果。

```java
public interface Future<V> {


    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();
	// 用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回
    V get() throws InterruptedException, ExecutionException;
	// 用来获取执行结果，如果在指定时间内，还没获取到结果，会抛出TimeoutException异常。
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```



## FutureTask

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}

public class FutureTask<V> implements RunnableFuture<V> {
    public FutureTask(Callable<V> callable) {
	}
    public FutureTask(Runnable runnable, V result) {
    }
}
```

提供了直接通过Thread获取线程任务结果的手段

```java
FutureTask futureTask = new FutureTask(task);
new Thread(futureTask).start();
while (!futureTask.isDone()) {
    System.out.println("future task is running......");
    Thread.sleep(1000);
}
```

## CompletableFuture

### Future缺点 

简单的说我有一个任务，提交给了Future，Future替我完成这个任务，这期间我可以去做别的事情。一段时间之后，我再从Future取出结果。不建议使用future.get()方式，而应该使用future.get(long timeout, TimeUnit unit), 尤其是在生产环境一定要设置合理的超时时间，防止程序无限期等待下去。另外就是要考虑异步任务执行过程中报错抛出异常的情况，需要捕获future的异常信息。通过代码可以看出一些简单的异步场景可以使用Future解决，但是对于结果的获取却不是很方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式相当于把异步变成了同步，显然和异步编程的初衷相违背，轮询的方式又会浪费CPU资源。Future没有提供通知的机制，就是回调，我们无法知道它什么时间完成任务。而且在复杂一点的情况下，比如多个异步任务的场景，一个异步任务依赖上一个异步任务的执行结果，异步任务合并等，Future无法满足需求。为了应对Future不足，jdk8或以上的版本有了CompletableFuture类来实现异步编程。

### 概要

CompletableFuture基于JDK1.8的流式编程以及Lambda表达式等实现一元操作符、异步性以及事件驱动编程模型，通过回调的方式处理异步任务结果，可以用来实现多线程的串行关系，并行关系，聚合关系。它的灵活性和更强大的功能是Future无法比拟的。

CompletableFuture提供了方法大约有50多个，单纯一个个记忆，是很麻烦的，因此将其划分为以下几类：

### 创建类

- completeFuture 可以用于创建默认返回值
- runAsync 异步执行，无返回值
- supplyAsync 异步执行，有返回值
- anyOf 任意一个执行完成，就可以进行下一步动作
- allOf 全部完成所有任务，才可以进行下一步任务

### 状态取值类

- join 合并结果，等待
- get 合并等待结果，可以增加超时时间;get和join区别，join只会抛出unchecked异常，get会返回具体的异常
- getNow 如果结果计算完成或者异常了，则返回结果或异常；否则，返回valueIfAbsent的值
- isCancelled
- isCompletedExceptionally
- isDone

### 控制类 用于主动控制CompletableFuture的完成行为

- complete
- completeExceptionally
- cancel

### 接续类 用于注入回调行为 CompletableFuture 最重要的特性

- thenApply, thenApplyAsync
- thenAccept, thenAcceptAsync
- thenRun, thenRunAsync
- thenCombine, thenCombineAsync
- thenAcceptBoth, thenAcceptBothAsync
- runAfterBoth, runAfterBothAsync
- applyToEither, applyToEitherAsync
- acceptEither, acceptEitherAsync
- runAfterEither, runAfterEitherAsync
- thenCompose, thenComposeAsync
- whenComplete, whenCompleteAsync
- handle, handleAsync
- exceptionally

虽然方法很多但有个特征：

- 以Async结尾的方法，都是异步方法，对应的没有Async则是同步方法由主线程调用，一般都是一个异步方法对应一个同步方法。

- 以Async后缀结尾的方法，都有两个重载的方法，一个是使用内容的forkjoin线程池，一种是使用自定义线程池

- 以run开头的方法，其入口参数一定是无参的，并且没有返回值，类似于执行Runnable方法。
- 以supply开头的方法，入口也是没有参数的，但是有返回值
- 以Accept开头或者结尾的方法，入口参数是有参数，但是没有返回值
- 以Apply开头或者结尾的方法，入口有参数，有返回值
- 带有either后缀的方法，表示谁先完成就消费谁

- 如果参数里有Runnable类型，则没有返回结果，即纯消费的方法

- 如果参数里没有指定executor则默认使用ForkJoinPool线程池，指定了ThreadPoolExecutor则以指定的线程池来执行任务

  默认使用 ForkJoinPool.commonPool()，commonPool是一个会被很多任务 共享 的线程池，比如同一 JVM 上的所有 CompletableFuture、并行 Stream 都将共享 commonPool，commonPool 设计时的目标场景是运行 非阻塞的 CPU 密集型任务，为最大化利用 CPU，其线程数默认为 CPU 数量 - 1。

以下列出一些常用的pattern
示例代码只关注核心功能，如果要实际使用需要考虑超时和异常情况，大家需要注意。

### pattern1 task2等待task1执行完再执行

在下面的代码中我们执行3个task
异步任务task2需要异步任务task1完成的结果才能执行，但对于我们的主线程来说，无须等到f1返回结果后再调用函数f2，即不会阻塞主流程，继续执行task3，而是告诉CompletableFuture当执行完了f1的方法再去执行f2，只有当需要最后的结果时再获取。

```java
package com.cy.utils;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

import org.junit.Test;

public class TaskThreadTest {
	
	
	@Test
	public void patter1() {
		try {
			// 执行task1
			CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> {
					System.out.format("thread name:(%s) -- task1\n", Thread.currentThread().getName());
					// 模拟执行任务
					try {
						Thread.sleep(2);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					return "hello";
				}
			);
			// f2依赖f1的结果做转换
			CompletableFuture<String> f2 = f1.thenApplyAsync( (result) -> {
					System.out.format("thread name:(%s) -- task2\n", Thread.currentThread().getName());
					return  result + " world";
				}
			);
			
			// 执行task3
			CompletableFuture.runAsync( () ->
				System.out.format("thread name:(%s) -- task3\n", Thread.currentThread().getName())
			);
		
			System.out.println("异步结果:" + f2.get());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
		}

}
```
输出结果
```
thread name:(ForkJoinPool.commonPool-worker-1) -- task1
thread name:(ForkJoinPool.commonPool-worker-2) -- task3
thread name:(ForkJoinPool.commonPool-worker-1) -- task2
异步结果:hello world
```
### pattern2 组合 : thenComposeAsync

```Java
@Test
public void patter2() {
    try {
        // 执行task1
        CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> {
                System.out.format("thread name:(%s) -- task1\n", Thread.currentThread().getName());
                // 模拟执行任务
                try {
                    Thread.sleep(5000);
                    System.out.println("task1 end");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return "hello";
            }
        );
        // f2依赖f1的结果做转换
        CompletableFuture<String> f2 = f1.thenComposeAsync( (t) -> {
                System.out.format("thread name:(%s) -- task2\n", Thread.currentThread().getName());
                // 模拟执行任务
                try {
                    Thread.sleep(2000);
                    System.out.println("task2 end");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return CompletableFuture.supplyAsync(() ->
                        t + " world"
                );
            }
        );
        
        // 执行task3
        CompletableFuture.runAsync( () -> {
                System.out.format("thread name:(%s) -- task3\n", Thread.currentThread().getName());
                // 模拟执行任务
                try {
                    Thread.sleep(2000);
                    System.out.println("task3 end");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        );
    
        System.out.format("thread name:(%s) -- 异步结果:%s\n", Thread.currentThread().getName(), f2.get());
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```
输出结果
```
thread name:(ForkJoinPool.commonPool-worker-1) -- task1
thread name:(ForkJoinPool.commonPool-worker-2) -- task3
task3 end
task1 end
thread name:(ForkJoinPool.commonPool-worker-2) -- task2
task2 end
thread name:(main) -- 异步结果:hello world
```

合并 : thenCombineAsync

### 二选一 : applyToEitherAsync

### allOf  

必须等待所有的future全部完成才可以

 CompletableFuture.allOf(cfuture1,cfuture2);

###  anyOf

只要有一个完成，则完成，有一个抛出异常，则携带异常

CompletableFuture. anyOf(cfuture1,cfuture2);

### other

虽然说CompletableFuture更适合I/O场景，但使用时一定要结合具体业务，比如说有些公共方法处理异步任务时需要考虑异常情况，这时候使用CompletableFuture.handle(BiFunction<? super T, Throwable, ? extends U> fn)更合适，handle方法会处理正常计算值和异常，因此它可以屏蔽异常，避免异常继续抛出。

CompletableFuture还有一个坑需要注意：如果线上流量比较大的情况下会出现响应缓慢的问题。

因为CompletableFuture默认使用的线程池是forkJoinPool，当时对一台使用了CompletableFuture实现异步回调功能的接口做压测，通过监控系统发现有大量的ForkJoinPool.commonPool-worker-* 线程处于等待状态，进一步分析dump信息发现是forkJoinPool的makeCommonPool问题，如下图：

看到这大家应该清楚了，如果在项目里没有设置java.util.concurrent.ForkJoinPool.common.parallelism的值，那么forkJoinPool线程池的线程数就是(cpu-1)，我们测试环境的机器是2核，这样实际执行任务的线程数只有1个，当有大量请求过来时，如果有耗时高的io操作，势必会造成更多的线程等待，进而拖累服务响应时间。

解决方案一个是设置java.util.concurrent.ForkJoinPool.common.parallelism这个值(要在项目启动时指定)，或者指定线程池不使用默认的forkJoinPool。

forkJoinPoll线程池不了解的可以看下这篇文章：线程池ForkJoinPool简介