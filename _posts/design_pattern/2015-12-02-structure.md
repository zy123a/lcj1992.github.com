---
layout: post
title: 结构型
categories: design_pattern
tags: design_pattern
---

*   [概述](#Summary)
*   [适配器*](#Adapter)
*   [桥接*](#Bridge)
*   [组合*](#Composite)
*   [装饰*](#Decorator)
*   [门面*](#Facade)
*   [享元*](#Flyweight)
*   [FrontController](#FrontController)
*   [Marker](#Marker)
*   [Module](#Module)
*   [代理*](#Proxy)
*   [Twin](#Twin)


<h3 id="Summary">概述</h3>
1.  适配器：将一个接口改成另一个接口
2.  装饰： 通过包装增加接口的功能
3.  门面：提供一个简单的接口

<h3 id="Adapter">适配器*</h3>

将一个接口改成另一个接口

Arrays#asList()

<h3 id="Bridge">桥接*</h3>

`client`和`provider`都经常变化,所以需要`bridge`来桥接

![桥接uml图](http://lcj1992.github.io/images/design_pattern/Bridge.png)

<h3 id="Composite">组合*</h3>

一个组合可以是单一的对象，也可以是一组对象，一个组合可以代表这两种状态

<h3 id="Decorator">装饰*</h3>

装饰

<h3 id="Facade">门面*<h3>

门面

<h3 id="Flyweight">享元*</h3>

菜单上的flavours就是共享的，当new了很多对象，而这些对象好多是一样的，就考虑下享元模式了


            // Menu acts as a staticfactory and cache for CoffeeFlavour flyweight objects
            class Menu {
                private Map<String, CoffeeFlavour> flavours = new ConcurrentHashMap<String, CoffeeFlavour>();

                CoffeeFlavour lookup(String flavorName) {
                    if (!flavours.containsKey(flavorName))
                        flavours.put(flavorName, new CoffeeFlavour(flavorName));
                    return flavours.get(flavorName);
                }

                int totalCoffeeFlavoursMade() {
                    return flavours.size();
                }
            }


<h3 id="FrontController">Front Controller</h3>

front controller

<h3 id="Marker">Marker</h3>

marker

<h3 id="Module">Module</h3>

module

<h3 id="Proxy">代理*</h3>

代理类和被代理类同时实现接口，代理类通过    **组合** 方式代理被代理类，实现   **额外**  的操作

<h3 id="Twin">Twin</h3>

twin

### 参考 {#ref}

[1]<http://linuxtools-rst.readthedocs.org/zh_CN/latest/>


[2]<https://en.wikipedia.org/wiki/Design_pattern>


[图说设计模式]<http://design-patterns.readthedocs.io/zh_CN/latest/structural_patterns/structural.html>
