---
layout: post
title: Java的Executor框架和线程池实现原理
tags: 多线程,Executor
categories: java
--- 

【toc】
#### 1、Executor框架  
<img src="https://zy123a.github.io/zy-blog/images/java/Executor框架.png" width="500" height="400" alt="image"/>  
    
#### 2、相关接口类介绍  
##### 2.1、Executor  
```java
public interface Executor {
    void execute(Runnable command);
}
```  
Executor作为Executor框架最基础的接口，只定义了一个执行Runnable的execute方法，其没有实现类，只有一个
继承其的接口ExecutorService     

##### 2.2、ExecutorService   
```java
public interface ExecutorService extends Executor {

    /**
    * 关闭方法，执行之前提交的方法，之后不再接受任务
     */
    void shutdown();

    /**
    * 暂停所有的等待任务，并返回该任务列表
     */
    List<Runnable> shutdownNow();

    /**
    * 执行器是否已关闭
     */
    boolean isShutdown();

    /**
    * 关闭后是否所有等待任务均已完成
     */
    boolean isTerminated();

    /**
    * 中断
     */
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    /**
    * 提交一个Runable任务，result要返回的结果
     */
    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    /**
    *  执行所有给定的任务，当所有任务完成，返回保持任务状态和结果的Future列表 
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    /**
    * 执行给定的任务，当所有任务完成或超时期满时（无论哪个首先发生），返回保持任务状态和结果的 Future 列表。 
     */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    /**
    * 执行给定的任务，如果某个任务已成功完成（也就是未抛出异常），则返回其结果。
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    /**
    * 执行给定的任务，如果在给定的超时期满前某个任务已成功完成（也就是未抛出异常），则返回其结果
     */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```   

##### 2.3、Executors   

Executors负责生成各种ExecutorService的实例：
* newFixedThreadPool() :固定线程数，ExecutorService负责创建一个固定数量线程的线程池，并行执行的线程
池数量不变，当前线程执行完成后可以执行另一个任务，具体实现new ThreadPoolExecutor(nThreads, nThreads,
                                                    0L, TimeUnit.MILLISECONDS,
                                                    new LinkedBlockingQueue<Runnable>());   
                                                    
* newCachedThreadPool():（可缓存线程池）ExecutorService 创建一个线程池，按需创建新线程，就是有任务时才创建，
空闲线程保存60s，当前面创建的线程可用时，则重用它们，具体实现new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                                                  60L, TimeUnit.SECONDS,
                                                                  new SynchronousQueue<Runnable>());  
                                                                  
* newScheduledThreadPool():创建一个线程池，用来执行给定执行时间的任务，具体实现：new ScheduledThreadPoolExecutor(corePoolSize)   

* SingleThreadExecutor()：单线程执行器，线程池里只有一个线程，依次执行队列中的任务，具体实现：new FinalizableDelegatedExecutorService
                                                                       (new ThreadPoolExecutor(1, 1,
                                                                                               0L, TimeUnit.MILLISECONDS,
                                                                                               new LinkedBlockingQueue<Runnable>()));   
##### 2.4、Runnable，Callable,Future，FutureTast  

* Runnable  
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```   

* Callable   
```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```   

* Future  
```java
public interface Future<V> {
    /**
    * 尝试取消一个任务，如果这个任务不能被取消（通常是因为已经执行完了），返回false，否则返回true。
    * mayInterruptIfRunning用于确定是否允许取消正在执行的任务
    */
    boolean cancel(boolean mayInterruptIfRunning);
    /**
    * 返回代表的任务是否在完成之前被取消了
    */
    boolean isCancelled();

    /**
    * 如果任务已经完成，返回true 
    */
    boolean isDone();
    
    /**
    * 方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回
    */
    V get() throws InterruptedException, ExecutionException;
    
    /**
    * 用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。
    */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```
                                                                                                
Runnable接口和Callable接口的实现类，都可以被ThreadPoolExecutor和ScheduledThreadPoolExecutor执行，
他们之间的区别是Runnable不会返回结果，而Callable可以返回结果。Executors可以把一个Runnable对象转换成Callable对象：
public static <T> Callable<T> callable(Runnable task, T result);
public static Callable<Object> callable(Runnable task);
实现上通过一个RunnableAdapter适配器直接适配过来的；  

* FutureTask:  
通常使用FutureTask来处理我们的任务。FutureTask类同时又实现了Runnable接口，所以可以直接提交给Executor执行;   
```
xecutorService executor = Executors.newSingleThreadExecutor();     
FutureTask<String> future =     
       new FutureTask<String>(new Callable<String>() {//使用Callable接口作为构造参数     
         public String call() {     
           //真正的任务在这里执行，这里的返回值类型为String，可以为任意类型     
       }});     
executor.execute(future);     
//在这里可以做别的任何事情     
try {     
    result = future.get(5000, TimeUnit.MILLISECONDS); //取得结果，同时设置超时执行时间为5秒。同样可以用future.get()，不设置执行超时时间取得结果     
} catch (InterruptedException e) {     
    futureTask.cancel(true);     
} catch (ExecutionException e) {     
    futureTask.cancel(true);     
} catch (TimeoutException e) {     
    futureTask.cancel(true);     
} finally {     
    executor.shutdown();     

```    

#### 3、线程池原理解析   
##### 3.1、参数介绍  
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)

* corePoolSize:线程基本大小，当提交一个任务到线程池时，线程会创建一个线程来执行任务，即使其他空闲的基本线程能创建线程也会
创建线程，等到到需要执行的任务数大于线程池基本大小corePoolSize时就不再创建。  

* maximumPoolSize：线程池允许最大线程数。如果阻塞队列满了，并且已经创建的线程数小于最大线程数，则线程池会再创建新的线程执
行。因为线程池执行任务时是线程池基本大小满了，后续任务进入阻塞队列，阻塞队列满了，在创建线程。   

* keepAliveTime：空闲线程的保持存活时间；  

* unit：线程存活保持时间的单位；   

* threadFactory：线程创建工厂，创建是如果传null，则使用默认工厂；    

* workQueue：用于保存等待执行的任务的阻塞队列。常用队列如下：   
    1、ArrayBlockingQueue;数组结构的有界阻塞队列，先进先出FIFO；  
    2、LinkedBlockingQueue;链表结构的无界阻塞队列，先进先出FIFO，Executors.newFixedThreadPool使用这个队列；   
    3、SynchronousQueue;不存储元素的阻塞队列，就是每次插入操作必须等到另一个线程调用移除操作，静态方法Executors.newCachedThreadPool
    使用这个方法；   

* handler（饱和策略）：表示当拒绝处理任务时的策略。当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处
理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常，其他策略如下：
    1、ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
    2、ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
    3、ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
    4、ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
   
##### 3.2、线程池执行流程    

<img src="https://zy123a.github.io/zy-blog/images/java/线程池执行步骤.png" width="500" height="400" alt="image"/>  

1、首先线程池判断基本线程池是否已满（< corePoolSize ？）？没满，创建一个工作线程来执行任务。满了，则进入下个流程。  
2、其次线程池判断工作队列是否已满？没满，则将新提交的任务存储在工作队列里。满了，则进入下个流程。  
3、最后线程池判断整个线程池是否已满（< maximumPoolSize ？）？没满，则创建一个新的工作线程来执行任务，满了，则交给饱和策略来处理这个任务。   

##### 3.3、线程池的关闭

• shutdown()：不会立即终止线程池，而是再也不会接受新的任务，要等所有任务缓存队列中的任务都执行完后才终止  
• shutdownNow()：立即终止线程池，再也不会接受新的任务，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务   

##### 3.4、线程池的状态   
      RUNNING 允许状态；
      SHUTDOWN 关闭状态；
      stop 停止状态；
      TERMINATED 终结状态；
    
1，当创建线程池后，初始时，线程池处于RUNNING状态；  
  
2，如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕，最后终止；  
  
3，如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务，返回没有执行的任务列表；  
  
4，当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。  

                                                                                              
                                                                                               
