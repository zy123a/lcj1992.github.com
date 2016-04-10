---
layout: post
title: java内存结构与字节码
categories: java
tags: memory byteCode
---

### class文件格式

一个编译后的class文件格式如下：

    ClassFile {
        u4			magic;
        u2			minor_version;
        u2			major_version;
        u2			constant_pool_count;
        cp_info		contant_pool[constant_pool_count – 1];
        u2			access_flags;
        u2			this_class;
        u2			super_class;
        u2			interfaces_count;
        u2			interfaces[interfaces_count];
        u2			fields_count;
        field_info		fields[fields_count];
        u2			methods_count;
        method_info		methods[methods_count];
        u2			attributes_count;
        attribute_info	attributes[attributes_count];
    }

|字段|说明|
|-|-|
|magic	|魔数CAFEBABE
|minor_version major_version| 这个class编译的版本|
|constant_pool	|相当于一个符号表
|access_flags	|提供这个类的一系列的标识
|this_class	|类名索引
|super_class|	父类名的索引
|interfaces	|所有实现了的接口的符号链接的索引
|fields	|...|
|methods|	...|
|attributes|	...|

使用javap命令可以查看class文件的字节码
如果你编译如下simple class

    package org.jvminternals;
    public class SimpleClass {
        public void sayHello() {
            System.out.println("Hello");
        }
    }
然后你会得到如下输出

    javap -v -p -s -sysinfo -constants classes/org/jvminternals/SimpleClass.class

    public class org.jvminternals.SimpleClass
       SourceFile: "SimpleClass.java"
       minor version: 0
       major version: 51
       flags: ACC_PUBLIC, ACC_SUPER
    Constant pool:
       #1 = Methodref          #6.#17         //  java/lang/Object."<init>":()V
       #2 = Fieldref           #18.#19        //  java/lang/System.out:Ljava/io/PrintStream;
       #3 = String             #20            //  "Hello"
       #4 = Methodref          #21.#22        //  java/io/PrintStream.println:(Ljava/lang/String;)V
       #5 = Class              #23            //  org/jvminternals/SimpleClass
       #6 = Class              #24            //  java/lang/Object
       #7 = Utf8               <init>
       #8 = Utf8               ()V
       #9 = Utf8               Code
       #10 = Utf8               LineNumberTable
       #11 = Utf8               LocalVariableTable
       #12 = Utf8               this
       #13 = Utf8               Lorg/jvminternals/SimpleClass;
       #14 = Utf8               sayHello
       #15 = Utf8               SourceFile
       #16 = Utf8               SimpleClass.java
       #17 = NameAndType        #7:#8          //  "<init>":()V
       #18 = Class              #25            //  java/lang/System
       #19 = NameAndType        #26:#27        //  out:Ljava/io/PrintStream;
       #20 = Utf8               Hello
       #21 = Class              #28            //  java/io/PrintStream
       #22 = NameAndType        #29:#30        //  println:(Ljava/lang/String;)V
       #23 = Utf8               org/jvminternals/SimpleClass
       #24 = Utf8               java/lang/Object
       #25 = Utf8               java/lang/System
       #26 = Utf8               out
       #27 = Utf8               Ljava/io/PrintStream;
       #28 = Utf8               java/io/PrintStream
       #29 = Utf8               println
       #30 = Utf8               (Ljava/lang/String;)V
    public org.jvminternals.SimpleClass();
            Signature: ()V
            flags: ACC_PUBLIC
            Code:
            stack=1, locals=1, args_size=1
            0: aload_0
            1: invokespecial #1    // Method java/lang/Object."<init>":()V
            4: return
            LineNumberTable:
            line 3: 0
            LocalVariableTable:
            Start  Length  Slot  Name   Signature
            0      5      0    this   Lorg/jvminternals/SimpleClass;

    public void sayHello();
            Signature: ()V
            flags: ACC_PUBLIC
            Code:
            stack=2, locals=1, args_size=1
            0: getstatic      #2    // Field java/lang/System.out:Ljava/io/PrintStream;
            3: ldc            #3    // String "Hello"
            5: invokevirtual  #4    // Method java/io/PrintStream.println:(Ljava/lang/String;)V
            8: return
            LineNumberTable:
            line 6: 0
            line 7: 8
            LocalVariableTable:
            Start  Length  Slot  Name   Signature
            0      9      0    this   Lorg/jvminternals/SimpleClass;
            }`

这个class文件展示了三个主要的区域：常量池，构造器和sayHello方法

常量池--...

方法----每个包含四个区域

>签名和访问标志
>字节码
>行数表：这个为debugger提供信息将行号和字节码顺序照应，如上边java文件的第六行对应于与字节码0 ,第七行对应于字节码8
>局部变量表：列举了这个栈帧里所有的局部变量，例子中的局部变量只有this引用，start是局部变量声明的位置(第几条字节码)；length是从起始位置作用的长度；从0开始，长度为9,也就是说this作用的范围是字节码0到字节码8,slot是插槽位置，除了long和double占两个插槽，其他都占一个；Name是变量名；Signature是变量类型。

这个class文件中的字节码：

>aload_0 将局部变量表中变量0 压入操作数栈
>ldc 将运行时常量池中#index常量压入操作数栈
>getstatic 从运行时常量池中#index的静态域的值压入栈中
>invokespecial,invokevirtual 这是调用方法的命令，类似的命令还有invokeinterface，invokedynamic，invokestatic。invokevirtual是调用这个对象的类方法，invokespecial调用的是类的初始化方法，private方法和父类的方法。
>return return 语句，类似的还有ireturn，lreturn，freturn，dreturn，areturn。

在任何字节码中，大部分的操作都是在局部变量表，操作数栈和常量池中进行交互的，如下。
构造器有两条指令，第一条是将this引用push到操作数栈。第二条指令是父类的构造器方法被调用，消费this的值，因此将他pop出操作数栈。

![simpleClass](http://blog.jamesdbloom.com/images_2013_11_17_17_56/bytecode_explanation_SimpleClass.png)

sayHello方法更加麻烦，因为它必须通过运行时常量池处理符号引用和真实引用。第一条指令getstatic将一个静态域的引用压入操作数栈，第二条指令ldc将字符串“Hello”压入栈中。最后的操作码invkespecial调用System.out的println方法，将"Hello"pop出栈作为方法的参数，并且在当前线程的栈中创建一个新的栈帧。

![sayHello](http://blog.jamesdbloom.com/images_2013_11_17_17_56/bytecode_explanation_sayHello.png)

### 类加载器classloader

jvm通过bootstrap classloader加载一个初始化类。这个类在调用public static void main方法之前被链接并初始化。这个方法的执行会反过来驱动其他类和接口的加载，链接，初始化。

加载是通过唯一的名字找到代表这个类或者接口的class文件，并以字节数组的方式读入内存的过程。接下来这些字节流会被解析来确认它代表一个class对象，有正确的major和minor版本号。它的直接父类也会被加载。一旦这个过程完成了，一个类或者接口对象就从二进制的格式被创建在jvm中。

链接验证类或者接口并准备它的直接父类或者父接口的过程。链接包含验证verifying，准备preparing和选择性的解析resolving。

验证是确认class或者接口有正确的结构，符合java语言规范，符合jvm规范的过程。例如下边的会被check。

>final的方法或者类没被重写
>方法没有不合理的操作栈
>变量在使用之前被初始化
>变量是该类型正确的值

在验证阶段执行这些意味着不需要在运行时进行这些验证。链接过程中的验证会减慢类的加载过程，但是它避免了执行字节码时的重复验证。准备会为静态存储和jvm用到的任意的数据结构（如方法表）分配内存。静态域会被创建并初始化为默认值，然而这时候没有进行任何其他的初始化，也没有任何的代码被执行。
解析会加载相关的类或者接口来验证符号链接是否正确。如果不在这时候，符号链接的解析会被延迟，直到它被用到。初始化类和接口的初始化就是调用类和接口的初始化方法。
在jvm中有多个不同角色的classloaders。每个classloader代理它的父classloader（父classloader加载子classloader），除了bootstrap classloader它是最顶层的classloader。
![class_load_link_init](http://blog.jamesdbloom.com/images_2013_11_17_17_56/Class_Loading_Linking_Initializing.png)
Bootstrap Classloader是由本地代码实现的，因为它很早的，随着jvm的启动就被初始化了。bootstrap classloader负责加载基本的java apis，如rt.jar。它仅仅加载boot classpath路径下的classes，由于这个路径下的class具有很高的可靠性，所以这里省去了常规的class需要进行的一些验证的步骤。Extension Classloader加载标准java扩展apis，如security extension functions System Classloader是默认的应用classloader，它load应用classes从classpath中User Defined Classloaders可以被用来选择性的加载应用classes。为了一些特殊的目的，我们需要自定义classloader，例如在运行时reload classes，web servers如tomcat实现不同groups下加载的classes之间的隔离
![class_loader_hierarchy](http://blog.jamesdbloom.com/images_2013_11_17_17_56/class_loader_hierarchy.png)
### 方法区在哪？
jvm规范7 明确的注明：虽然方法区逻辑上来说是堆的一部分，但是通常的实现，可能选择不垃圾回收它。 然而oracle jvm的jconsole显示了方法区和code cache是在non-heap。openjdk的源码也表明code cache也是独立于堆的。

### classloader reference
所有已被加载的类都包含有它对classloader的引用。classloader也会包含所有的它加载的类。

### 运行时常量池
jvm维护一个常量池，一个运行时的数据结构，它和符号表很类似，但是包含更过的数据。字节码需要数据，但是数据直接以字节码存储又太大，于是它会被存放在常量池中，字节码仅仅是持有对常量池的引用。

常量池中包含的几种类型：

>数字类型
>字符串类型
>类引用
>域引用
>方法引用

例如下边的代码

`Object foo = new Object();`
将会被编译成如下的字节码

    0: 	new #2 		    // Class java/lang/Object
    1:	dup
    2:	invokespecial #3    // Method java/ lang/Object "<init>"( ) V

new 操作符后跟#2,#2代表的是常量池的索引，第二条记录，第二条记录是一个类的引用，这个引用又指向常量池中另一条记录，是一个utf8的字符串，值为// Class java/lang/Object。这个符号链接会被用来找类java.lang.Object。new操作符会创建一个类的实例并初始化它的值。一个新的类实例的引用会被添加到操作栈上。dup指令创建一个栈顶元素的copy，并将其push到栈顶。最后一个类的实例化方法会被调用invokespecial，这个操作数栈也持有常量池的引用。初始化方法会消费（pop）栈顶元素的值，并将其作为参数传给初始化方法（这里会新建一个栈帧）。最后，栈顶会有一个新建的并初始化好的对象。

编译simple class：

    package org.jvminternals;

    public class SimpleClass {

       public void sayHello() {
           System.out.println("Hello");
       }

    }

 常量池会如下所示：

    Constant pool:
            #1 = Methodref          #6.#17         //  java/lang/Object."<init>":()V
            #2 = Fieldref           #18.#19        //  java/lang/System.out:Ljava/io/PrintStream;
            #3 = String             #20            //  "Hello"
            #4 = Methodref          #21.#22        //  java/io/PrintStream.println:(Ljava/lang/String;)V
            #5 = Class              #23            //  org/jvminternals/SimpleClass
            #6 = Class              #24            //  java/lang/Object
            #7 = Utf8               <init>
            #8 = Utf8               ()V
            #9 = Utf8               Code
            #10 = Utf8               LineNumberTable
            #11 = Utf8               LocalVariableTable
            #12 = Utf8               this
            #13 = Utf8               Lorg/jvminternals/SimpleClass;
            #14 = Utf8               sayHello
            #15 = Utf8               SourceFile
            #16 = Utf8               SimpleClass.java
            #17 = NameAndType        #7:#8          //  "<init>":()V
            #18 = Class              #25            //  java/lang/System
            #19 = NameAndType        #26:#27        //  out:Ljava/io/PrintStream;
            #20 = Utf8               Hello
            #21 = Class              #28            //  java/io/PrintStream
            #22 = NameAndType        #29:#30        //  println:(Ljava/lang/String;)V
            #23 = Utf8               org/jvminternals/SimpleClass
            #24 = Utf8               java/lang/Object
            #25 = Utf8               java/lang/System
            #26 = Utf8               out
            #27 = Utf8               Ljava/io/PrintStream;
            #28 = Utf8               java/io/PrintStream
            #29 = Utf8               println
            #30 = Utf8               (Ljava/lang/String;)V

常量池类型包含如下

>Integer               4字节的常量
>Long                  8字节的常量
>Float                 4字节的常量
>Double                8字节的常量
>String                指向一个utf8类型的条目，那里边才是String真实的bytes
>utf8                  一段utf8格式的字节流
>Class                 指向一个utf8格式的条目，那里是class的名字（jvm格式，这会在动态链接的过程中用到）
>NameAndType           冒号分割的一对值，都指向常量池中utf8类型的一个条目，冒号前是方法名或者域名，对于域来说，冒号后是全类名，对于方法来说，冒号后是类名加参数
>Fieldref，Methodref   逗号分割的一对之，逗号前指向的常量池中的Class类型的一个条目，逗号后指向的常量池中NameAndType类型的一个条目。

### 异常表
异常表存储每个异常handler的信息，诸如：
>开始点
>结束点
>程序计数器偏移for handler code
>常量池中被捕获的异常类的索引

如果一个方法中定义了try-catch和try-finally异常handler，然后一个异常表就会被建立。这包含每个异常处理handler或者finally块的信息，包括作用的范围，将要被处理的异常的类型和异常处理的具体字节码的位置。

当一个异常被抛出时，jvm会在当前方法中寻找一个符合的hadnler，如果在当前方法中没有找到，方法会被终止，并将当前栈帧pop出栈，异常会被抛出到调用的方法中（一个新的栈帧）。如果所有的栈帧都被pop出栈，仍然没有找到异常handler，当前线程会被终止。如果在最后一个非守护线程中（例如如果这个线程是主线程），异常被抛出，那么会造成jvm自己终止terminate。

finally异常handlers匹配所有类型的异常，所有总是执行，无论何时异常被抛出。这个例子中，没有异常被抛出，finally块在一个方法的最后仍然会被执行，方法会在return之前，执行finally块的代码。

### 符号表??
jvm有一个存在于永久代的符号表。这个符号表是一个符号指针到符号映射的哈希表，包含一个指向每个类中所有符号的指针。

当一个符号被从符号表中移除时，引用计数用来控制。例如，当一个类被卸载之后，常量池中的所有引用基数就会减少。当一个符号的引用计数减为0时，符号表知道这个符号不再被引用，这个符号会被从符号表中卸载。符号表和下边的字符串表，所有的条目被以一种规范话的格式，用来提高效率并确保每个条目只出现一次。

### interned Strings（String table）？？
java语言规范规定相同的字符串(string literals)，包含有相同的unicode 序列,必须引用相同的String实例。如果String.intern()方法被调用在一个String引用上，必须返回和字符串（string literals）相同的引用。所以下边返回true

    ("j" + "v" + "m").intern() == "jvm"

In the Hotspot JVM interned string are held in the string table, which is a Hashtable mapping object pointers to symbols (i.e. Hashtable<oop, Symbol>), and is held in the permanent generation. For both the symbol table (see above) and the string table all entries are held in a canonicalized form to improve efficiency and ensure each entry only appears once.

String literals are automatically interned by the compiler and added into the symbol table when the class is loaded. In addition instances of the String class can be explicitly interned by calling String.intern(). When String.intern() is called, if the symbol table already contains the string then a reference to this is returned, if not the string is added to the string table and its reference is returned.


## 参考

[java内存分配](http://www.cnblogs.com/redcreen/archive/2011/05/04/2036387.html)  

[class文件](http://blog.csdn.net/dc_726/article/details/7944154)  

[class文件包含的信息](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7)   
