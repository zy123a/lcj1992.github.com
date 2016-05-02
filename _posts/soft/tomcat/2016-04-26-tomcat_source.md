---
layout: post
title: tomcat的启动与加载server.xml
categories: soft
tags: tomcat source
---

基于7.0.42.0

#### 启动流程

tomcat的入口为`BootStrap#main`

首先看下BootStrap是干什么的?

Catalina的启动加载器,加载catalina.home目录下的classes,并启动容器.

这样保证了Catalina内部的类不在系统的path下,因此对于其他应用是不可见的(应用隔离),自定义类加载器的好处.

    Bootstrap loader for Catalina.  This application constructs a class loader
    for use in loading the Catalina internal classes (by accumulating all of the
    JAR files found in the "server" directory under "catalina.home"), and
    starts the regular execution of the container.  The purpose of this
    roundabout approach is to keep the Catalina internal classes (and any
    other classes they depend on, such as an XML parser) out of the system
    class path and therefore not visible to application level classes.

1.  BootStrap#init,设置catalina.home,设置catalina.base
2.      

[tomcat8官方文档]<https://tomcat.apache.org/tomcat-8.0-doc/config/service.html>

[tomcat的启动和关闭]<http://yikebocai.com/2014/11/tomcat-source-code-study-2/>

[tomcat处理请求]<http://yikebocai.com/2014/11/tomcat-source-code-study-3/>

[tomcat处理HTTP请求源码分析]<http://www.infoq.com/cn/articles/zh-tomcat-http-request-1>