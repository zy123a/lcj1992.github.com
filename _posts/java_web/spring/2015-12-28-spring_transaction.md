---
layout: post
title: spring事务
categories: java_web
tags: spring transaction
---

*   [常用spring-mybatis的数据库配置](#common_config)


### 常用spring-mybatis的数据库配置 {#common_config}

不借助框架,自己连接数据库,我们通常是这么干的,以mysql为例:

1.  实例化一个Driver对象  class.forName("com.mysql.jdbc.Driver")
2.  获取一个连接, DriverManager.getConnection("jdbc:mysql://localhost:3306/test?xxxxxx", "test", "test")
3.  创建一个statement / PrepareStatement, connect.createStatement()
4.  执行sql,获取结果集ResultSet,解析结果集
5.  关闭resultSet,关闭connection

有了spring mybatis我们通常是这么配置的:
  
数据源设置
  
1.  声明(可以不实例化)parentDatasource, 设置driver的className,连接池大小,超时时间等
2.  实例化各子DataSource(如果有多数据源)parent指定为1声明的 `parent="parentDatasource"`,然后设置该数据源的url,username,password等.
3.  如果配置了多数据源,通常我们还是设置dataSources路由,dynamicDataSource [做法见](/2015/12/28/spring_databases)

mybatis设置

1.  实例化sqlSessionFactory, 设置一般有这四个`dataSource`,`mapperLocations`,`typeAliasesPackage`,`typeHandlersPackage`:
    *   dataSource(单一dataSource或者DynamicDataSource)
    *   mapperLocations mapper的位置
    *   typeAliasesPackage  省略包名
    *   typeHandlersPackage 自己定制的类型处理器
2.  实例化mapperScannerConfigurer ,指定basePackage,指定dao包的位置.

***spring的事务是与线程绑定的***

mode=“proxy”使用spring的动态代理，但是这个事务有点限制

1.  对于类内的方法调用，事务不生效
2.  只作用于public方法

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
