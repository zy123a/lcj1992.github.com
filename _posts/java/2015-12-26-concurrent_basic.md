---
layout: post
title: java并发基础
categories: java
tags: concurrent
---

*   [thread](#thread)
*   [ThreadLocal](#thread_local)
*   [CopyOnWriteArrayList](#CopyOnWriteArrayList)
*   [ReadWriteLock](#ReadWriteLock)
*   [ConcurrentHashMap](#ConcurrentHashMap)
*   [volatile_static_threadLocal](#volatile_static_threadLocal)
*   [ThreadExecutor](#ThreadExecutor)

### thread

1.  run()和start()
2.  join()/join(long millis) 等待当前线程die或者超时

<h3 id="thread_local">ThreadLocal</h3>

1.  相当于线程范围的局部变量，Map<Thread,object>。
2.  线程1set改变ThreadLocal的变量，不会影响线程2get该变量
3.  业务处理完成之后一定要释放ThreadLocal的变量，对于使用线程池的，线程会复用
4.  ThreadLocal只是本线程有效,InheritableThreadLocal对由本线程创建的子线程也有效.


<h3 id="CopyOnWriteArrayList">CopyOnWriteArrayList</h3>

<h3 id="ReadWriteLock">ReadWriteLock</h3>

<h3 id="ConcurrentHashMap">ConcurrentHashMap</h3>

<h3 id="volatile_static_threadLocal">volatile vs static  vs  ThreadLocal</h3>

1.  volatile 不同线程的同一对象之间共享
2.  static  同一线程的不同对象之间共享
3.  static volatile 不同线程的不同对象之间共享[他人的讨论](http://stackoverflow.com/questions/2423622/volatile-vs-static-in-java)
4.  threadLocal 同一线程之间共享，不同线程之间互斥,声明为ThreadLocal，通常需要声明为static,一个是per thread per object,一个是per thread
[他人的讨论](http://stackoverflow.com/questions/2784009/why-should-java-threadlocal-variables-be-static)

![volatile_static](/images/java/volatile_static.png)

<h3 id="ThreadExecutor">ThreadExecutor</h3>

#### 官方注释

        @param corePoolSize the number of threads to keep in the pool, even
        if they are idle, unless{@code allowCoreThreadTimeOut}is set
        线程池中的线程数量，即使他们是空闲的，除非allowCoreThreadTimeOut设置了

        @param maximumPoolSize the maximum number of threads to allow in the pool
        线程池中的最大数量

        @param keepAliveTime when the number of threads is greater than
        the core, this is the maximum time that excess idle threads will
        wait for new tasks before terminating.
        当线程池中线程数量超过coreSize时，这是超出的线程能等待新任务的最长时间
        也就是说keepAliveTime是在线程池中线程数量大于coreSize小于maxSize时起作用。

        @param unit the time unit for the {@code keepAliveTime} argument
        时间的单位

        @param workQueue the queue to use for holding tasks before they are
        executed. This queue will hold only the {@code Runnable} tasks submitted by the
        {@code execute} method.

#### 处理流程

![处理流程](/images/java/executor.jpg)

1.当coreSize未满时，当有新任务submit时，即使有空闲线程，也会新起线程。如果调用了线程池的prestartAllcoreThreads方法，线程池会提前创建并启动所有基本线程。

2.当coreSize满了，会添加到队列中

3.当队列也满了，但maxSize未满，会新起线程

4.超过maxSize，按照策略执行。

#### 各线程池

*   Executors.newFixedThreadPool(size)
*   ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
线程池数量固定，无界队列，将保持所有线程running，直到被显式的终止，长时任务建议使用
*   Executors.newCachedThreadPool()
*   ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>())
线程池数量无界，SynchronousQueue一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue。
大量短时异步任务建议使用。
*   Executors.newSingleThreadScheduledExecutor()

参考：

[1]<http://www.infoq.com/cn/articles/java-threadPool>

[2]<http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Executors.html>

[3]<http://ifeve.com/java-concurrency-thread-directory/>


