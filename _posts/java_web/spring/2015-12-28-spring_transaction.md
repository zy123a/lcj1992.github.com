---
layout: post
title: spring事务
categories: java_web
tags: spring transaction
---

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
