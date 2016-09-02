---
layout: post
title: spring事务
categories: java_web
tags: spring transaction
---

*   [常用spring-mybatis的数据库配置](#common_config)
    *   [原理](#origin)
    *   [有了spring和mybatis](#spring-mybatis)
        * [数据源](#datasource)
        * [mybatis](#mybatis)
        * [事务配置](#transaction)
        * [spring事务中两种的mode](#transaction_mode)
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
     //con.rollback();


#### 有了spring,mybatis {#spring-mybatis}

我们通常是这么配置的,必须清楚知道的是框架只是帮我们封装了这些，里边还是那么掉的:

##### 数据源设置 {#datasource}

1.  声明(可以不实例化`abstract="true"`)parentDatasource, 设置driver的className,连接池大小,超时时间等
2.  实例化各子DataSource(如果有多数据源)parent指定为1中声明的 `parent="parentDatasource"`,然后设置该数据源的url,username,password等.
3.  如果配置了多数据源,通常我们还是设置dataSources路由,dynamicDataSource [做法见](/2015/12/28/tables_databases#spring_databases)

##### mybatis设置 {#mybatis}

1.  实例化sqlSessionFactory, 设置一般有这四个`dataSource`,`mapperLocations`,`typeAliasesPackage`,`typeHandlersPackage`:
    *   `dataSource`(单一dataSource或者DynamicDataSource)
    *   `mapperLocations` mapper的位置
    *   `typeAliasesPackage`  省略包名
    *   `typeHandlersPackage` 自己定制的类型处理器
2.  实例化mapperScannerConfigurer ,指定basePackage, 各种dao包的位置.

ps:  datasource建立连接的内部实现还是跟[原理](#origin)类似

##### 事务配置 {#transaction}

1.  声明式事务 实例化一个`DataSourceTransactionManager`,然后加上`<<tx:annotation-driven transaction-manager="transactionManager">` 实例化AnnotationTransactionAttributeSource 扫描@Transactional注解
2.  编程式事务 不care

#### spring事务中两种的mode {#transaction_mode}

1.  mode=“proxy” 默认的使用spring的动态代理（jdk or cglib），但是这种方式有点限制,对于类内的方法调用，事务即使声明了不生效（考虑spring aop的原理）
2.  mode=“aspectj” 类内类外都生效，需引入spring-aspects和aspectjrt包，使用aspectj编译器进行编译，编译时织入。具体配置见[附](#appendix)

比较说明：

    <tx:annotation-driven transaction-manager="transactionManager" mode="proxy"/>
    <tx:annotation-driven transaction-manager="transactionManager" mode="aspectj"/>

    class TestService{
        @Resource
        XxService xxService；
        public void test(){
            // methodA的事务在两种mode中都生效
            xxService.methodA();
            // methodC的事务仅在aspectj mode下生效
            xxService.methodB();
        }
    }

    @Service
    public class XxService{

        @Transactional
        public void methodA(){...}

        public void methodB(){
            methodC();
        }

        @Transactional
        public void methodC(){...}
    }

### 事务流程 {#how_to_work}

涉及的几个重要类:

`TransactionAspectSupport`  事务的具体操作都在这里类中进行。它又一个属
性`TransactionInfoHolder`，封装了事务的上下文TransactionInfo。

`DataSourceTransactionManager` implement PlatformTransactionManager  事务管理器

`TransactionInfo`  事务信息

`AnnotationTransactionAttributeSource` implement TransactionAttributeSource      提供有方法来扫注解@Transactional

`TransactionStatus` implement SavepointManager

`TransactionInterceptor` extends TransactionAspectSupport implement

`MethodInterceptor`  事务拦截器

`ConnectionHolder` 封装java.sql.connection

`TransactionSynchronizationManager` 各种ThreadLocal的事务的变量

`TransactionSynchronization`

`DelegatingTransactionAttribute`  动态代理transactionAttribute

`RuleBasedTransactionAttribute`

![原理](/images/java_web/spring_transaction_source.png)

### spring中事务的传播特性 {#propagation}

|传播特性|含义|场景|
|-|-|-|
|PROPAGATION_REQUIRED|如果存在一个事务，则支持当前事务。如果没有事务则开启|默认的。|
|PROPAGATION_SUPPORTS|如果存在一个事务，则支持当前事务。如果没有事务，则非事务的执行||
|PROPAGATION_MANDATORY|如果存在一个事务，则支持当前事务。如果没有一个活动的事务，则抛出异常。||
|PROPAGATION_REQUIRES_NEW|总是开启一个新的事务。如果一个事务已经存在，则将这个存在的事务挂起。||
|PROPAGATION_NOT_SUPPORTED|总是非事务地执行，并挂起任何存在的事务。||
|PROPAGATION_NEVER|总是非事务地执行，如果存在一个活动事务，则抛出异常||
|PROPAGATION_NESTED|如果一个活动的事务存在，则运行在一个嵌套的事务中. 如果没有活动事务, 则按TransactionDefinition.PROPAGATION_REQUIRED 属性执行||

#### aspectj配置 {#appendix}

使用aspectj编译器编译pom中配置

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>4.3.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjrt</artifactId>
        <version>1.8.7</version>
    </dependency>

    <build>
    <finalName>web</finalName>
    <plugins>
       <plugin>
           <groupId>org.codehaus.mojo</groupId>
           <artifactId>aspectj-maven-plugin</artifactId>
           <version>1.8</version>
           <executions>
               <execution>
                   <goals>
                       <goal>compile</goal>
                       <goal>test-compile</goal>
                   </goals>
               </execution>
           </executions>
           <configuration>
               <complianceLevel>1.8</complianceLevel>
               <source>1.8</source>
               <target>1.8</target>
               <encoding>UTF-8</encoding>
               <aspectLibraries>
                   <aspectLibrary>
                       <groupId>org.springframework</groupId>
                       <artifactId>spring-aspects</artifactId>
                   </aspectLibrary>
               </aspectLibraries>
           </configuration>
       </plugin>
    </plugins>
    </build>
