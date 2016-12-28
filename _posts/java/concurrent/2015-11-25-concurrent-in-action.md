---
layout: post
title: 线程间通信方式
categories: java
tags: CountDownLatch,CyclicBarrier,Exchanger,Semaphore
---

* TOC
{:toc}

## 线程间通信

### CountDownLatch：某个线程等待其他线程

1. 某个线程等待其他线程执行完任务后，它才执行。
2. 不可重用
3. await无限等待或等到超时，直到count为0.

eg. 多线程获取所有待执行task，获取完成之后执行之。具体见代购proxy MessageReadTask

### CyclicBarrier：一组线程互相等待

1. 一组线程`互相`等待至某个状态,然后执行对应的barrierCommand
2. 可重用（回环栅栏),barrierCommand执行完成之后，会进入nextGeneration，count会被重置成parities

eg. CountDownLatch的例子同样适用。

### Exchanger一对线程交换数据

1. exchange方法帮一对线程交换数据
2. exchange方法会阻塞调用方线程直至另一方线程参与交换

### Semaphore 信号量

1. 通过计数器控制对共享资源的访问
