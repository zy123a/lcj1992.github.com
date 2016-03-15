---
layout: post
title: 性能
categories: os performance
tags: performance os
---

类型
CPU密集型

IO密集型

磁盘
网络
设置线程数
明确痛点在哪： cpu速度 > 线程切换上下文速度 > io速度

对于cpu密集型，避免频繁切换线程上下文， 理想的是 `线程数 = CPU核数+1`

对于io密集型，线程切换上下文的时间相比于io阻塞的时间可以忽略，设置较多线程,避免减少io 造成的线程阻塞   

***最佳线程数目 = （线程等待时间与线程CPU时间之比 + 1）\* CPU数目***

参考：

[1]<http://www.blogjava.net/bolo/archive/2015/01/20/422296.html>

[2]<http://unixboy.iteye.com/blog/174173>