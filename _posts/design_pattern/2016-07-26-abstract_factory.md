---
layout: post
title: 抽象工厂模式
categories: design_pattern
tags: abstract_factory
---

### 概述

抽象工厂创建具体的工厂,每一个工厂创建多种产品，这些个产品是相互关联的。

![产品族](/images/design_pattern/abstract_factory.jpeg)
1.  工厂方法中具体工厂与具体的产品是一对一的关系
2.  抽象工厂方法中具体工厂与具体的产品族是一对一的关系

### 类图

抽象工厂、具体工厂、抽象产品1、具体产品1、抽象产品2、具体产品2

GUIFactory、WinFactory、Label、WinLabel、Button、WinButton

![类图](/images/design_pattern/abstract_factory.png)

[代码举例](https://github.com/lcj1992/learn/tree/master/java/designPattern/src/main/java/creational/abstractFactory)
