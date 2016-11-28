---
layout: post
title: spring ioc启动流程
categories: java_web
tags: spring ioc ServletContext
---

* TOC
{:toc}

### ContextLoaderListener
 
    <?xml version="1.1" encoding="UTF-8"?>
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
    <display-name>xx Web Application</display-name>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:root-context.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

对于互联网的java后台开发来说，这段配置应该都熟悉。

web.xml中的启动遵循context-param -> listener -> filter -> servlet。spring ioc的容器启动就是从ContextLoaderListener开始的。

    public class ContextLoaderListener extends ContextLoader implements ServletContextListener 

关于ContextLoaderListener

1. 继承自`org.springframework.web.context.ContextLoader`，具备load context的能力，这个context是spring的ioc容器。
2. 实现了`javax.servlet.ServletContextListener`,监听ServletContext的event(init和destory)
   * contextInitialized 在contextInitialized方法中load context(对于XmlWebApplicationContext是initWebApplicationContext方法)
   * contextDestroyed
3. 三种context
   * ServletContext，web容器，可以是tomcat，可以是jetty等
   * Root WebApplicationContext，spring的ioc容器
   * 如果使用了spring mvc，还会有DispatcherServlet对应的webApplicationContext。

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

本文只讨论前Servletcontext和Root WebApplicationContext，dispatcherServlet的context不在本文讨论范围之内。

以web应用使用的WebXmlWebApplicationContext作为切入点分析spring ioc的建立流程，ClassPathXmlWebApplicationContext、FileXmlWebApplicationContext类似。

### XmlWebApplicationContext类图

![xmlWebApplicationContext类图](/images/java_web/xmlwc_uml.png)

从类图中可以看出其实现的接口：

1. ResourceLoader: 具备加载资源的能力
2. BeanFactory: ioc容器的最顶层接口，获取容器中的bean
3. ...

所以对于xmlWebAplicationContext(ApplicationContext)的加载过程可以分为这几步

1. 加载resource, 由context-param标签中的参数指定。
2. 解析resource，向容器中注册resource对应的bean
3. 依据BeanDefinition，实例化bean(lazy-init = false)
4. ...

### 启动流程

#### ContextLoader#initWebApplicationContext

initWebApplicationContext创建webApplicationContext实例，然后configureAndRefreshWebApplicationContext.

    public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {

        ...

		// Store context in local instance variable, to guarantee that
		// it is available on ServletContext shutdown.
		if (this.context == null) {
			// 实例化webApplicationContext,默认为org.springframework.web.context.support.XmlWebApplicationContext
            this.context = createWebApplicationContext(servletContext);
		}
		if (this.context instanceof ConfigurableWebApplicationContext) {
			ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
			if (!cwac.isActive()) {
				// The context has not yet been refreshed -> provide services such as
				// setting the parent context, setting the application context id, etc
				if (cwac.getParent() == null) {
					// The context instance was injected without an explicit parent ->
					// determine parent for root web application context, if any.
					ApplicationContext parent = loadParentContext(servletContext);
					cwac.setParent(parent);
				}
                // 核心代码都在这里边！
				configureAndRefreshWebApplicationContext(cwac, servletContext);
			}
		}
		servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

		ClassLoader ccl = Thread.currentThread().getContextClassLoader();
		if (ccl == ContextLoader.class.getClassLoader()) {
			currentContext = this.context;
		}
		else if (ccl != null) {
			currentContextPerThread.put(ccl, this.context);
		}

		if (logger.isDebugEnabled()) {
			logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" +
					WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]");
		}
		if (logger.isInfoEnabled()) {
			long elapsedTime = System.currentTimeMillis() - startTime;
			logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
		}

		return this.context;
        ....
    }

#### ContextLoader#configureAndRefreshWebapplicationContext

    protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
		
        if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
			// The application context id is still set to its original default value
			// -> assign a more useful id based on available information
			// 如果设置了contextId, 采用设置的，一般我们都没设置，如果没有使用applicationContext的类名+contextPath作为id
            String idParam = sc.getInitParameter(CONTEXT_ID_PARAM);
			if (idParam != null) {
				wac.setId(idParam);
			}
			else {
				// Generate default id...
				wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
						ObjectUtils.getDisplayString(sc.getContextPath()));
			}
		}
        
        // 设置容器所在的web容器上下文ServletContext,见上文。 
		wac.setServletContext(sc);
        // 配置文件的地址，一般我们都会设置的。（文章开头那个root-context.xml）
		String configLocationParam = sc.getInitParameter(CONFIG_LOCATION_PARAM);		
        if (configLocationParam != null) {
			wac.setConfigLocation(configLocationParam);
		}

		// The wac environment's #initPropertySources will be called in any case when the context
		// is refreshed; do it eagerly here to ensure servlet property sources are in place for
		// use in any post-processing or initialization that occurs below prior to #refresh
		ConfigurableEnvironment env = wac.getEnvironment();
		if (env instanceof ConfigurableWebEnvironment) {
			// 初始化配置源，对于web applicationContext就是web容器的根目录,xx/xx/webapp
            ((ConfigurableWebEnvironment) env).initPropertySources(sc, null);
		}

		customizeContext(sc, wac);
		// 重中之重
        wac.refresh();
	}

#### AbstractApplicationContext#refresh

    @Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			// 1.设置启动时间
            // 2.设置active 标志位为true
            // 3.资源的初始化 不太懂！
            prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			// 1. refreshBeanFactory():a.创建beanFactory;b.加载beanDefinitions
            // 2. getBeanFactory()并返回beanFactory
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}

#### XmlApplicationContext#loadBeanDefinitions

 


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

#### ddDispatcherServlet对应WebApplicationContext

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

参考：

[1]<http://blog.csdn.net/c289054531/article/details/9196149>

[2]<https://www.ibm.com/developerworks/cn/java/j-lo-servlet/>
