---
layout: post
title: 单例模式
categories: design_pattern
tags: singleton
---

*   [lazy 懒汉式](#lazy)
    *   [简单版本](#lazy_easy)
    *   [双重检查](#double_check)
    *   [holder](#holder)
*   [eager 饿汉式](#eager)

对于singleton有`lazy`方式和`eager`方式。对于一些依赖参数或者配置文件的，在getInstance()之前必须调用某个方法设置参数给它，那就必须是用lazy的方式，而不能使用eager方式。

### lazy

1.  instance必须声明为volatile，保证初始化的过程中，不会发生重排序。
2.  保证check and act的原子性。

#### 简单版本 {#lazy_easy}

    public class SimpleVersion {
        private static volatile SimpleVersion instance = null;
        private SimpleVersion() {}
        public static synchronized SimpleVersion getInstance() {
            if (instance == null) {
                instance = new SimpleVersion();
            }
            return instance;
        }
    }

#### 双重检查 {#double_check}

    public class DoubleCheckVersion {
        // 必须声明为volatile, 禁止重排序
        private static volatile DoubleCheckVersion instance;
        private DoubleCheckVersion() {}
        public static DoubleCheckVersion getInstance() {
            // 先判断下，避免读也需要同步，同步是个很重的操作啊
            if (instance == null) {
                // 先check后act，这应该是个原子操作，所以使用synchronized保证原子性
                synchronized (DoubleCheckVersion.class) {
                    if (instance == null) {
                        instance = new DoubleCheckVersion();
                    }
                }
            }
            return instance;
        }
    }

ps：

    instance = new DoubleCheckVersion();
这句并不是原子的操作， 事实上在jvm层面大概做了三件事：

1.  给instance分配内存
2.  调用instance的构造函数来初始化成员变量，形成实例
3.  将instance对象指向分配的内存空间（执行完这步，instance才不是null）

如果instance不声明为volatile，则在jvm的即时编译器中可能存在指令重排序的优化，也就是说存在1-3-2的情形发生，这样instance已经不为null，但是并没有初始化成员变量呢，被别的线程引用到那就呵呵了。

#### holder

    public class InitOnDemandHolderVersion {
        private InitOnDemandHolderVersion() {}

        private static class SingletonHolder {
            private static final InitOnDemandHolderVersion instance = new InitOnDemandHolderVersion();
        }
        private static InitOnDemandHolderVersion getInstance() {
            return SingletonHolder.instance;
        }
    }

1.  延迟加载，静态内部类在单例类被加载时并不会立即实例化，而是在在需要实例化时，调用getInstance(),才会装载SingletonHolder，从而完成InitOnDemandHolderVersion的实例化
2.  线程安全，在类进行初始化时，别的线程是无法进入的，jvm层面保证了线程安全。

### eager

1.  instance声明为final的，在构造完成之前引用已经完全确定了

eager的实现方式有enum版本的，还有最简单的直接实例化的,这里就不说了,详见[单例](https://github.com/lcj1992/learn/tree/master/java/designPattern/src/main/java/creational/singleton)。

### 参考 {#ref}

[深入浅出单实例Singleton设计模式]<http://coolshell.cn/articles/265.html>

[wikipedia]<https://en.wikipedia.org/wiki/Singleton_pattern>
