---
layout: post
title: tomcat的启动关闭与请求处理
categories: soft
tags: tomcat source
---

基于7.0.42.0, tomcat源码如何导入idea[参看这篇](/2015/12/28/import_tomcat_to_idea)

#### 背景

最早看tomcat源码是为了研究spring mvc的启动,然后有了个了解也就放那了.
最近airlineav老是跪(听说之前也是经常跪),决定搞下.
毕竟工作中遇到技术性的问题也不多,再放任其走掉,什么时候能提高,尽自己能力尽可能地深入,总能有所收获.
尝试着改了下tomcat的参数,发现好像有必要研究下它的源码,毕竟这是工作中接触最广的开源项目之一了.

#### 整体架构

#### 启动关闭流程 {#start_stop}

tomcat的入口为`BootStrap#main`

首先看下BootStrap是干什么的?

    Bootstrap loader for Catalina.  This application constructs a class loader
    for use in loading the Catalina internal classes (by accumulating all of the
    JAR files found in the "server" directory under "catalina.home"), and
    starts the regular execution of the container.  The purpose of this
    roundabout approach is to keep the Catalina internal classes (and any
    other classes they depend on, such as an XML parser) out of the system
    class path and therefore not visible to application level classes.

大致意思:Catalina的启动加载器,加载catalina.home目录下的classes,并启动容器.

这样保证了Catalina内部的类不在系统的path下,因此对于其他应用是不可见的(应用隔离),自定义类加载器的好处.

启动:

1.  Bootstrap#init
    
    *   1.1 设置catalina.home,设置catalina.base (ps: [CATALINA_HOME vs CATALINA_BASE](https://tomcat.apache.org/tomcat-8.0-doc/introduction.html#Directories_and_Files))
    *   1.2 初始化类加载器,`commonLoader`,`catalinaLoader`,`sharedLoader`
    *   1.3 用catalinaLoader对`org.apache.catalina.startup.Catalina`类进行加载,并进行实例化startupInstance,设置catalinaDaemon为startupInstance
    *   1.4 在bootstrap初始化完毕之后,设置daemon为bootstrap
    
2.  根据传入命令, 已start为例,加载,调用Bootstrap#load(args),实际上是调用catalinaDaemon#load方法

    *   2.1 Catalina#createStartDigester 创建解析规则, 就是创建Server,Service,Executor等元素与相关类的对应关系,根据其配置实例化相应模块,参见[server.xml](https://github.com/lcj1992/tomcat_study/blob/master/conf/server.xml),
            
            digester.addObjectCreate("Server","org.apache.catalina.core.StandardServer","className");
            digester.addSetProperties("Server");
            digester.addSetNext("Server","setServer","org.apache.catalina.Server");
            
            digester.addObjectCreate("Server/Listener",null, // MUST be specified in the element(必须在你的server.xml中制定)
                                             "className");
            digester.addSetProperties("Server/Listener");
            digester.addSetNext("Server/Listener","addLifecycleListener","org.apache.catalina.LifecycleListener");
            
    *   2.2 读取`$CATALINA_BASE/conf/server.xml`,然后解析之,digester#parse(inputSource),会实例化一个Server,并Catalina#setServer指向这个实例,并将Server#setCatalina指向这个Catalina实例.
    开始调用StandardServer#initInternal,初始化server实例,初始化时会对状态机进行校验  
        *   globalNamingResources的初始化  
        *   依据我们server.xml中的我们的配置,初始化services(可多个)(StandardService)  
            *   初始化Engine(StandardEngine),创建一个线程池,用于启动和停止服务  
            *   初始化Executors(可多个)(StandardThreadExecutor)  
            *   初始化Connectors(可多个)(Connector)  
                *   初始化protocolHandler(eg: Http11Protocol)    
                    *   初始化endpoint(eg JioEndpoint),然后endpoint#bind(),绑定地址和端口,设置线程池的大小,并创建serverSocket EndPoint的bindState 有UNBOUND -> BOUND_ON_START  
                *   初始化mapperListener 
    *   2.3 启动各组件,还是调用Catalina#start(),启动时都会动状态机进行校验
        *   globalNamingResources的启动
        *   启动各services
            *   启动Engine
                *   findChildren() 获取子容器,比如StandardHost的实例,将这些子容器用StartChild类包装成Callable的类，使用线程池启动
                    *   child#start() -> (eg)StandardHost#startInternal() -> setState(LifecycleState.STARTING) -> LifecycleBase#setState -> LifecycleBase#setStateInternal -> LifecycleBase#fireLifecycleEvent(LifecycleBase的实现为EngineConfig)
                        *   HostConfig#deployApps(deploy configBase的xml文件,war包s,加压后的项目s等)
                            *   以DeployWars为例,new一个DeployWar(HostConfig config,ContextName cn,File war),DeployWar实际上是一个runnable,然后将其扔进host.getStartStopExecutor()中
            *   启动Executors
            *   启动connectors
                *   启动protocol
                    *   启动endpoint,初始化connectionLimitLatch,启动acceptorThreads,启动timeoutThread
                *   启动mapperListener, addListeners(engine),registerHost(host)
    *   2.4 如果useShutdownHook为true,添加CatalinaShutdownHook`
    *   2.5 Catalina#await(),new 一个server socket to wait on (默认端口号为8005,你懂的)
                
#### 状态机 {#fsm} 
   
   Lifecycle的实现类都具有如下的状态机,
   
                 start()
       -----------------------------
       |                           |
       | init()                    |
      NEW ->-- INITIALIZING        |
      | |           |              |     ------------------<-----------------------
      | |           |auto          |     |                                        |
      | |          \|/    start() \|/   \|/     auto          auto         stop() |
      | |      INITIALIZED -->-- STARTING_PREP -->- STARTING -->- STARTED -->---  |
      | |         |                                                  |         |  |
      | |         |                                                  |         |  |
      | |         |                                                  |         |  |
      | |destroy()|                                                  |         |  |
      | -->-----<--       auto                    auto               |         |  |
      |     |       ---------<----- MUST_STOP ---------------------<--         |  |
      |     |       |                                                          |  |
      |    \|/      ---------------------------<--------------------------------  ^
      |     |       |                                                             |
      |     |      \|/            auto                 auto              start()  |
      |     |  STOPPING_PREP ------>----- STOPPING ------>----- STOPPED ---->------
      |     |                                ^                  |  |  ^
      |     |               stop()           |                  |  |  |
      |     |       --------------------------                  |  |  |
      |     |       |                                  auto     |  |  |
      |     |       |                  MUST_DESTROY------<-------  |  |
      |     |       |                    |                         |  |
      |     |       |                    |auto                     |  |
      |     |       |    destroy()      \|/              destroy() |  |
      |     |    FAILED ---->------ DESTROYING ---<-----------------  |
      |     |                        ^     |                          |
      |     |     destroy()          |     |auto                      |
      |     -------->-----------------    \|/                         |
      |                                 DESTROYED                     |
      |                                                               |
      |                            stop()                             |
      --->------------------------------>------------------------------
     
#### 参考 {#ref}

[tomcat8官方文档]<https://tomcat.apache.org/tomcat-8.0-doc/config/service.html>

[tomcat的启动和关闭]<http://yikebocai.com/2014/11/tomcat-source-code-study-2/>

[tomcat处理请求]<http://yikebocai.com/2014/11/tomcat-source-code-study-3/>

[tomcat处理HTTP请求源码分析]<http://www.infoq.com/cn/articles/zh-tomcat-http-request-1>