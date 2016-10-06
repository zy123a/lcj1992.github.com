---
layout: post
title: spring bean
categories: java_web
tags: spring bean
---

DI和AOP是spring框架最核心的部分。本文就讲下spring中使用DI装配bean。

### bean的生命周期

![bean的生命周期](/images/java_web/spring_beans_lifecycle.jpg)

*  spring对bean进行实例化
*  spring将值和bean的引用注入到bean对应的属性中
*  如果bean实现了BeanNameAware接口，spring将bean的ID传递给setBeanName()方法。
*  如果bean实现了BeanFactoryAware接口，spring将调用setBeanFactory()方法，将BeanFacotry容器实例传入
*  如果bean实现了ApplicationContextAware接口，spring将调用setApplicationContext()方法，将bean所在的应用上下文的引用传入进来
*  如果bean实现了BeanPostProcessor接口，spring将调用他们的postProcessBeforeInitialization()方法
*  如果bean实现了InitializationBean接口，spring将调用他们的afterPropertiesSet()方法，类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
*  如果bean实现了BeanPostProcessor接口，spring将调用他们的postProcessorAfterInitialization()方法
*  此时，bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁
*  如果bean实现了disposableBean接口，spring将调用它的destory()接口方法，同样，如果bean使用destory-method声明了销毁方法，该方法也会被调用

