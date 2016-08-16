---
layout: post
title: 设计模式
categories: design_pattern
tags: proxy factory adapter facade decorator
---

### 创建型 {#create}

 下图中的例子都来自[示例代码](https://github.com/lcj1992/learn/tree/master/java/designPattern)
、`jdk`、[gitbook](https://quanke.gitbooks.io/design-pattern-java/content/)或者[wikipedia](https://en.wikipedia.org/wiki/Software_design_pattern)

ps： 下边的gitbook xxx是指的gitbook中的相应的例子。

|模式|概述|备注or例子|详述|
|-|-|-|-|-|-|
|单例|限制一个类只能有一个实例|`java.lang.Runtime`|1.[wiki](https://en.wikipedia.org/wiki/Singleton_pattern)   2.[me](/2016/07/26/singleton)|
|简单工厂|不指定具体的对象，而通过工厂来创建对象,工厂与产品一对多,工厂方法里if-else来确定生产哪个产品|`java.util.Calendar#getInstance()`||
|工厂方法|不指定具体的对象，而通过工厂来创建对象,工厂与产品一对一|`Collection#iterator()`|1.[wiki](https://en.wikipedia.org/wiki/Factory_method_pattern)   2.[me](/2016/07/26/factory)|
|抽象工厂|跟工厂方法类似，不过抽象工厂可以创建一组对象（有关联的），工厂与产品组一对一|`GUIFactory和WinFactory`|1.[wiki](https://en.wikipedia.org/wiki/Abstract_factory_pattern)   2.[me](/2016/07/26/abstract_factory)|
|建造者|创建模块化的复杂的对象|`gitbook游戏角色`|1.[wiki](https://en.wikipedia.org/wiki/Builder_pattern)     2.[me](/2016/07/26/builder)|
|原型|得到一个对象的拷贝|`gitbook工作日报`,深拷贝vs浅拷贝|1.[wiki](https://en.wikipedia.org/wiki/Prototype_pattern)   2.[me](/2016/07/26/prototype)

### 行为型 {#behavior}

|模式|概述|备注or例子|详述|
|-|-|-|-|-|-|
|责任链|链式，流程需分步进行，且各步之间有层次关系，1执行完成执行2，2执行完成执行3|`报销`,`代购模拟支付`,`java.util.logging.Logger#log()`|1.[wiki](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern) 2.[me](/2016/07/26/chain_of_responsibility)|
|命令|invoker，command，receiver三种角色,command需持有receiver的引用|`开关、打开和关闭操作、灯泡`|1.[wiki](https://en.wikipedia.org/wiki/Command_pattern) 2.[me](/2016/07/26/command)|
|迭代器|太多了|Iterator|1.[wiki](https://en.wikipedia.org/wiki/Iterator_pattern) 2.[me](/2016/07/26/iterator)|
|解释器|..|..|1.[wiki](https://en.wikipedia.org/wiki/Interpreter_pattern) 2.[me](/2016/07/26/interpreter)|
|中介者|mediator中介者、被中介的双方或三方。中介者需持有被中介者的引用，被中介者需持有中介者的引用（真别扭），需声明一个联络方法|`猎头、我、公司们 猎头就是个中介者`|1.[wiki](https://en.wikipedia.org/wiki/Mediator_pattern) 2.[me](/2016/07/26/mediator)|
|备忘录|在捕获一个对象内部状态，并在该对象之外保存这个状态，在以后可以使对象恢复到原先保存的状态|`gitbook象棋悔棋`|1.[wiki](https://en.wikipedia.org/wiki/Memento_pattern) 2.[me](/2016/07/26/memento)|
|观察者|Observer、Subject，subject持有所有Observers的引用，Observer持有subject的引用|`java.util.Observable  java.util.Observer`|1.[wiki](https://en.wikipedia.org/wiki/Observer_pattern) 2.[me](/2016/07/26/observer)|
|状态|Context、State，Context持有State的引用，其内部方法会因State的不同而采取不同的行为||1.[wiki](https://en.wikipedia.org/wiki/State_pattern) 2.[me](/2016/07/26/state)|
|策略|类图感觉跟状态模式一样，感觉要解决的问题不一样（好别扭）|`Collections.sort(list,comparator)`|1.[wiki](https://en.wikipedia.org/wiki/Strategy_pattern) 2.[me](/2016/07/26/strategy)|
|模板方法|定义一个算法的框架，子类可以实现具体的某些步骤（abstract的方法）。|太多了,AbstractCollection#isEmpty()|1.[wiki](https://en.wikipedia.org/wiki/Template_method_pattern) 2.[me](/2016/07/26/template)|
|访问者|Vistor、Element、Elements，不同的vistor实现visit同一Element，结果不一样，不同的visitor关注点不同，Elements提供遍历操作|gitbook `hr和财务访问员工list`|1.[wiki](https://en.wikipedia.org/wiki/Visitor_pattern) 2.[me](/2016/07/26/visitor)|

### 结构型 {#structure}

|模式|概述|备注or例子|详述|
|-|-|-|-|-|-|
|适配器|Adapter、Adaptee、Target，（对象适配器）Adapter继承自Target，持有Adaptee的引用（对象适配器）；除此之外还有类适配器和双向适配器|`官网代购的各个adapter`|1.[wiki](https://en.wikipedia.org/wiki/Adapter_pattern) 2.[me](/2016/07/26/adapter)|
|桥接|多维度变化，其中一个维度持有其他维度的引用|`gitbook PNG,JPG格式图片和Win，Mac，Linux操作系统`|1.[wiki](https://en.wikipedia.org/wiki/Bridge_pattern) 2.[me](/2016/07/26/bridge)|
|组合|对待组合和对待元素是一样一样的。组合和元素实现统一接口||1.[wiki](https://en.wikipedia.org/wiki/Composite_pattern) 2.[me](/2016/07/26/composite)|
|装饰|如果只增强一层的话，和代理模式类图是一样一样的，关注点不同。可以不断的装饰|InputStream、FilterInputStream、BufferedInputStream、DataInputStream|1.[wiki](https://en.wikipedia.org/wiki/Decorator_pattern) 2.[me](/2016/07/26/decorator)|
|门面|封装，对外提供api|`gitbook自己泡茶和去茶馆喝茶`|1.[wiki](https://en.wikipedia.org/wiki/Facade_pattern) 2.[me](/2016/07/26/facade)|
|享元|大量细粒度对象的重用|`缓存`，`Integer#valueOf()`|1.[wiki](https://en.wikipedia.org/wiki/Flyweight_pattern) 2.[me](/2016/07/26/flyweight)|
|代理|代理类和被代理类实现统一接口，代理类持有被代理类引用，在实现接口方法时改变被代理类的行为|静态代理，spring动态代理|1.[wiki](https://en.wikipedia.org/wiki/Proxy_pattern) 2.[me](/2016/07/26/proxy)|

### 参考 {#ref}

[1]<http://linuxtools-rst.readthedocs.org/zh_CN/latest/>

[wikipedia]<https://en.wikipedia.org/wiki/Design_pattern>

[gitbook]<https://quanke.gitbooks.io/design-pattern-java/content/>

[图说设计模式]<http://design-patterns.readthedocs.io/zh_CN/latest/>
