---
layout: post
title: spring代理(译)
categories: java_web
tags: spring proxy
---

spring Aop使用jdk动态代理或者cglib代理来为实例创建代理。（更推荐使用jdk动态代理）。

如果目标对象实现了至少一个接口，那么jdk动态代理将会被使用。所有被这个目标类实现的接口都将被代理。如果目标对象没有实现任何的接口，那么cglib代理将被创建。

如果你想强制使用cglib代理（例如，想代理目标对象中所有的方法，而不仅仅是实现的接口的方法），你可以做这些，但是有一些问题需要考虑：

final 方法不能被advised，因为他们不能被重写
从3.2版本，cglib被包含org.srpingframework  的spring-croe jar包中
为了强制使用cglib代理，将<aop:config>中proxy-target-class设为true

    <aop:config proxy-target-class="true">
        <!-- other beans defined here... -->
    </aop:config>

在使用@Aspectj自动代理时，为了强制使用cglib，需要设置<aop:aspectj-autoproxy>的属性‘proxy-target-class’为true。

    <aop:aspectj-autoproxy proxy-target-class="true"/>
    
使用proxy-target-class=true 在<tx:annotation-driven/>,<aop:aspectj-autoproxy/>,<aop:config/>任一，会使cglib代理在三者都生效。
