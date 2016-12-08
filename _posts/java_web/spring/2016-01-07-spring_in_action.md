---
layout: post
title: spring实战
categories: java_web
tags: spring bean context cglib dynamicProxy
---
* TOC
{:toc}

IOC和AOP是spring框架最核心的部分。

spring ioc容器来管理类，减少耦合。本文以我们web开发中最常使用的XmlWebApplicationContext为例，分析下spring的ioc容器的启动及处理细节。

# xmlwc的类图


# bean的生命周期

![bean的生命周期](/images/java_web/spring_beans_lifecycle.jpg)

- spring对bean进行实例化
- spring将值和bean的引用注入到bean对应的属性中
- 如果bean实现了BeanNameAware接口，spring将bean的ID传递给setBeanName()方法。
- 如果bean实现了BeanFactoryAware接口，spring将调用setBeanFactory()方法，将BeanFacotry容器实例传入
- 如果bean实现了ApplicationContextAware接口，spring将调用setApplicationContext()方法，将bean所在的应用上下文的引用传入进来
- 如果bean实现了BeanPostProcessor接口，spring将调用他们的postProcessBeforeInitialization()方法
- 如果bean实现了InitializationBean接口，spring将调用他们的afterPropertiesSet()方法，类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
- 如果bean实现了BeanPostProcessor接口，spring将调用他们的postProcessorAfterInitialization()方法
- 此时，bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁
- 如果bean实现了disposableBean接口，spring将调用它的destory()接口方法，同样，如果bean使用destory-method声明了销毁方法，该方法也会被调用

# 实战 {#ref}

```
1.<context:component-scan base-package="com.xxx">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
----------
2.<context:annotation-config/>
```

关于spring-bean的扫描有两个配置需要说下：

- context:component-scan

  1. 扫描注解@Component、@Service等实例化bean，
  2. 扫描注解@Resource、@Autowired等，管理依赖

- context:annotation-config

  1. 扫描注解@Resource、@Autowired等，管理依赖

  也就是说声明了context:annotation-config，还需要在xml中显式的声明bean

> `<context:annotation-config>` is used to activate annotations in beans already registered in the application context (no matter if they were defined with XML or by package scanning).

> `<context:component-scan>` can also do what `<context:annotation-config>` does but `<context:component-scan>` also scans packages to find and register beans within the application context.

> both tags register the same processing tools (`<context:annotation-config />` can be omitted if `<context:component-scan>` is specified) but Spring takes care of running them only once.

> I found this nice summary of which annotations are picked up by which declarations. By studying it you will find that `<context:component-scan/>` recognizes a superset of annotations recognized by `<context:annotation-config/>`, namely: @Component, @Service, @Repository, @Controller, @Endpoint @Configuration, @Bean, @Lazy, @Scope, @Order, @Primary, @Profile, @DependsOn, @Import, @ImportResource. As you can see

> <context:component-scan> logically extends <context:annotation-config> with CLASSPATH component scanning and Java @Configuration features.</context:annotation-config></context:component-scan>


ApplicationListener<ContextRefreshEvent>

Constructor > @PostConstruct > InitializingBean > init-method，构造函数最优先

# 参考 {#ref}

[1.component-scan-vs-annotation-config](http://stackoverflow.com/questions/7414794/difference-between-contextannotation-config-vs-contextcomponent-scan)
