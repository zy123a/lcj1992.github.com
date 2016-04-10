---
layout: post
title: java字节码
categories: java
tags: jvm 字节码 操作数栈 局部变量表 
---



#### 基础 {#basic}

1.指令类型

存储指令 （例如：aload_0, istore）   
算术与逻辑指令 （例如: ladd, fcmpl）   
类型转换指令 （例如：i2b, d2i）  
对象创建与操作指令 （例如：new, putfield）  
堆栈操作指令 （例如：swap, dup2）  
控制转移指令 （例如：ifeq, goto）  
方法调用与返回指令 （例如：invokespecial, areturn)  

2.前/后缀 操作数类型  

i 整数 l 长整数 s 短整数 b 字节 c 字符 f 单精度浮点数 d 双精度浮点数 z 布尔值 a 引用

#### 一个例子 {#example}

![程序与其字节码](/images/java/jvm_example.jpg)

*   栈的深度为3，局部变量为3，方法参数为0(非静态方法，默认有个参数this)。

        stack=3, locals=3, arg_size=0
*  将常量池中#index为5的string（含有#index指向utf8类型的常量，那才是真的"ab"的bytes）压入操作数栈，然后pop出栈保存在局部变量0中,`栈深为0,局部变量表size为1`.
        
         0: ldc             #5    // String ab
         2: astore_0    
*  同样，将常量池中#index为6的string保存在局部变量1中，`栈深为0,局部变量表size为2`。
    
         3: ldc             #6    // Sting b
         5: astore_1
*  在堆中实例化一个StringBuilder对象，并将其引用压入操作数栈中。`栈深为1`

         6: new             #7    // class java/lang/StringBuilder
*  复制栈顶元素，并压入操作数栈,此时`栈深度为2`

         9: dup
*  将栈顶元素（一个StringBuilder的引用）pop出栈，调用其构造方法，创建新的栈帧,进行初始化操作，`栈深为1`

        10: invokesepcial   #8    // Method java/lang/StringBuilder."<init>";()V
*  将常量a压入操作数栈,栈深为2

        13: ldc             #9    // String a 
*   栈顶两个元素pop出栈，栈顶的字符串a的引用（方法参数）和StringBuilder对象的引用（相关对象引用）。然后将返回值压入操作数栈，还是那个StringBuilder的引用，只不过现在不是空的了，为“a”;`栈深为1`.

        15: invokevirtual  #10    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
注：invokespecial和invokevirtual等方法调用，`方法参数`和`相关对象的引用`都需要pop出栈，不然jvm如何创建新的栈帧啊。invokestatic就不需要了。
   
*   将局部变量1压入操作数栈,`栈深为2`
    
        18: aload_1 
*   调用append方法,栈顶元素为StringBuilder引用，”ab“,调用append方法

        19: invokevirtual  #10    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
*   pop栈顶的StringBuilder引用，调用其toString方法，返回String引用

        22: invokevirtual  #11    // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
*   将栈顶的String引用保存在局部变量2中,`局部变量表size为2,栈深为0`

        25: astore_2

*   将类型为Ljava/io/PrintStream的静态域System.out压入栈中
        
        26: getstatic      #3     // Field java/lang/System.out:Ljava/io/printStream;

*   将局部变量0压入栈中

        29: aload_0

*   将局部变量2压入栈中，此时`栈深为3`!

        30: aload_2
*   pop栈顶两个元素,并比较他们,如果两个对象引用不等,跳到38

        31: if_acmpne      38

*   将常量1入栈

        34: iconst_1 
        
*   然后跳到39 

        35: goto           39
*   将常量0入栈

        38: iconst_0
*   pop栈顶两元素PrintStream引用和比较结果

        39: invokevirtual  #4     // Method java/io/PrintStream.println:(Z)V (false)
*   pop栈顶元素(其实栈顶为空,其他类型另说)

        42: return
        
*   行号表源代码14行,对应于字节码前的序号.
    
        >a.字节码前的序号为相对于该栈帧的偏移量.
        >b.字节码都是单字节的,为什么字节码前的序号不是连续的?
        >b1.有的指令不需要参数,有的需要多个参数
        >b2.有的参数为单字节的,有的参数是多字节的
    
        LineNumberTable
           line 14: 0
           line 15: 3
           line 16: 6
           
#### 参考 {#ref}

[java字节码 wikipedia]<https://en.wikipedia.org/wiki/Java_bytecode>  

[java字节码指令 wikipedia]<https://en.wikipedia.org/wiki/Java_bytecode_instruction_listings>

[java字节码指令 这个感觉更赞]<http://cs.au.dk/~mis/dOvs/jvmspec/ref-Java.html>
    