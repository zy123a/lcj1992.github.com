---
layout: post
title: ReentrantLock源码解析
categories: java
tags: AQS ReentrantLock CLH AbstractQueuedSynchronizer
---
* [概述](#overview)
* [非公平锁](#nonFairLock)
    * [加锁过程](#ReentrantLock_lock)
        * [NonfairLock#lock](#NonfairLock_lock)
        * [AbstractQueuedSynchronizer#acquire](#AbstractQueuedSynchronizer_acquire)
        * [Sync#nonfairTryAcquire](#Sync_nonfairTryAcquire)
        * [AbstractQueuedSynchronizer#addWaiter](#AbstractQueuedSynchronizer_addWaiter)
        * [AbstractQueuedSynchronizer#enq](#AbstractQueuedSynchronizer_enq)
        * [AbstractQueuedSynchronizer#acquireQueued](#AbstractQueuedSynchronizer_acquireQueued)
    * [解锁过程](#ReentrantLock_unlock)
        * [Sync#tryRelease](#Sync_tryRelease)
* [公平锁](#FairLock)
    * [加锁过程](#ReentrantLock_lock_fair)
        * [FairLock#lock](#FairLock_lock)
        * [FairSync#tryAcquire](#FairSync_tryAcquire)

## 概述  {#overview}

ReentrantLock的结构体如下

![reentrantLock](/images/java/reentrantLock.png)

1.  内部类`Sync`继承自`AbstractQueuedSynchronizer` 参见[aqs](/2016/08/24/aqs)
2.  Sync两个子类 `NonfairSync`实现非公平锁、`FairSync`实现公平锁
2.  使用AQS的独占API

## 非公平锁 {#nonFairLock}

### 加锁过程 {#ReentrantLock_lock}

#### NonfairLock#lock {#NonfairLock_lock}

ReentrantLock#lock() -> NonfairLock#lock()

     // 执行加锁，立即闯入，如果失败的话执行正常的acquire
    final void lock() {
        // CAS操作保证操作的原子性
        // state声明为 volatile保证内存可见性,state=0表明可以争用。
        if (compareAndSetState(0, 1))
            // 如果state 0->1设置成功,设置当前线程为同步器的属主线程
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

然后acquire(1)

#### AbstractQueuedSynchronizer#acquire {#AbstractQueuedSynchronizer_acquire}

AbstractQueuedSynchronizer#acquire(arg)

    public final void acquire(int arg) {
        // 以独占模式acquire失败
        if (!tryAcquire(arg) &&
        // 包装一个当前线程的Node，入队列，然后尝试去acquire
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            // 如果失败，中断当前线程
            selfInterrupt();
    }

#### Sync#nonfairTryAcquire {#Sync_nonfairTryAcquire}

NonfairSync#tryAcquire(arg) -> Sync#nonfairTryAcquire(arg)

    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        // 获取state标志位，如果为0，没有线程持有锁
        if (c == 0) {
            // CAS尝试去获取锁。
            if (compareAndSetState(0, acquires)) {
                // 如果获取成功，设置属主线程为当前线程。
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 如果不为0，判断当前线程是否为属主线程，这里实现可重入
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            // 设置state的值，state的值表明当前线程的重入数。
            setState(nextc);
            return true;
        }
        return false;
    }

如果tryAcquire失败，把当前线程包装为一个Node入队列，并再次尝试acquire

包装当前线程为一个node

#### AbstractQueuedSynchronizer#addWaiter {#AbstractQueuedSynchronizer_addWaiter}

AbstractQueuedSynchronizer#addWaiter()

    private Node addWaiter(Node mode) {
       Node node = new Node(Thread.currentThread(), mode);
       // Try the fast path of enq; backup to full enq on failure
       Node pred = tail;
       // 如果队列不为空
       if (pred != null) {
           // node的先驱节点设置为tail
           node.prev = pred;
           // 设置当前节点为tail
           if (compareAndSetTail(pred, node)) {
               // 原有的tail的后继节点指向node
               pred.next = node;
               return node;
           }
       }
       // 如果pred为null或者设置当前节点为tail失败，表明有别的线程并发入队列。执行enq方法，自旋方式入队列。
       enq(node);
       return node;
   }

#### AbstractQueuedSynchronizer#enq {#AbstractQueuedSynchronizer_enq}

AbstractQueuedSynchronizer#enq()

    private Node enq(final Node node) {
        // 自旋方式入队列
        for (;;) {
            Node t = tail;
            // 如果队列为空，初始化head和tail节点
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }

入队列后，然后尝试从队列中acquire

#### AbstractQueuedSynchronizer#acquireQueued {#AbstractQueuedSynchronizer_acquireQueued}

AbstractQueuedSynchronizer#acquireQueued()

    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                // 如果先驱节点为head，尝试acquire
                if (p == head && tryAcquire(arg)) {
                    // acquire成功，将head节点置为node
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 如果先驱节点不是head，或者是head，但是尝试acquire失败
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

AbstractQueuedSynchronizer#setHead()

    private void setHead(Node node) {
        head = node;
        // 头节点不绑定线程
        node.thread = null;
        node.prev = null;
    }

#### AbstractQueuedSynchronizer#shouldParkAfterFailedAcquire {#AbstractQueuedSynchronizer_shouldParkAfterFailedAcquire}

AbstractQueuedSynchronizer#shouldParkAfterFailedAcquire(Node pred,Node node)

     /** waitStatus value to indicate thread has cancelled */
     // 当前线程已经被取消
     static final int CANCELLED =  1;
     /** waitStatus value to indicate successor's thread needs unparking */
     // 后继节点线程等待被触发
     static final int SIGNAL    = -1;
     /** waitStatus value to indicate thread is waiting on condition */
     // 线程等待条件
     static final int CONDITION = -2;
     /**
      * waitStatus value to indicate the next acquireShared should
      * unconditionally propagate
      */
     // 节点状态需要向后传播
     static final int PROPAGATE = -3;

    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            // 只有当前节点的前一个节点为SIGNAL时，当前节点才能被挂起
            return true;
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }

### 解锁过程 {#ReentrantLock_unlock}

解锁过程较简单点

#### Sync#tryRelease {#Sync_tryRelease}

ReentrantLock#unlock() -> AbstractQueuedSynchronizer#release() -> Sync#tryRelease(releases)

    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        // 可重入，只有减为0时，才是free，别的线程才能抢得到。
        if (c == 0) {
            free = true;
            // 设置属主线程为null
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

## 公平锁 {#FairLock}

### 加锁过程 {#ReentrantLock_lock_fair}

整体流程和非公平加锁是一样的。

#### FairLock#lock {#FairLock_lock}

    final void lock() {
        // if (compareAndSetState(0, 1))
        //    setExclusiveOwnerThread(Thread.currentThread());
        // else
        // 必须严格按照队列顺序来
        acquire(1);
    }

#### FairSync#tryAcquire {#FairSync_tryAcquire}

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 比非公平加锁，多了这个判定条件 hasQueuedPredecessors，必须没有前驱节点才让你acquire，否则的话，好好等着，轮到你再来。
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

加锁过程和非公平锁是一样一样的。

## 参考 {#ref}

[x86的指令集]<http://www.felixcloutier.com/x86/>

[AQS,老爷子的论文]<http://gee.cs.oswego.edu/dl/papers/aqs.pdf>

[AbstractQueuedSynchronizer的实现分析（上）]<http://www.infoq.com/cn/articles/jdk1.8-abstractqueuedsynchronizer>

[AbstractQueuedSynchronizer的实现分析（下）]<http://www.infoq.com/cn/articles/java8-abstractqueuedsynchronizer>
