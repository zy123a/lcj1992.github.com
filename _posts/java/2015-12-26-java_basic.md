---
layout: post
title: java基础
categories:  java
tags: java autoboxing transient thread jdbc
---

*   [java初始化顺序](#init)
*   [自动装箱拆箱](#autobox)
*   [transient](#transient)
*   [thread](#thread)

### 初始化顺序 {#init}

1.  静态变量（类变量）、静态初始化块 > 实例变量、 初始化块 > 构造器

2.  父类 > 子类  子类的静态初始化在父类的实例变量初始化之前

3.  静态代码块是在类加载时主动执行的，静态方法是在被调用的时候被动执行的

### 自动装箱拆箱 {#autobox}

装箱
基本类型-》引用类型

The Java compiler applies autoboxing when a primitive value is:

1.  Passed as a parameter to a method that expects an object of the corresponding wrapper class.

2.  Assigned to a variable of the corresponding wrapper class.

拆箱
引用类型-》基本类型

The Java compiler applies unboxing when an object of a wrapper class is:

1.  Passed as a parameter to a method that expects a value of the corresponding primitive type.

2.  Assigned to a variable of the corresponding primitive type.

If the value p being boxed is true, false, a byte, or a char in the range \u0000 to \u007f, or an int or short number between -128 and 127 (inclusive), then let r1 and r2 be the results of any two boxing conversions of p. It is always the case that r1 == r2.

### transient

1.  一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

2.  transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

3.  被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。

### thread

结合[jvm内存模型](http://lcj1992.github.io/2015/09/03/java_internal)

在Java当中，线程通常都有五种状态，`创建`、`就绪`、`运行`、`阻塞`和`死亡`。

|状态 |说明|
|--|--|
|创建|在生成线程对象，并没有调用该对象的start方法，这是线程处于创建状态。|
|就绪|当调用了线程对象的start方法之后，该线程就进入了就绪状态，但是此时线程调度程序还没有把该线程设置为当前线程，此时处于就绪状态。在线程运行之后，从等待或者睡眠中回来之后，也会处于就绪状态。|
|运行|线程调度程序将处于就绪状态的线程设置为当前线程，此时线程就进入了运行状态，开始运行run函数当中的代码。|
|阻塞|线程正在运行的时候，被暂停，通常是为了等待某个时间的发生(比如说某项资源就绪)之后再继续运行。sleep,suspend，wait等方法都可以导致线程阻塞。|
|死亡|如果一个线程的run方法执行结束或者调用stop方法后，该线程就会死亡。对于已经死亡的线程，无法再使用start方法令其进入就绪。|

thread.run()和thread.start()

start方法会新起一个线程，而run方法只是普通的方法调用，还是在原线程。

### shallow copy & deep copy

1.  浅拷贝，对于基本类型和String，没有问题，独立的一份，对于对象，只是拷贝了引用，共用同一片内存区域
2.  深拷贝，两对象独立

参考

[1]<http://javaconceptoftheday.com/difference-between-shallow-copy-vs-deep-copy-in-java/>

[2]<https://www.cs.utexas.edu/~scottm/cs307/handouts/deepCopying.htm>

[3]<https://en.wikipedia.org/wiki/Object_copying>


### Object#wait() &&  Thread.sleep()

[1]<http://javaconceptoftheday.com/difference-between-wait-and-sleep-methods-in-java/>


### Unsafe

[1]<http://blog.csdn.net/fenglibing/article/details/17138079>

String.format格式化输出%?

 %%

所有的Serializable的类必须含有serialVersionUID属性，这是出于性能考虑，如果没有serialVersionUID属性，jre会自己计算一个值，这个值的计算很消耗资源。
能double check最好double check，真的，不要让别人的不靠谱性影响自己的不靠谱性
加了事务的方法，不要catch异常，抛出来才能rollback

httpclient使用完后要release
文件打开之后要close
封装的线程上下文要remove

   

    
