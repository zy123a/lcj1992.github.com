---
layout: post
title: 常用并发类的使用
categories: java
tags: CountDownLatch,CyclicBarrier,Exchanger,Semaphore
---

* TOC
{:toc}

## 线程间通信

### CountDownLatch

1. 某个线程等待其他线程执行完任务后，它才执行。
2. 不可重用
3. await无限等待或等到超时，知道count为0.

eg. 多线程获取所有待执行task，获取完成之后执行之。具体见代购proxy MessageReadTask

### CyclicBarrier

1. 一组线程`互相`等待至某个状态,然后执行对应的barrierCommand
2. 可重用（回环栅栏),

eg. CountDownLatch的例子同样适用。

### Exchanger


### Semaphore

### ThreadLocal
