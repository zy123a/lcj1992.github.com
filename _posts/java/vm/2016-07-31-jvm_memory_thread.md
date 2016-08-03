---
layout: post
title: java内存模型与线程
categories: java
tags: jvm memory thread
---


### 工作内存与主内存的交互 {#thread_memory}

1.  一个变量如何从主内存拷贝到工作内存
2.  如何从工作内存同步回主内存

java内存模型定义了8种操作来完成，虚拟机实现必须保证这八种操作是原子不可再分的，其交互如下图所示：（对于double和long型变量来说，load，store，read和write在某些平台允许有例外）

![jvm_memory_thread](/images/java/jvm_memory_thread.png)


上述8个操作还需要满足如下规则:

1.  不允许read和load、store和write操作之一单独出现，即不允许一个变量从主内存读取了但工作内存不接受，或者从工作内存发起了回写但是主内存不接受的情况出现
2.  不允许丢弃它的最近的assign操作，即变量在工作内存中改变了之后必须把该变化同步回主内存。
3.  不允许线程无原因地（没有发生过任何assign操作）把数据从线程的工作内存同步到主内存中
4.  use、store操作之前必须先执行assign、load操作，一个新的变量只能在主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量
5.  一个变量在同一时刻只允许一个线程对其进行lock操作，同一个线程可以多次执行lock，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。
6.  如果对一个变量执行lock操作，线程会清空工作内存中此变量的值，执行引擎使用时，必须重新执行load或assign操作来初始化变量的值。
7.  如果变量事先没有被lock操作，就不能对其进行unlock操作，也不允许unlock一个被其他线程锁定住的变量
8.  对变量进行unlock操作之前，必须先把此变量同步回主内存（store、write）。

### volatile型变量的特殊规则

1.  线程T对变量V的use操作必须和线程T对变量V的read、load动作相关联
2.  线程T对比变量V的assign动作必须和线程T对变量V的store、write动作相关联
3.  假定动作A是线程T对变量V实施的use或assign动作，假定动作F是和动作A相关联的load或store动作，假定动作P是和动作F相对应的对变量V的read或write动作；类似的，假定动作B是线程T对变量W实施的use或assign动作，假定动作G是和动作B相关联的load或store动作，假定动作Q是和动作G相对应的对变量W的read或write动作。如果A优先于B，那么P优先于Q。（这条规则要求volatile修饰的变量不会被指令重排序优化，保证代码的执行顺序与程序的顺序相同）。

volatile变量只能保证可见性，在不符合以下两条规则的运算场景下，我们仍然需要通过加锁（使用synchronized或者java.util.concurrent中的原子类）来保证原子性：

1.  运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值
2.  变量不需要与其他状态变量共同参与不变性约束。

### 重排序 {#reordering}

Doug Lea的图。 此人写了Collection和Concurrent包的。

图中标no的就是不能进行重排序的。

![重排序规则](/images/java/reordering.png)

其中：
1. Normal Loads are getfield, getstatic, array load of non-volatile fields
2. Normal Stores are putfield, putstatic, array store of non-volatile fields
3. Volatile Loads are getfield, getstatic of volatile fields that are accessible by multiple threads
4. Volatile Stores are putfield, putstatic of volatile fields that are accessible by multiple threads
5. MonitorEnters (including entry to synchronized methods) are for lock objects accessible by multiple threads.
6. MonitorExits (including exit from synchronized methods) are for lock objects accessible by multiple threads.

### happen-before

根据以上的规定，我们可以总结出一下8条happen-before规则，happen-before的前后两个操作会被重排序且后者对前者的内存可见。


1. 程序顺序规则：一个线程中的每个操作，happens- before 于该线程中的任意后续操作（控制流操作而不是程序代码顺序）。

2. 监视器锁规则：对一个监视器的解锁，happens- before 于随后对这个监视器的加锁。

3. volatile变量规则：对一个 volatile域的写，happens- before于任意后续对这个volatile域的读。

4. 线程启动规则：Thread对象的start()方法happens - before 于此线程的每一个动作。

5. 线程终止规则：线程中所有操作都happens - before 于对此线程的终止检测。

6. 线程中断规则：对线程interrupt()方法的调用happens - before 于被中断线程的代码检测到中断事件的发生。

7. 对象终结规则：一个对象的初始化完成（构造函数执行结束）happens - before 于它的finalize()方法的开始。

8. 传递性：如果 A happens- before B，且 B happens- before C，那么 A happens- before C。

### 参考 {#ref}

[Threads and Locks]<https://docs.oracle.com/javase/specs/jvms/se6/html/Threads.doc.html>

[深入理解java虚拟机-12章java内存模型与线程]<https://book.douban.com/subject/24722612/>

[jvm内存模型]<https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html>

[Memory Barriers and JVM Concurrency]<https://www.infoq.com/articles/memory_barriers_jvm_concurrency>

[The JSR-133 Cookbook for Compiler Writers]<http://g.oswego.edu/dl/jmm/cookbook.html>
