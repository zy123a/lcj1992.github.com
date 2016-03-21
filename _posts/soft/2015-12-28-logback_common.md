---
layout: post
title: logback日常总结
categories: soft java
tags: logback asyncAppender 
---

*   打印异常

LOGGER.error("aa{}",e); 是错误的

*   性能

     logger.debug("Entry number: " + i + " is " +  String.valueOf(entry[i]));
     logger.debug("Entry number: {} is {}", i, entry[i]);

前者无论是否打都会出发字符串拼接，所以要用后者。

*   异步appender

在十线程并发下，输出200字符的INFO日志，AsyncAppender的吞吐量最高能是FileAppender的3.7倍

    <appender name="search" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${catalina.base}/logs/search.log</File>
        <encoder>
            <pattern>%d|%X{sys_name}_web|%X{_monitor_cookie}|%m%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${catalina.base}/logs/search.log.%d{yyyy-MM-dd-HH}</fileNamePattern>
            <maxHistory>5</maxHistory>
        </rollingPolicy>
    </appender>
    
    
    <!-- 异步输出 -->
    <appender name ="asyncSearch" class= "ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold >100</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>512</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref ="search"/>
    </appender>

*   分布式系统中一个请求可能会经过多个不同的子系统，这是最好生成一个UUID附在请求中，每个子系统在大一日志时都将给UUID放在MDC里，便于以后打日志

[1]<http://www.infoq.com/cn/articles/things-of-java-log-performance>
