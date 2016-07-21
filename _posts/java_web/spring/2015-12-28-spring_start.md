---
layout: post
title: spring启动
categories: java_web
tags: spring ServletContext RootWebApplicationContext
---

### 概念关系

ServletContext，Root WebApplicationContext，以及每个disPatcherServlet对应的WebApplicationContext。

#### ServletContext

本文以tomcat为例，tomcat的容器分为四个等级，真正管理Servlet的容器的是Context容器，一个Context对应一个web应用。

这个Context提供全局的上下文环境。而这个Context就是ServletContext

下是关于ServletContext的官方描述。

    /**
     * Defines a set of methods that a servlet uses to communicate with its servlet
     * container, for example, to get the MIME type of a file, dispatch requests, or
     * write to a log file.
     *
     * There is one context per "web application" per Java Virtual Machine. (A
     * "web application" is a collection of servlets and content installed under a
     * specific subset of the server's URL namespace such as /catalog
     * and possibly installed via a .war file.)
     *
     * In the case of a web application marked "distributed" in its deployment
     * descriptor, there will be one context instance for each virtual machine. In
     * this situation, the context cannot be used as a location to share global
     * information (because the information won't be truly global). Use an external
     * resource like a database instead.
     *
     * The ServletContext object is contained within the
     *{@link ServletConfig}object, which the Web server provides the servlet when
     * the servlet is initialized.
     */

#### Root WebApplicationContext

spring Ioc容器。其对应的bean定义的配置由context-param标签指定

#### 每个Servlet对应的WebApplicationContext

每个Servlet对应有自己的WebApplicationContext，共用Root WebApplicationContext中的bean。但其之间相互独立。

### 启动流程

首先build tomcat源码，然后拷一个web工程，debug

#### Root WebApplicationContext启动

web.xml中的启动遵循context-param》listener》filter》servlet。
在web容器启动时，会去读取context-param和listener节点，用于向ServletContext提供键值对。

而spring的listener是contextLoaderListener。它会去监听初始化事件，并调用contextInitialized方法

    /**
    *Initialize the root web application context.
    */
    public void contextInitialized(ServletContextEvent event) {
    this.contextLoader = createContextLoader();
    if (this.contextLoader == null)
    { this.contextLoader = this; }
    this.contextLoader.initWebApplicationContext(event.getServletContext());
    }

event.getServletContext()拿到事件中的ServletContext(ServletContext是一个接口，这里具体实现为ApplicationContext)

ApplicationContext的官方描述

    /**
    * Standard implementation of ServletContext that represents
    * a web application's execution environment. An instance of this class is
    * associated with each instance of StandardContext.
    */

后调用initWebApplicationContext()方法(在ClassLoaderListener.java中)，

实例化根Web应用上下文(WebApplicationContext也是一个接口，这里的实现为XmlWebApplicationContext)，

并将其注入ServletContext(具体是实现为ApplicationContext)容器中

    servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
    //键为 WebApplicationContext.class.getName() + ".ROOT";

至此ContextLoaderListener监听器初始化完毕。

#### DispatcherServlet对应WebApplicationContext

然后开始初始化xml配置中的Servlet，以DispatcherServlet为例。

调用initWebApplicationContext()方法(在FrameWorkServlet.java中)

    WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;

这里的rootContext即Spring Ioc容器(Root WebApplicationContext)，并初始化WebApplicationContext。

先找是否存在

    wac = findWebApplicationContext();

没有的话，创建

    wac = createWebApplicationContext(rootContext);

创建过程中，会获取Root WebApplicationContext作为自己的parent上下文，并读取Servlet名字-servlet.xml中信息，实例化相应的bean

    wac.setParent(parent);
    ......
    configureAndRefreshWebApplicationContext(wac);

最后注入ServletContext(具体实现为ApplicationContext)容器中

    getServletContext().setAttribute(attrName, wac);
    //键为org.springframework.web.servlet.FrameworkServlet.CONTEXT.web

spring ioc容器读取相应的xsd文件，读取bean的attributes（class类名）和properties（属性）等，然后反射根据类名和属性等实例化相应的bean，加入到容器中。

done!



InitializingBean: AbstractAutowireCapableBeanFactory#invokeInitMethods

在spring初始化bean的时候，如果该bean是实现了InitializingBean接口，并且同时在配置文件中指定了init-method，系统则是先调用afterPropertiesSet方法，然后在调用init-method中指定的方法。

aop
动态代理： 运行时，生成新的代理类
    cglib: 代理具体的类时，spring会采用这种代理方式[cglib](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop-api.html#aop-pfb-proxy-types)
    java动态代理：只能代理接口。
静态织入：编译时候，把class改了。
    aspectj:

CglibAopProxy-> DynamicAdvisedInterceptor#intercept


Constructor > @PostConstruct > InitializingBean > init-method，构造函数最优先

ApplicationListener<ContextRefreshEvent>
InitializingBean
ApplicationContextAware
BeanFactoryAware

参考：

[1]<http://blog.csdn.net/c289054531/article/details/9196149>

[2]<https://www.ibm.com/developerworks/cn/java/j-lo-servlet/>
