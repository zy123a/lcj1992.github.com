---
layout: post
title: redis基础
categories: soft
tags: redis ds 数据结构
---


* [基本用法](#basic)
* [数据结构与对象](#ds_object)

### 基本用法 {#basic}

***首先明确一点这是一个key-value缓存系统，key是字符串，而这个value可以是多样的：string，map，list，set，sort set，bitmaps，hyperLogLogs***

    // 连接
    redis-cli  -h host -p port

    // 查看手册
    help set

    // 字符串
    set name lcj

    // hash
    hmset user:1 name fuck birth 18400101 phone 15945612345(对象,多个域)
    hset user:1 name fuck（一个域）
    hget user:1 name

    // 列表
    lpush lcj aa
    lrange a 0 10

    // 集合
    sadd lcj aa
    sadd lcj bb
    smembers lcj

    // 有序集合
        key     score       member
    zadd     9C.order  -235  235|sueccess
    zadd     9C.order - 234  234|fail
    zrange    9C.order 0 -1

    // 位图
    key   offset  0|1
    setbit active_user 001 1
    setbit active_user 002 0
    setbit active_user 003 1
    bitcount active_user
    getbit active 003

    // hyperLogLog
    PFADD ip '192.168.0.1'
    PFADD ip '192.168.0.1'
    PFCOUNT ip

除此之外还有发布订阅，事务的支持

    //一端
    subscribe heheda
    //另一端
    publish heheda helloIamHeheda

    //事务
    multi
    ...
    exec

###  数据结构与对象 {#ds_object}


### 参考 {#ref}

[redis设计与实现]<http://redisbook.com/>

[mit公开课算法导论:跳跃表]<http://open.163.com/movie/2010/12/7/S/M6UTT5U0I_M6V2TTJ7S.html>
