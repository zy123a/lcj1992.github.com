---
layout: post
title: spring ioc启动流程
categories: java_web
tags: spring ioc ServletContext
---

* TOC
{:toc}

本文基于spring4.3.1

### 入口-ContextLoaderListener

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

本文只讨论Servletcontext和Root WebApplicationContext.  dispatcherServlet的context不在本文讨论范围之内。

以web应用使用的WebXmlWebApplicationContext作为切入点分析spring ioc的建立流程，ClassPathXmlWebApplicationContext、FileXmlWebApplicationContext类似。

### XmlWebApplicationContext类图

![xmlWebApplicationContext类图](/images/java_web/xmlwc_uml.png)

通过类图，XmlWebApplicationContext实现的接口，我们可以看出其具备的行为：

1. ResourceLoader: 具备加载资源的能力

2. BeanFactory: ioc容器的最顶层接口，获取容器中的bean

3. ...

对应的其处理流程：

1. 加载resource, 由context-param标签中的参数指定。

2. 解析resource，向容器中注册resource对应的bean,即填充beanDefinitionMap等bean的元信息

3. 依据BeanDefinition，实例化bean(lazy-init = false或者在第一次使用bean时)

4. ...

### 启动流程

下边这个方法算是ioc容器启动的主体方法了。
核心方法：

obtainFreshBeanFactory : 加载resource，解析Resource，向beanFactory中填充BeanDefinition等bean的元信息。

finishBeanFactoryInitialization : 实例化非lazy-init的bean


    @Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
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

#### obtainFreshBeanFactory

实例化bean有两种方式：xml中配bean或者扫注解。以component-scan扫注解的为例

首先看下其调用栈，![beandefinitionMap](/images/java_web/ioc_call_stack.png)

1. 从调用栈可以看出，从入口ContextLoaderListener到扫描（scan）涉及的对象有：`ContextLoaderListener`、AbstractApplicationContext`、`AbstractRefreshableApplicationContext`、`XmlWebApplicationContext`、`AbstractBeanDefinitionReader`、`XmlBeanDefinitionReader`、`DefaultBeanDefinitionDocumentReader`、`BeanDefinitionParserDelegate`、`NamespaceHandlerSupport`、`ComponentScanBeanDefinitionParser`、`ClassPathBeanDefinitionScanner`
2. DefaultBeanDefinitionDocumentReader#parseDefaultElement: 默认namespace的解析，（xml） 解析bean、import、beans、alias等标签
3. BeanDefinitionParserDelegate#parseCustomElement: 其他namespace 的解析 （注解），如tx、context等

   * 每个namespace对应有不同的`NamespaceHandler`，eg :tx、context
   * 每个namespaceHandler可能对应多个`BeanDefinitionParser`,eg: ContextNamespaceHandler中包含有PropertyPlaceholderBeanDefinitionParser、ComponentScanBeanDefinitionParser、AnnotationConfigBeanDefinitionParser等用于处理context namespace下的不同的标签。
具体的处理逻辑可以看对应的parser。

下是component-scan扫注解，加载bean配置的时序图（最后打开个新连接看，图太小了）
![spring_ioc_bean_definition](/images/java_web/spring_ioc_beanDefinitions.jpg)

ScopedProxyMode : DEFAULT,NO,INTERFACES,TARGET_CLASS

#### finishBeanFactoryInitialization


### 几个问题

#### xml声明的bean会覆盖注解声明的bean

ClassPathBeanDefinitionScanner#checkCandidate

DefaultListableBeanFactory#registerBeanDefinition

![annotation_xml](/images/java_web/annotation_xml.png)

![xml_annotation](/images/java_web/xml_annotation.png)

#### classpath*与classpath

1.  classpath*: 包含jar中xx.xml 同名的resource按照classpath中的次序拼接成一个大文件后载入，后边的覆盖前边的。

      a. classpath*:xx/*/?   含*或者?通配符的， findPathMatchingResources  eg:classpath*:spring/**/springMVC-*.xml

      b. classpath*:xxx.xml  不含*或者?通配符的，findAllClassPathResources eg:classpath*:application.xml

2.  classpath:  不包含jar中的xml findPathMatchingResources

#### beanName

    public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
		String id = ele.getAttribute(ID_ATTRIBUTE);
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

		List<String> aliases = new ArrayList<String>();
		if (StringUtils.hasLength(nameAttr)) {
			String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			aliases.addAll(Arrays.asList(nameArr));
		}

        // 首先取id
		String beanName = id;
		if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
 			// 如果id为空，name不为空，则取name
            beanName = aliases.remove(0);
			if (logger.isDebugEnabled()) {
				logger.debug("No XML 'id' specified - using '" + beanName +
						"' as bean name and " + aliases + " as aliases");
			}
		}

		if (containingBean == null) {
			checkNameUniqueness(beanName, aliases, ele);
		}

		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
		if (beanDefinition != null) {
			if (!StringUtils.hasText(beanName)) {
				try {
					if (containingBean != null) {
						beanName = BeanDefinitionReaderUtils.generateBeanName(
								beanDefinition, this.readerContext.getRegistry(), true);
					}
					else {
						beanName = this.readerContext.generateBeanName(beanDefinition);
						// Register an alias for the plain bean class name, if still possible,
						// if the generator returned the class name plus a suffix.
						// This is expected for Spring 1.2/2.0 backwards compatibility.
						String beanClassName = beanDefinition.getBeanClassName();
						if (beanClassName != null &&
								beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
								!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
							aliases.add(beanClassName);
						}
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Neither XML 'id' nor 'name' specified - " +
								"using generated bean name [" + beanName + "]");
					}
				}
				catch (Exception ex) {
					error(ex.getMessage(), ele);
					return null;
				}
			}
			String[] aliasesArray = StringUtils.toStringArray(aliases);
			return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
		}

		return null;
	}


参考：

[1]<http://blog.csdn.net/c289054531/article/details/9196149>

[2]<https://www.ibm.com/developerworks/cn/java/j-lo-servlet/>
