---
layout: post
title: 字节码角度看jvm内存模型
categories: java
tags: jvm byte_code
---

![程序与其字节码](/images/java/jvm_example.jpg)

*   栈的深度为3，局部变量为3，方法参数为0(非静态方法，默认有个参数this)。

        ldc #5
        astore_0
*  将常量池中#index为5的string（含有#index指向utf8类型的常量，那才是真的"ab"的bytes）压入操作数栈，然后pop出栈保存在局部变量0中

*  同样，将常量池中#index为6的string保存在局部变量1中，这时候栈的深度为0，局部变量表中含有两个局部变量。

        new #7
*  在堆中实例化一个StringBuilder对象，并将其引用压入操作数栈中。

        dup
*  复制栈顶元素，并压入操作数栈,此时栈深度为2

        invokesepcial
*  将栈顶元素（一个StringBuilder的引用）pop出栈，调用其构造方法，创建新的栈帧,进行初始化操作，栈深为1

*  将常量a压入操作数栈,栈深为2

        invokevirtual  #10
*   栈顶两个元素pop出栈，栈顶的字符串a的引用（方法参数）和StringBuilder对象的引用（相关对象引用）。然后将返回值压入操作数栈，还是那个StringBuilder的引用，只不过现在不是空的了，为“a”;栈深为1.

注：invokespecial和invokevirtual等方法调用，方法参数和相关对象的引用都需要pop出栈，不然jvm如何创建新的栈帧啊。invokestatic就不需要了。

*   将局部变量1压入操作数栈

*   同7，调用append方法,栈深为1,栈顶元素为StringBuilder引用，”ad“

        invokevirtual  #11
*   pop栈顶的StringBuilder引用，调用其toString方法，返回String引用

        getstatic  #3
*   将类型为Ljava/io/PrintStream的静态域System.out压入栈中

*   将局部变量0压入栈中

*   将局部变量2压入栈中，此时栈深为3!

...
