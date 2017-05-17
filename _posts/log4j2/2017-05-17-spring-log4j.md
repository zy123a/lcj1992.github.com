---
layout: post
title: log4j2与springmvc的集成
categories: log
tags: log4j2 springmvc
---
* toc
[: toc]

## 引入相关jar包
     <!--日志系统-->
     <!-- 将common-log牵引到slf4j-->  
     <dependency>  
       <groupId>org.slf4j</groupId>  
       <artifactId>jcl-over-slf4j</artifactId>
       <version>1.7.9</version>
     </dependency>
     
     <!-- slf4j核心包-->
     <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-api</artifactId>
       <version>1.7.9</version>
     </dependency>
     
     <!--核心log4j2jar包-->
     <dependency>
       <groupId>org.apache.logging.log4j</groupId>
       <artifactId>log4j-api</artifactId>
       <version>2.8.2</version>
     </dependency>
     <dependency>
       <groupId>org.apache.logging.log4j</groupId>
       <artifactId>log4j-core</artifactId>
       <version>2.8.2</version>
     </dependency>
     
     <!--log4j 桥接slf4j-->
     <dependency>
       <groupId>org.apache.logging.log4j</groupId>
       <artifactId>log4j-slf4j-impl</artifactId>
       <version>2.8.2</version>
     </dependency>
     
     <!--web工程需要包含log4j-web，非web工程不需要-->
     <dependency>
       <groupId>org.apache.logging.log4j</groupId>
       <artifactId>log4j-web</artifactId>
       <version>2.8.2</version>
     </dependency>
