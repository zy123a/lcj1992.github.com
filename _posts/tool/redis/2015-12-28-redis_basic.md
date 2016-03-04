---
layout: post
title: redis基础
categories: tool
tags: redis
---

#### 查看命令
    
    redis-cli  -h host -p port

进入客户端之后

输入help 然后tab键即可查看redis的命令
#### 数据类型
***首先明确一点这是一个key-value缓存系统，这个value可以是string，map，list，set，sort set，bitmaps，hyperLogLogs***

#### 字符串

    set name lcj

#### 哈希值
    
    hmset user:1 name fuck birth 18400101 phone 15945612345(对象,多个域)
    hset user:1 name fuck（一个域）
    hget user:1 name

#### 列表

    lpush lcj aa
    lrange a 0 10

#### 集合
    
    sadd lcj aa
    sadd lcj bb
    smembers lcj

#### 集合排序 
    
        key     score       member
    zadd     9C.order  -235  235|sueccess
    zadd     9C.order - 234  234|fail
    zrange    9C.order 0 -1
 
#### 位图(节省空间)

    key   offset  0|1
    setbit active_user 001 1
    setbit active_user 002 0
    setbit active_user 003 1
    bitcount active_user
    getbit active 003

#### hyperLogLog

    PFADD ip '192.168.0.1'
    PFADD ip '192.168.0.1'
    PFCOUNT ip

#### 发布订阅
一端
    
    subscribe heheda

另一端
    
    publish heheda helloIamHeheda
    
事务

    multi
    ...
    exec
