---
layout: post
title: 设计模式
categories: clean_code
tags: proxy factory adapter facade decorator
---


* TOC
{:toc}

## 设计原则

### OO基础

1. 抽象

2. 封装

3. 多态

4. 继承

### 设计原则

SOLID

1. [单一职责SRP](https://en.wikipedia.org/wiki/Single_responsibility_principle)
    
    * a class should have only a single responsibility (i.e. only one potential change in the software's specification should be able to affect the specification of the class)

2. [开闭OCP](https://en.wikipedia.org/wiki/Open/closed_principle)
    
    * “software entities … should be open for extension, but closed for modification.”

3. [里式代换LSP](https://en.wikipedia.org/wiki/Liskov_substitution_principle)
    
    * “objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.”

4. [接口隔离ISP](https://en.wikipedia.org/wiki/Interface_segregation_principle)
    
    * “many client-specific interfaces are better than one general-purpose interface.”

5. [依赖倒转DIP](https://en.wikipedia.org/wiki/Dependency_inversion_principle)

    * one should “depend upon abstractions, [not] concretions.”

6. [迪米特法则](https://en.wikipedia.org/wiki/Law_of_Demeter)

7. [合成复用](https://en.wikipedia.org/wiki/Composition_over_inheritance)

### head first设计模式提到的OO原则：

1. 封装变化

2. 多用组合，少用继承 

3. 针对接口编程，不针对实现编程

4. 为交互对象之间的松耦合设计而努力

5. 类应该对扩展开放，对修改关闭

6. 要依赖抽象，不要依赖具体类

7. 只和朋友交谈

8. 别找我，我会找你

9. 类应该只有一个改变的理由

other:

1. 良好的OO设计必须具备可复用，可扩充，可维护三个特性。

2. 大多数的模式和原则，都着眼于软件变化的主题。

3. 大多数的模式都允许系统局部改变独立于其他部分。

ps:

1. 类名取名时，取为对象，英语中一般为er，or，ist等后缀

2. 方法取名时，取为动作，doExecute, createOrder等

3. 减少耦合性，好多都是中间加了一层管理者，比如ioc、桥接、观察者等。

代码的坏味道：

1. 重复，程序中太多的重复代码

2. 太多的if-else

下图中的例子都来自[示例代码](https://github.com/lcj1992/learn/tree/master/java/designPattern)、`jdk`、[gitbook](https://quanke.gitbooks.io/design-pattern-java/content/)或者[wikipedia](https://en.wikipedia.org/wiki/Software_design_pattern)

##设计模式

### 创建型 {#create}

ps： 下边的gitbook xxx是指的gitbook中的相应的例子。

|模式|概述|备注or例子|详述|
|-|-|-|-|-|-|
|单例|限制一个类只能有一个实例|`java.lang.Runtime`|[详](/2016/07/26/singleton)|
|简单工厂|不指定具体的对象，而通过工厂来创建对象,工厂与产品一对多,工厂方法里if-else来确定生产哪个产品|`java.util.Calendar#getInstance()`||
|工厂方法|不指定具体的对象，而通过工厂来创建对象,工厂与产品一对一|`Collection#iterator()`|[详](/2016/07/26/factory)|
|抽象工厂|跟工厂方法类似，不过抽象工厂可以创建一组对象（有关联的），工厂与产品组一对一|`GUIFactory和WinFactory`|[详](/2016/07/26/abstract_factory)|
|建造者|创建模块化的复杂的对象|`gitbook游戏角色`|[详](/2016/07/26/builder)|
|原型|得到一个对象的拷贝|`gitbook工作日报`,深拷贝vs浅拷贝|[详](/2016/07/26/prototype)

### 行为型 {#behavior}

|模式|概述|备注or例子|详述|
|-|-|-|-|-|-|
|责任链|链式，流程需分步进行，且各步之间有层次关系，1执行完成执行2，2执行完成执行3|`报销`,`代购模拟支付`,`java.util.logging.Logger#log()`|[详](/2016/07/26/chain_of_responsibility)|
|命令|invoker，command，receiver三种角色,command需持有receiver的引用|`开关、打开和关闭操作、灯泡`|[详](/2016/07/26/command)|
|迭代器|太多了|Iterator|[详](/2016/07/26/iterator)|
|解释器|..|..|[详](/2016/07/26/interpreter)|
|中介者|mediator中介者、被中介的双方或三方。中介者需持有被中介者的引用，被中介者需持有中介者的引用（真别扭），需声明一个联络方法|`猎头、我、公司们 猎头就是个中介者`|[详](/2016/07/26/mediator)|
|备忘录|在捕获一个对象内部状态，并在该对象之外保存这个状态，在以后可以使对象恢复到原先保存的状态|`gitbook象棋悔棋`|[详](/2016/07/26/memento)|
|观察者|Observer、Subject，subject持有所有Observers的引用，Observer持有subject的引用|`java.util.Observable  java.util.Observer`|[详](/2016/07/26/observer)|
|状态|Context、State，Context持有State的引用，其内部方法会因State的不同而采取不同的行为||[详](/2016/07/26/state)|
|策略|类图感觉跟状态模式一样，感觉要解决的问题不一样（好别扭）|`Collections.sort(list,comparator)`|[详](/2016/07/26/strategy)|
|模板方法|定义一个算法的框架，子类可以实现具体的某些步骤（abstract的方法）。|太多了,AbstractCollection#isEmpty()|[详](/2016/07/26/template)|
|访问者|Vistor、Element、Elements，不同的vistor实现visit同一Element，结果不一样，不同的visitor关注点不同，Elements提供遍历操作|gitbook `hr和财务访问员工list`|[详](/2016/07/26/visitor)|

### 结构型 {#structure}

|模式|概述|备注or例子|详述|
|-|-|-|-|-|-|
|适配器|Adapter、Adaptee、Target，（对象适配器）Adapter继承自Target，持有Adaptee的引用（对象适配器）；除此之外还有类适配器和双向适配器|`官网代购的各个adapter`|[详](/2016/07/26/adapter)|
|桥接|多维度变化，其中一个维度持有其他维度的引用|`gitbook PNG,JPG格式图片和Win，Mac，Linux操作系统`，`java.util.logging.Handler及Filter、Formatter、ErrorManager`|[详](/2016/07/26/bridge)|
|组合|对待组合和对待元素是一样一样的。组合和元素实现统一接口||[详](/2016/07/26/composite)|
|装饰|如果只增强一层的话，和代理模式类图是一样一样的，关注点不同。可以不断的装饰|InputStream、FilterInputStream、BufferedInputStream、DataInputStream|[详](/2016/07/26/decorator)|
|门面|封装，对外提供api|`gitbook自己泡茶和去茶馆喝茶`|[详](/2016/07/26/facade)|
|享元|大量细粒度对象的重用|`缓存`，`Integer#valueOf()`|[详](/2016/07/26/flyweight)|
|代理|代理类和被代理类实现统一接口，代理类持有被代理类引用，在实现接口方法时改变被代理类的行为|静态代理，spring动态代理|[详](/2016/07/26/proxy)|

### 参考 {#ref}

[如何提升代码可读性？其实不是你想的那样](http://www.iteye.com/news/27610)

[wikipedia]<https://en.wikipedia.org/wiki/Design_pattern>

[gitbook]<https://quanke.gitbooks.io/design-pattern-java/content/>

[图说设计模式]<http://design-patterns.readthedocs.io/zh_CN/latest/>

[设计模式]<http://alicharles.com/category/designpattern/>

[oodesign]<http://www.oodesign.com/>

[Software quality](https://en.wikipedia.org/wiki/Software_quality)

[List of system quality attributes](https://en.wikipedia.org/wiki/List_of_system_quality_attributes)

[设计模式六大原则](http://www.uml.org.cn/sjms/201211023.asp)

[SOLID(Object-oriented design)](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))

[The Principles of OOD](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)

[类与类之间的几种关系](http://www.cnblogs.com/liuling/archive/2013/05/03/classrelation.html)

[UML类图几种关系的总结](http://blog.csdn.net/tianhai110/article/details/6339565)

[Examples of GoF Design Patterns in Java's core libraries](http://stackoverflow.com/questions/1673841/examples-of-gof-design-patterns-in-javas-core-libraries)

[依赖倒置原则的“倒置”体现在哪里](https://q.cnblogs.com/q/72496/)

[Dependency Injection Is NOT The Same As The Dependency Inversion Principle](https://lostechies.com/derickbailey/2011/09/22/dependency-injection-is-not-the-same-as-the-dependency-inversion-principle/)

[谈一谈自己对依赖、关联、聚合和组合之间区别的理解](http://www.open-open.com/lib/view/open1427621514639.html)
