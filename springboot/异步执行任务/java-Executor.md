# java ExecutorService，Future
因为直接创建thread比较耗时，所有jdk提供了Executors用来先创建线程池再使用

Java并发包提供了一套“异步任务执行服务”机制，将“任务的提交”和“任务的执行”相分离。

任务执行服务涉及到的基本接口：

- Runnable和Callable：表示要执行的异步任务
- Executor和ExecutorService：表示执行服务
- Future：表示异步任务的结果
- Runnable和Callable都是接口，Runnable没有返回结果，而Callable有，Runnable不会抛出异常，而Callable会。
- Executor表示最简单的执行服务

    ```java
    public interface Executor {
        void execute(Runnable command);
    }
    ```
- ExecutorService扩展了Executor，定义了更多服务
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

## 一个简单例子
```java
package com.javakk;

import java.util.concurrent.*;

public class FutureTest {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newCachedThreadPool(); // 创建线程池
        Future<String> future = executor.submit(() ->{
            Thread.sleep(200); // 模拟接口调用，耗时200ms
            return "hello world";
        });
        // 在输出下面异步结果时主线程可以不阻塞的做其他事情
        // TODO 其他业务逻辑

        System.out.println("异步结果:"+future.get()); //主线程获取异步结果
        // 或者通过下面轮询的方式
        // while(!future.isDone());
    }
}

```

## 缺点

简单的说我有一个任务，提交给了Future，Future替我完成这个任务，这期间我可以去做别的事情。一段时间之后，我再从Future取出结果。

上面的代码有2个地方需要注意，在15行不建议使用future.get()方式，而应该使用future.get(long timeout, TimeUnit unit), 尤其是在生产环境一定要设置合理的超时时间，防止程序无限期等待下去。另外就是要考虑异步任务执行过程中报错抛出异常的情况，需要捕获future的异常信息。

通过代码可以看出一些简单的异步场景可以使用Future解决，但是对于结果的获取却不是很方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式相当于把异步变成了同步，显然和异步编程的初衷相违背，轮询的方式又会浪费CPU资源。

Future没有提供通知的机制，就是回调，我们无法知道它什么时间完成任务。

而且在复杂一点的情况下，比如多个异步任务的场景，一个异步任务依赖上一个异步任务的执行结果，异步任务合并等，Future无法满足需求。

# CompletableFuture
jdk8或以上的版本，那可以直接使用CompletableFuture类来实现异步编程。

Java8新增的CompletableFuture类借鉴了Google Guava的ListenableFuture，它包含50多个方法，默认使用forkJoinPool线程池，提供了非常强大的Future扩展功能，可以帮助我们简化异步编程的复杂性，结合函数式编程，通过回调的方式处理计算结果，并且提供了转换和组合CompletableFuture的多种方法，可以满足大部分异步回调场景。

虽然方法很多但有个特征：

- 以Async结尾的方法签名表示是在异步线程里执行，没有以Async结尾的方法则是由主线程调用
- 如果参数里有Runnable类型，则没有返回结果，即纯消费的方法
- 如果参数里没有指定executor则默认使用forkJoinPool线程池，指定了则以指定的线程池来执行任务

以下列出一些常用的pattern
示例代码只关注核心功能，如果要实际使用需要考虑超时和异常情况，大家需要注意。

## pattern1 task2等待task1执行完再执行

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
			CompletableFuture<String> f2 = f1.thenApplyAsync( (t) -> {
					System.out.format("thread name:(%s) -- task2\n", Thread.currentThread().getName());
					return  t + " world";
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
## pattern2 组合 : thenComposeAsync

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

## 合并 : thenCombineAsync

## 二选一 : applyToEitherAsync

## allOf / anyOf

## other

虽然说CompletableFuture更适合I/O场景，但使用时一定要结合具体业务，比如说有些公共方法处理异步任务时需要考虑异常情况，这时候使用CompletableFuture.handle(BiFunction<? super T, Throwable, ? extends U> fn)更合适，handle方法会处理正常计算值和异常，因此它可以屏蔽异常，避免异常继续抛出。

CompletableFuture还有一个坑需要注意：如果线上流量比较大的情况下会出现响应缓慢的问题。

因为CompletableFuture默认使用的线程池是forkJoinPool，当时对一台使用了CompletableFuture实现异步回调功能的接口做压测，通过监控系统发现有大量的ForkJoinPool.commonPool-worker-* 线程处于等待状态，进一步分析dump信息发现是forkJoinPool的makeCommonPool问题，如下图：

看到这大家应该清楚了，如果在项目里没有设置java.util.concurrent.ForkJoinPool.common.parallelism的值，那么forkJoinPool线程池的线程数就是(cpu-1)，我们测试环境的机器是2核，这样实际执行任务的线程数只有1个，当有大量请求过来时，如果有耗时高的io操作，势必会造成更多的线程等待，进而拖累服务响应时间。

解决方案一个是设置java.util.concurrent.ForkJoinPool.common.parallelism这个值(要在项目启动时指定)，或者指定线程池不使用默认的forkJoinPool。

forkJoinPoll线程池不了解的可以看下这篇文章：线程池ForkJoinPool简介