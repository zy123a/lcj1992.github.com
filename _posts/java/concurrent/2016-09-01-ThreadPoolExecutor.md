---
layout: post
title: ThreadPoolEexcutor
categories: java
tags: ThreadPoolExecutor FutureTask RunnableFuture ExecutorService
---

* [ctl](#ctl)
* [FutureTask](#FutureTask)
* [Worker](#Workder)
* [workQueue](#workQueue)
* [invokeAll](#invokeAll)

首先需要明确的是ThreadPoolExecutor虽然是1.5的东西，但是底层还是一些jdk1.0的东西，Thread和Runnable，然后加以管理.

### ctl

线程池最重要的一个状态了。一个int型，包含两个域：1.  线程池数量(workCount后29位) 2.  运行状态(runState前3位)   
 
    // 后29位保存线程池数量
    private static final int COUNT_BITS = Integer.SIZE - 3;
    // 最大的线程池数量是2的29次方-1
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // runState is stored in the high-order bits
    // 111000..000
    private static final int RUNNING    = -1 << COUNT_BITS;
    // 000000..000
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    // 001000..000
    private static final int STOP       =  1 << COUNT_BITS;
    // 010000..000
    private static final int TIDYING    =  2 << COUNT_BITS;
    // 011000..000
    private static final int TERMINATED =  3 << COUNT_BITS;hreadPoolExecutor几个重要方法为突破口
  
    // 提供ctl、runState、workCount的转换方法
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
    private static int workerCountOf(int c)  { return c & CAPACITY; }
    private static int ctlOf(int rs, int wc) { return rs | wc; }

### FutureTask {#FutureTask}

因为所有提交给线程池的Runnable或者Callable都会被newTaskFor()包装成一个FutureTask，所以有必要提下。

    FutureTask<V> implement RunnableFuture<V>
    RunnableFuture<V> extend Runnable,Future<V> 

首先看FutureTask的继承体系，继承自RunnableFuture，实现了Runnable和Future接口。

也就是说FutureTask可以作为Runnable扔给线程池去执行，然后通过Future.get()拿到返回的结果。

然后看下FutureTask几个关键的域：关联的`callable`，`outcome`运行结果、`runner`运行这个futureTask的线程.

其流程:通过runner.start()方法启动线程，调用run()方法、run()方法里会调用callable()的call方法，并把结果保存在outcome中。然后通过future.get()拿到outcome中的结果。

    /** The underlying callable; nulled out after running */
    // 所关联的callable 
    private callable<V> callable;
    
    /** The result to return or exception to throw from get() */
    // 结果
    private Object outcome; // non-volatile, protected by state reads/writes
    
    /** The thread running the callable; CASed during run() */
    // 运行这个futureTask的线程
    private volatile Thread runner;

### Worker {#Workder}

Worker继承自AbstractQueuedSynchronizer和Runnable。

     /** Thread this worker is running in.  Null if factory fails. */
     final Thread thread;
 
     /** Initial task to run.  Possibly null. */
     Runnable firstTask;
 
     /** Per-thread task counter */
     volatile long completedTasks;
 
1. 包装Runnable为一个RunnableFuture，实际类型为FutureTask，持有一个Callable的引用。
4. workQueue
5. workers

invokeAll(callableList)

1. Callable如何实现返回结果的？

通过FutureTask来实现。
    a.FutureTask继承自RunnableFuture.
    b.有两个很重要的域Callable、outcome.
    c.在其run方法中调用Callable的call方法，并把结果保存在outcome中。

2. invokeAll怎么实现等待所有结果都返回？

    for (int i = 0, size = futures.size(); i < size; i++) {
        Future<T> f = futures.get(i);
        // 先判定类是否完成。
        if (!f.isDone()) {
            try {
                // 如果没有完成，这里会自旋阻塞当前结果。
                f.get();
            } catch (CancellationException ignore) {
            } catch (ExecutionException ignore) {
            }
        }
    }

3. worker继承自Runnable，内部的thread初始化时，newThread(this)把worker注入进自己的Thread。执行时，run() -> runWorker().

    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }
