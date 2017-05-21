---
layout: post
title: springmvc+slf4j+log4j2的集成
categories: log
tags: log4j2 springmvc
---
[TOC]

#### 引入相关jar包
```
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
```  
#### 配置文件     
     
**配置文件必须放置到ClassPath下供log4j2初始化能够寻找到，否者需要在web.xml里面指定日志配置位置**  
```
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration status="warn">
  
      <Properties>
          <Property name="log_collect_path">/Users/zhengyin</Property>
      </Properties>
  
      <appenders>
          <!--输出日志到控制台-->
          <Console name="stdout" target="SYSTEM_OUT">
              <PatternLayout pattern="%5p [%t] %d{yyyy-MM-dd HH:mm:ss.SSS} %l %m %n"/>
          </Console>
  
          <!--输出日志到指定位置，appender为回滚-->
          <RollingRandomAccessFile name="info"
                                   fileName="${log_collect_path}/info.log"
                                   filePattern="${log_collect_path}/info.log.%d{yyyy-MM-dd}">
              <PatternLayout
                      pattern="%5p [%t] %d{yyyy-MM-dd HH:mm:ss.SSS} %l %m %n"/>
               <Policies>
                <!--默认一天一翻滚-->
                <TimeBasedTriggeringPolicy/>
                <!--当日志文件大于100翻滚-->
                <SizeBasedTriggeringPolicy size="100 MB"/>
                </Policies>
              <!--当日志文件超过了20个开始删除-->
              <DefaultRolloverStrategy max="20"/>
          </RollingRandomAccessFile>
      </appenders>
      
      <loggers>
          <root level="info" >
              <appender-ref ref="stdout"/>
              <appender-ref ref="info"/>
          </root>
      </loggers>
  </configuration>
  ```
  
#### 使用  
  
  ``` 
  // 注解的方式
  @Slf4j
  public class Test {
  //    直接获取slf4j的logger对象进行日志的输出
  //    private static final Logger log = LoggerFactory.getLogger(Test.class);
  
      public static void main(String[] args) {
          log.info("测试info日志输出");
          log.warn("测试warn日志输出");
          log.error("测试error日志输出");
      }
  }
  ```
  

  
