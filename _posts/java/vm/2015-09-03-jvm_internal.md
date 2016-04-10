---
layout: post
title: jvm internal(译)
categories: java
tags: jvm byte_code thread stack
---

*   [线程](#thread)
    *   [系统线程](#sysThread)
    *   [线程私有](#perThread)
        *   [程序计数器](#pc)
        *   [栈](#stack)
            *   [局部变量表](#localVarTable)
            *   [操作数栈](#operandStack)
            *   [动态链接](#dynamicLink)
*   [堆](#heap)
*   [非堆](#nonHeap)
    *   [方法区](#methodArea)
    *   [符号表](#symbol_table)
    *   [internedString](#interned_string)

这篇文章解释了java虚拟机jvm内部的架构。下边这幅图展示了符合jvm版本7规范的典型的jvm的内部关键的组成部分。
上图所示的组件会在下边的两节中逐一解释。第一部分讲的是每个线程创建的，第二部分展示的独立于线程的。   

## 线程 {#thread}
![jvm memory](http://blog.jamesdbloom.com/images_2013_11_17_17_56/JVM_Internal_Architecture.png)

jvm允许一个应用程序拥有多线程，并发地运行。在hotspot虚拟机中在java线程和操作系统的线程有一个直接的映射关系。在准备好java线程所有的状态（state）诸如thread-local的存储，分配的buffers，同步的对象，栈和程序计数器之后，会创建本地线程。一旦java线程终止了，本地线程就会被回收。操作系统调度所有的线程并分发给任意可用的cpu。一旦本地的线程被初始化后，它调用java线程的run()方法。当run方法返回时，未被捕获的异常将被处理，线程终止，然后本地线程确认jvm是否需要被终止（比如如果该线程是最终的非守护线程的话，jvm就应该被teminated）。当线程被终止时，本地线程和jvm线程所需要的所有资源都会被释放。      

### 系统线程 {#sysThread}
 
如果你用jconsole或者任意的debugger，可以看到许多线程都在后台运行。除了主线程，这些在调用public static void main(String[] args)作为一部分被创建的线程也会运行。所有的线程都是有主线程创建的。jvm中主要的一些后台线程如下：      

|系统线程　|作用|
|-|-|
|vm thread|	这个线程等待一些操作出现，这需要jvm到达安全点。这些操作的必须发生在一个独立的线程中，因为他们都需要在jvm在安全点，这时候堆不能被改变，这种类型的此操作诸如"stop the world”垃圾回收，线程栈dump，线程阻塞（thread suspension），biased locking revocation ？？？
|Periodic task thread	|这个线程主要是为一些需要调度一些周期性操作的定时任务（如中断）
|gc threads	|这些线程为jvm中不同类型的垃圾回收活动提供支持
|compiler threads	|这些线程在运行时将字节码编译成本地代码
|Signal dispatcher thread	|这个线程接收发给jvm的信号，同时通过调用合适的jvm方法在jvm中对其进行处理
|Signal dispatcher thread	|这个线程接收发给jvm的信号，同时通过调用合适的jvm方法在jvm中对其进行处理

### 线程私有 {#perThread}
每个线程的执行都有下列的组成部分,`程序计数器`，`栈`；每个栈有许多`栈帧`组成；每个栈帧又包含`局部变量表`,`操作数栈`,`当前方法所在类的运行时常量池引用`等。

#### 程序计数器（program counter PC）{#pc}

当前的指令instruction或者叫操作码opcode的地址，native除外。如果当前的方法是native，程序计数器是未定义的，所有的cpu都有一个程序计数器，一般情况下，在一条指令执行完后程序计数器加一，然后指向下一条将要被执行的指令的地址。jvm使用pc来追踪程序现在在哪执行指令，事实上，pc指向的是方法区中的一个内存地址.   

#### 栈 {#stack}

每个线程都有它自己的栈（线程私有），持有这个线程要执行的每个方法的栈帧。栈是后进先出的数据结构，所以要被执行的方法是位于栈顶的。当一个方法被调用时，一个新的栈帧将被创建并add（push）到栈顶。当这个方法return或者在执行过程中未捕获的异常被throw，这个栈帧将会被remove（pop）出栈。栈不是直接被操作的 ，除了push和pop帧对象，因此，帧对象可以在堆中分配，并且内存也不需要连续。（The stack is not directly manipulated, except to push and pop frame objects, and therefore the frame objects may be allocated in the Heap and the memory does not need to be contiguous.）???

native栈,不是所有的java虚拟机都支持本地方法，支持的都会创建一个本地方法栈。如果一个jvm用C的连接模型来实现本地方法调用（java native Invocation，本地方法栈就是一个C的栈。这种情况下，本地方法栈中的参数的顺序和返回值将和典型的c程序一样。通常情况下，一个本地方法可以call back into jvm，并且调用java方法。这样的本地到java的调用将发生在栈上（通常是java栈）。线程将离开native栈，并创建一个新的栈帧在栈上（通常是java栈）

一个栈可以是动态的，也可以固定大小的。如果一个线程需要超过允许的大小，就会报栈溢出错误StackOverFlowError，如果一个栈需要一个新的栈帧，但是没有足够的内存来分配它，就会报内存溢出OutOfMemoryError。

每个方法调用时，一个新的栈帧就会被创建并add（push）到栈顶。当方法return或者在执行过程中未捕获的异常被抛出时，栈帧会被removed（pop）。

每个栈帧包含：

1.  局部变量表
2.  返回值
3.  操作数栈
4.  当前方法所在类的运行时常量池的引用

##### 局部变量表 {#localVarTable}
局部变量表包含执行该方法时，所有用到的局部变量，包括this引用，所有的方法参数和一些定义的局部变量，对于类方法（如static方法）来说，方法参数从0开始，对于实例方法，0 slot为this引用。  
  
一个局部变量可以是：   

1.  boolean
2.  byte
3.  char
4.  long
5.  short
6.  int
7.  float
8.  double
9.  reference
10. returnAddress
     
在局部变量表除了long和double外，所有的类型占用一个slot，long和double占用连续的两个slot因为这些类型是64bit而不是32bit。

##### 操作数栈{#operandStack}

操作数栈在字节码执行的过程中使用，作用和cpu的寄存器差不多。大多数jvm的字节码在入栈，出栈，复制栈顶元素，交换，执行产生值消费值的操作。因此，在字节码中，交换局部变量表和操作数栈的指令是非常频繁的。例如，一个简单的变量初始化编译成字节码后是两条与操作数栈交互的字节码。   

int i;
编译成字节码为：

    0:iconst_0    // 将常量池0入操作数栈
    1:istore_1    // 将栈顶元素出栈，并存入局部变量表中的变量1。

##### 动态链接 {#dynamicLink}

每个栈帧都包含一个运行时常量池的引用。这个引用指向当前栈帧所对应的方法所在的类。这个引用为动态链接提供支持。

C/C++代码首先是被编译成一个object文件(xxx.o),然后这些.o文件被链接成一个可执行文件如（executable或者dll）。在链接的过程中，每个xxx.o文件中的符号链接被真实的内存地址（当然这个地址还是相对的，相对于最终的可执行文件。可执行文件加载到内存中还是会有相对地址，线性地址，绝对地址的转换）在java中，这个过程是在运行时动态进行的。  

当一个java的class文件被编译，所有指向变量的引用，方法名都被作为符号链接存储在class的常量池中。一个符号链接是逻辑引用而不是指向真实的内存地址。jvm实现可以选择在什么时候来处理这些符号链接，可以在class文件加载之后，被验证的时候，这叫做急切的或者静态的解析（eager or static resolution）；或者这可以发生在符号链接第一次被使用的时候，这叫做延迟解析（lazy or late resolution）。然而在每个引用第一次被使用的时候，jvm必须表现的好像这个解析发生过，不能抛出任何处理异常？？？（However the JVM has to behave as if the resolution occurred when each reference is first used and throw any resolution errors at this point）。绑定是这些有符号链接标识的域，方法或者类被一个直接引用代替的过程，这仅仅发生一次，因为符号链接已经完全有直接引用代替。如果指向类的符号链接没有被处理，这个类会被加载。每个直接引用作为一个相对于存储结构的偏移地址，这个存储结构和运行时的变量和方法相关。

## 堆 {#heap}

堆被用作在运行时分配类的实例和数组。数组和对象不能存储在栈中，因为一个栈帧在创建之后就不能改变大小。栈帧仅仅存储堆中对象或者数组的指针。不像基本类型和局部变量表中的引用（在每个栈帧中）对象总是存储在堆中，因此当一个方法结束时,它们不被移除。对象仅会被垃圾收集器收集。  

为了支持垃圾回收，堆被划分为这三个部分：      
 
1.  年轻代(经常被划分为eden和survivor)
2.  老年代
3.  永久代 ?
  
对象和数组只会被垃圾回收器回收。  
典型的过程如下：       

1.  新的对象和数组在年轻代创建
2.  minor gc将在年轻代发生。仍然存活的对象会从eden区移入survivor区
3.  major gc（full gc）这通常会造成应用线程暂停，会将对象在代之间移动。仍然存活的对象会被从年轻代移入老年代（old 或者tenured）
4.  当老年代被回收的时候，永久代也会被回收，他们任一满的时候都会触发full gc来对他们进行回收。

## 非堆内存 {#nonHeap}

非堆内存包括：  

*   永久代
    *   方法区
    *   interned strings
*   code cache 被用来编译，并存储被jit编译器编译成本地代码的方法

即时编译just in time（jit）compilation

java字节码被解释执行，然而它没有本地代码快，为了提高性能，hotspot jvm寻找字节码中的“hot”区域，并把他们编译成本地代码，这个本地代码被保存在code cache在非堆内存中。

### 方法区 {#methodArea}
 
方法区存储一些类的信息，诸如：    

*  classloader引用
*  运行时常量池
    *   数字型常量
    *   域引用
    *   方法引用
    *   属性
*   域数据
    *   每个域
        *   名字
        *   类型
        *   标识符
        *   属性
*   方法数据
    *   每个方法
        *   名字
        *   返回值
        *   参数（有序）
        *   标识符
        *   属性
*   方法代码
    *   每个方法
        *   字节码
        *   操作数栈的大小
        *   局部变量表的大小
        *   局部变量表
        *   异常表
            *   每个异常handler
            *   开始点
            *   结束点
            *   程序计数器偏移for handler code
            *   常量池中被捕获的异常类的索引

所有的线程共享方法区，所以访问方法区数据和动态链接的过程必须是线程安全的。如果两个线程同时尝试去访问一个还没有被加载的类的域或者方法，它必须仅被加载一次，两个线程必须等到类被加载之后才能执行。

虽然方法区逻辑上来说是堆的一部分，但是通常的实现，可能选择不垃圾回收它。 然而oracle jvm的jconsole显示了方法区和code cache是在non-heap。openjdk的源码也表明code cache也是独立于堆的。

### 符号表 {#symbol_table}
jvm有一个存在于永久代的符号表。这个符号表是一个符号指针到符号映射的哈希表，包含一个指向每个类中所有符号的指针。

当一个符号被从符号表中移除时，引用计数用来控制。例如，当一个类被卸载之后，常量池中的所有引用基数就会减少。当一个符号的引用计数减为0时，符号表知道这个符号不再被引用，这个符号会被从符号表中卸载。符号表和下边的字符串表，所有的条目被以一种规范化的格式，用来提高效率并确保每个条目只出现一次。

### interned Strings（String table） {#interned_string}
java语言规范规定相同的字符串(string literals)，包含有相同的unicode 序列,必须引用相同的String实例。如果String.intern()方法被调用在一个String引用上，必须返回和字符串（string literals）相同的引用。所以下边返回true

    ("j" + "v" + "m").intern() == "jvm"

In the Hotspot JVM interned string are held in the string table, which is a Hashtable mapping object pointers to symbols (i.e. Hashtable<oop, Symbol>), and is held in the permanent generation. For both the symbol table (see above) and the string table all entries are held in a canonicalized form to improve efficiency and ensure each entry only appears once.

String literals are automatically interned by the compiler and added into the symbol table when the class is loaded. In addition instances of the String class can be explicitly interned by calling String.intern(). When String.intern() is called, if the symbol table already contains the string then a reference to this is returned, if not the string is added to the string table and its reference is returned.


参考

[1]<http://blog.jamesdbloom.com/JVMInternals.html>

[2]<http://www.artima.com/insidejvm/ed2/jvm4.html>