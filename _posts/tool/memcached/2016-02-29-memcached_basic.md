---
layout: post
title: memcached基础
categories: tool
---

### 使用

连接
    
    telnet host port

增改

    command <key> <flags> <expiration time> <bytes>
    <value>

*   set
*   add
*   replace

删查
    
    command <key>

*   get
*   delete

其他

*   flush_all

### 参考

[1] <http://blog.csdn.net/zzulp/article/details/7823511>