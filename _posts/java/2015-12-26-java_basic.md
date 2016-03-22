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
*   [annotation](#annotation)


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

### annotation

#### java中内置的三种注解
`@Override`: 重写方法，或者实现接口
`@Deprecated`:不鼓励使用，很危险或者是有更好的替代。
`@SuppressWarnings`: 压制警告

#### 元注解
元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它annotation类型作说明。

1.@Target

表示该注解可以用于什么地方，可能的ElementType参数包括：
CONSTRUCTOR：构造器的声明
FIELD：域声明（包括enum实例）
LOCAL_VARIABLE:局部变量声明
METHOD：方法声明
PACKAGE：包声明
PARAMETER：参数声明
TYPE：类，接口（包括注解类型）或enum声明

2.@Retention

表示需要在什么级别保存该注解信息。可选的RetentionPolicy参数包括：
SOURCE：注解将被编译器丢弃
CLASS：注解在class文件中可用，但会被VM丢弃
RUNTIME：VM将在运行期也保留注解，因此可以通过反射机制读取注解的信息。

3.@Documented

将此注解包含在JavaDoc中

4.@Inherited

允许子类继承父类中的注解

Class，Method，Field等类都实现了AnnotatedElement接口

#### 注解元素可用的类型

八大基本类型
String
Class
enum
Annotaion
以上的数组
注解允许嵌套

#### 自定义注解

        //Age
        @Target({ ElementType.METHOD, ElementType.FIELD })
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Age {
            public int value() default 0;
        }
        //Hometown
        @Target({ ElementType.METHOD, ElementType.FIELD })
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Hometown {
            public String value() default "";
        }
        //Name
        @Target({ ElementType.METHOD, ElementType.FIELD })
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Name {
            public String value() default "";
        }
        //User
        public class User {
            @Name(value = "lcj")
            private String name;

            @Hometown(value = "henan")
            private String hometown;

            @Age(value = 22)
            private Integer age;
            ...

            getter/setter
            @Override
            public String toString() {
                return MoreObjects.toStringHelper(User.class).add("name", this.getName()).add("hometown", this.getHometown())
                    .add("age", this.getAge()).toString();
            }
        }

        @Test
        public void getUserAnnotation() {
            User user = new User();
            Field[] fields = User.class.getDeclaredFields();
            for (Field field : fields) {
                field.setAccessible(true);
                if ("name".equals(field.getName())) {
                    Name name = field.getAnnotation(Name.class);
                    user.setName(name.value());
                    System.out.println(name.value());
                }
                if ("hometown".equals(field.getName())) {
                    Hometown hometown = field.getAnnotation(Hometown.class);
                    user.setHometown(hometown.value());
                    System.out.println(hometown.value());
                }
                if ("age".equals(field.getName())) {
                    Age age = field.getAnnotation(Age.class);
                    user.setAge(age.value());
                    System.out.println(age.value());
                } else {
                    continue;
                }
            }
            System.out.println(user);
        }


### shallow copy & deep copy

1.  浅拷贝，对于基本类型和String，没有问题，独立的一份，对于对象，只是拷贝了引用，共用同一片内存区域
2.  深拷贝，两对象独立

参考

[1]<http://javaconceptoftheday.com/difference-between-shallow-copy-vs-deep-copy-in-java/>

[2]<https://www.cs.utexas.edu/~scottm/cs307/handouts/deepCopying.htm>


String.format格式化输出%?

 %%

所有的Serializable的类必须含有serialVersionUID属性，这是出于性能考虑，如果没有serialVersionUID属性，jre会自己计算一个值，这个值的计算很消耗资源。
能double check最好double check，真的，不要让别人的不靠谱性影响自己的不靠谱性
加了事务的方法，不要catch异常，抛出来才能rollback

httpclient使用完后要release
文件打开之后要close
封装的线程上下文要remove

   

    
