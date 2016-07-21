---
layout: post
title: spring事务
categories: java_web
tags: spring transaction
---

*   [常用spring-mybatis的数据库配置](#common_config)
    *   [原理](#origin)
    *   [有了spring和mybatis](#spring-mybatis)
*   [spring事务流程](#how_to_work)
    *   [spring中事务的传播特性](#propagation)

### 常用spring-mybatis的数据库配置 {#common_config}

#### 原理 {#origin}

不借助框架,自己连接数据库,我们是这么干的,以mysql为例:

1.  实例化一个Driver对象  class.forName("com.mysql.jdbc.Driver"),参看[code1](#code1)
2.  获取一个连接, DriverManager.getConnection("jdbc:mysql://localhost:3306/test?xxxxxx", "test", "test")，参看[code2](#code2)
3.  创建一个statement / PrepareStatement, connect.createStatement()
4.  执行sql,获取结果集ResultSet,解析结果集
5.  关闭resultSet,关闭connection
6.  使用PrepareStatement，帮助我们避免了sql注入攻击，自动规避了特殊字符；允许我们执行带参数输入的动态查询；支持批量操作
7.  使用事务[code3](#code3)

##### code1

Class.forName()时，先执行static代码块，会先new一个Driver实例，然后注册进DriverManager

    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }

##### code2

在DriverManger#getConnection时，会遍历所有已经注册的Drivers，调用其connect方法进行连接。

    for(DriverInfo aDriver : registeredDrivers) {
        // If the caller does not have permission to load the driver then
        // skip it.
        if(isDriverAllowed(aDriver.driver, callerCL)) {
            try {
                println("    trying " + aDriver.driver.getClass().getName());
                Connection con = aDriver.driver.connect(url, info);
                if (con != null) {
                    // Success!
                    println("getConnection returning " + aDriver.driver.getClass().getName());
                    return (con);
                }
            } catch (SQLException ex) {
                if (reason == null) {
                    reason = ex;
                }
            }

        } else {
            println("    skipping: " + aDriver.getClass().getName());
        }
    }

##### code3

我们这样管理事务

     Class.forName("com.mysql.jdbc.Driver");
     con = DriverManager.getConnection(
             "jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8", "root", "root");
     con.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
     con.setAutoCommit(false);
     String sql = "insert into  name values(?,?)";
     PreparedStatement preparedStatement = con.prepareStatement(sql);
     preparedStatement.setInt(1,1);
     preparedStatement.setString(2,"lcj");
     preparedStatement.execute();
     con.commit();
     con.rollback();


#### 有了spring,mybatis {#spring-mybatis}

我们通常是这么配置的,必须清楚知道的是框架只是帮我们封装了这些，里边还是那么掉的:

a.数据源设置

1.  声明(可以不实例化`abstract="true"`)parentDatasource, 设置driver的className,连接池大小,超时时间等
2.  实例化各子DataSource(如果有多数据源)parent指定为1中声明的 `parent="parentDatasource"`,然后设置该数据源的url,username,password等.
3.  如果配置了多数据源,通常我们还是设置dataSources路由,dynamicDataSource [做法见](/2015/12/28/spring_databases)

b.mybatis设置

1.  实例化sqlSessionFactory, 设置一般有这四个`dataSource`,`mapperLocations`,`typeAliasesPackage`,`typeHandlersPackage`:
    *   `dataSource`(单一dataSource或者DynamicDataSource)
    *   `mapperLocations` mapper的位置
    *   `typeAliasesPackage`  省略包名
    *   `typeHandlersPackage` 自己定制的类型处理器
2.  实例化mapperScannerConfigurer ,指定basePackage, 各种dao包的位置.

c.事务配置

1.  声明式事务 实例化一个`DataSourceTransactionManager`,然后加上`<<tx:annotation-driven transaction-manager="transactionManager">`
2.  编程式事务 ..todo

### 事务流程 {#how_to_work}

例子中以`org.apache.commons.dbcp.BasicDataSource`作为我们的DataSource,其创建连接的原理还是一样的
不详说了,涉及这几个方法,BasicDataSource#getConnection() -> createDataSource() -> createConnectionFactory()

InitializingBean: AbstractAutowireCapableBeanFactory#invokeInitMethods

在spring初始化bean的时候，如果该bean是实现了InitializingBean接口，并且同时在配置文件中指定了init-method，系统则是先调用afterPropertiesSet方法，然后在调用init-method中指定的方法。

aop

cglib和java的动态代理
cglib：编译时织入的代码。
java动态代理： 运行时，生成新的代理类

TransactionDefinition

MethodInterceptor#intercept()
DynamicAdvisedInterceptor
CglibMethodInvocation#proceed()
TransactionInterceptor#invoke
TransactionAspectSupport#invokeWithinTransaction  TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification) txInfo.bindToThread();

ReflectiveMethodInvocation#proceed()

TransactionAspectSupport有一个属性叫做transactionInfoHolder

Constructor > @PostConstruct > InitializingBean > init-method，构造函数最优先

ApplicationListener<ContextRefreshEvent>
InitializingBean
ApplicationContextAware
BeanFactoryAware


对于上述声明式事务的配置方式,

1.  第一步是实例化一个`DataSourceTransactionManager` 事务管理,
2.  第二步实例化`AnnotationTransactionAttributeSource`,扫注解@Transactional

涉及的几个重要类:

*  `TransactionInfo`
*  `DataSourceTransactionManager` implement PlatformTransactionManager
*  `AnnotationTransactionAttributeSource` implement TransactionAttributeSource   determineTransactionAttribute这个方法来扫注解
*  `TransactionStatus` implement SavepointManager
*  `TransactionInterceptor` extends TransactionAspectSupport implement MethodInterceptor
*  `ConnectionHolder`
*  `TransactionSynchronizationManager` 各种ThreadLocal的事务的变量
*  `TransactionSynchronization`

#### spring中事务的传播特性

|传播特性|含义|场景|
|-|-|-|
|PROPAGATION_REQUIRED|如果存在一个事务，则支持当前事务。如果没有事务则开启||
|PROPAGATION_SUPPORTS|如果存在一个事务，则支持当前事务。如果没有事务，则非事务的执行||
|PROPAGATION_MANDATORY|如果存在一个事务，则支持当前事务。如果没有一个活动的事务，则抛出异常。||
|PROPAGATION_REQUIRES_NEW|总是开启一个新的事务。如果一个事务已经存在，则将这个存在的事务挂起。||
|PROPAGATION_NOT_SUPPORTED|总是非事务地执行，并挂起任何存在的事务。||
|PROPAGATION_NEVER|总是非事务地执行，如果存在一个活动事务，则抛出异常||
|PROPAGATION_NESTED|如果一个活动的事务存在，则运行在一个嵌套的事务中. 如果没有活动事务, 则按TransactionDefinition.PROPAGATION_REQUIRED 属性执行||

#### spring中配置事务

mode=“proxy”使用spring的动态代理，但是这种方式有点限制

1.  对于类内的方法调用，事务不生效
2.  只作用于public方法

eg:

    public class ClassA{
         public void testMethod1(){
             //事务不生效
             testTransactionMethod();
         }

         @Transactional
         public void testTransactionMethod(){
         }
    }

    public Class ClassB{
        @Service
        private ClassA classA;

        public void testMethod2(){
        //事务生效
            classA.testTransactionMethod();
        }
    }

mode=“aspectj”上述代码中ClassA和ClassB中的事务都会生效，但是前提是你得用aspectj编译器编译或者<context:load-time-weaver>加载时织入

maven中使用aspectj编译

    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>aspectj-maven-plugin</artifactId>
            <version>1.4</version>
            <dependencies>
                <dependency>
                    <groupId>org.aspectj</groupId>
                    <artifactId>aspectjrt</artifactId>
                    <version>1.6.11</version>
                </dependency>
            </dependencies>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>test-compile</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <Xlint>warning</Xlint>
                <aspectLibraries>
                    <aspectLibrary>
                        <groupId>org.springframework</groupId>
                        <artifactId>spring-aspects</artifactId>
                    </aspectLibrary>
                </aspectLibraries>
                <source>1.6</source>
                <target>1.6</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.bsc.maven</groupId>
            <artifactId>maven-processor-plugin</artifactId>
            <version>2.0.5</version>
            <executions>
                <execution>
                    <id>process</id>
                    <goals>
                        <goal>process</goal>
                    </goals>
                    <phase>process-sources</phase>
                    <configuration>
                        <compilerArguments>-encoding UTF-8</compilerArguments>
                    </configuration>
                </execution>
            </executions>
            <dependencies>
                <dependency>
                    <groupId>org.hibernate</groupId>
                    <artifactId>hibernate-validator-annotation-processor</artifactId>
                    <version>4.2.0.Final</version>
                    <scope>compile</scope>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
