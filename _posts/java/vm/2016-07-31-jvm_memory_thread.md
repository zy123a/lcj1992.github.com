---
layout: post
title: java内存模型与线程
categories: java
tags: jvm memory thread
---

工作内存与主内存之间的交互：

1.  一个变量如何从主内存拷贝到工作内存
2.  如何从工作内存同步回主内存

java内存模型定义了8种操作来完成，虚拟机实现必须保证这八种操作是原子不可再分的，其交互如下图所示：（对于double和long型变量来说，load，store，read和write在某些平台允许有例外）

![jvm_memory_thread](/images/java/jvm_memory_thread.png)

### 参考 {#ref}

[Threads and Locks]<https://docs.oracle.com/javase/specs/jvms/se6/html/Threads.doc.html>

[深入理解java虚拟机-12章java内存模型与线程]<https://book.douban.com/subject/24722612/>

[jvm内存模型]<https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html>
