---
layout: post
title: redis源码分析
categories: soft
tags: redis ds
---

* [内存结构](#memory)
* [对象](#obj)
* [数据结构](#ds)
  * [sds](#sds)
  * [list](#list)
  * [dict](#dict)
  * [skipList](#skipList)
  * [intSet](#intSet)
  * [ziplist](#zipList) 
* [调试redis](#debug_redis)
* [redis启动流程](#redis_start)
* [redis接收请求](#redis_accept_request)
* [redis多机](#redis_distributed)
* [使用场景](#redis_situation)


### 内存结构 {#memory}

首先需要明确的是redis是一个key-value存储。

1.  key就是字符串，而value的可以是:string、list、map、set、sort set、(bitmaps、hyperloglogs、geospatial indexes)。
2.  各种value对应的是字符串对象、list object(列表对象)、hash object(哈希对象)、set object(集合对象)、sorted set object(有序集合对象)。
3.  每一种对象又是由这些数据结构组成的:sds(c string的变种)、list、dict、skipList、intSet、 zipList。
4.  一种对象可以底层对应多种数据结构实现，eg列表对象底层可能是list或者ziplist，可以进行转换。

其结构图如下：

![redis结构](/images/soft/redisServer.png)

### 对象 {#obj}

    typedef struct redisObject {
    
        // 类型          
        unsigned type:4;
    
        // 编码
        // 
        unsigned encoding:4;
    
        // 对象最后一次被访问的时间
        unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */
    
        // 引用计数
        int refcount;
    
        // 指向实际值的指针
        void *ptr;
    
    } robj;

type

|对象|对象type属性的值|type命令的输出|
|-|-|
|字符串对象|REDIS_STRING|"string"|
|列表对象|REDIS_LIST|"list"|
|哈希对象|REDIS_HASH|"hash"|
|集合对象|REDIS_SET|"set"|
|有序集合对象|REDIS_ZSET|"zset"|

encoding

|编码常量|编码所对应的底层数据结构|
|-|-|
|REDIS_ENCODING_INT|long类型的整数|
|REDIS_ENCODING_EMBSTR|embstr编码：的简单动态字符串|
|REDIS_ENCODING_RAW|简单动态字符串|
|REDIS_ENCODING_HT|字典|
|REDIS_ENCODING_LINKEDLIST|双端链表|
|REDIS_ENCODING_ZIPLIST|压缩列表|
|REDIS_ENCODING_INTSET|整数集合|
|REDIS_ENCODING_SKIPLIST|跳跃表和字典|

type和encoding可以相互结合.具体结合方式可参照开头的[大图](#memory)。

eg:

1.  redisObject的type属性为REDIS_STRING、encoding为REDIS_ENCODING_INT表明使用整数值实现字符串对象

2.  redisObject的type属性为REDIS_HASH、encoding为REDIS_ENCODING_ZIPLIST表明使用压缩列表实现的哈希对象;encoding为REDIS_ENCODING_HT表明使用双向链表实现的哈希。

可以使用type和object encoding来查看redis对象的类型type和底层的数据结构类型encoding

    127.0.0.1:6379> set msg "xx"
    OK
    127.0.0.1:6379> type msg
    string
    127.0.0.1:6379> object encoding msg
    "embstr"

### 数据结构 {#ds}

redis底层的数据结构有这么几种:sds、list、dict、skipList、intSet、 zipList。

1.  对象的底层实现，比如列表对象listObject其底层可能就是使用的list数据结构。

2.  服务器的一些其他功能的底层实现也是这些数据结构，比如redisServer保持多个客户端的状态信息，监视器等功能底层实现也是list；redis本身就是个kv存储系统，redis数据库的底层就是使用了dict，然后dict对象底层也是用的dict，有点蒙了，可以对照上图。

#### sds {#sds}

sds: simple dynamic string

    struct sdshdr {
    
        // 记录 buf 数组中已使用字节的数量
        // 等于 SDS 所保存字符串的长度
        int len;
    
        // 记录 buf 数组中未使用字节的数量
        int free;
    
        // 字节数组，用于保存字符串
        char buf[];
    
    }; 


#### list {#list}

    typedef struct listNode {
    
        // 前置节点
        struct listNode *prev;
    
        // 后置节点
        struct listNode *next;
    
        // 节点的值
        void *value;
    
    } listNode;


    typedef struct list {
    
        // 表头节点
        listNode *head;
    
        // 表尾节点
        listNode *tail;
    
        // 链表所包含的节点数量
        unsigned long len;
    
        // 节点值复制函数
        void *(*dup)(void *ptr);
    
        // 节点值释放函数
        void (*free)(void *ptr);
    
        // 节点值对比函数
        int (*match)(void *ptr, void *key);
    
    } list;

#### dict(hash) {#dict}

    typedef struct dict {
    
        // 类型特定函数
        dictType *type;
    
        // 私有数据
        void *privdata;
    
        // 哈希表
        dictht ht[2];
    
        // rehash 索引
        // 当 rehash 不在进行时，值为 -1
        int rehashidx; /* rehashing not in progress if rehashidx == -1 */
    
    } dict;
    
    typedef struct dictType {
    
        // 计算哈希值的函数
        unsigned int (*hashFunction)(const void *key);
    
        // 复制键的函数
        void *(*keyDup)(void *privdata, const void *key);
    
        // 复制值的函数
        void *(*valDup)(void *privdata, const void *obj);
    
        // 对比键的函数
        int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    
        // 销毁键的函数
        void (*keyDestructor)(void *privdata, void *key);
    
        // 销毁值的函数
        void (*valDestructor)(void *privdata, void *obj);
    
    } dictType;

    typedef struct dictht {
    
        // 哈希表数组
        dictEntry **table;
    
        // 哈希表大小
        unsigned long size;
    
        // 哈希表大小掩码，用于计算索引值
        // 总是等于 size - 1
        unsigned long sizemask;
    
        // 该哈希表已有节点的数量
        unsigned long used;
    
    } dictht;
    
    typedef struct dictEntry {
    
        // 键
        void *key;
    
        // 值
        union {
            void *val;
            uint64_t u64;
            int64_t s64;
        } v;
    
        // 指向下个哈希表节点，形成链表
        struct dictEntry *next;
    
    } dictEntry;

扩容与收缩、链地址法。

渐进式rehash

#### skipList {#skipList}

    typedef struct zskiplist {
    
        // 表头节点和表尾节点
        struct zskiplistNode *header, *tail;
    
        // 表中节点的数量
        unsigned long length;
    
        // 表中层数最大的节点的层数
        int level;
    
    } zskiplist;


    typedef struct zskiplistNode {
    
        // 后退指针
        struct zskiplistNode *backward;
    
        // 分值
        double score;
    
        // 成员对象
        robj *obj;
    
        // 层
        struct zskiplistLevel {
    
            // 前进指针
            struct zskiplistNode *forward;
    
            // 跨度
            unsigned int span;
    
        } level[];
    
    } zskiplistNode;

跳跃表的介绍。

#### intSet  {#intSet}

    typedef struct intset {
    
        // 编码方式
        uint32_t encoding;
    
        // 集合包含的元素数量
        uint32_t length;
    
        // 保存元素的数组
        int8_t contents[];
    
    } intset;

类型转换

#### zipList {#zipList}

    typedef struct zlentry {
    
        // prevrawlen ：前置节点的长度
        // prevrawlensize ：编码 prevrawlen 所需的字节大小
        unsigned int prevrawlensize, prevrawlen;
    
        // len ：当前节点值的长度
        // lensize ：编码 len 所需的字节大小
        unsigned int lensize, len;
    
        // 当前节点 header 的大小
        // 等于 prevrawlensize + lensize
        unsigned int headersize;
    
        // 当前节点值所使用的编码类型
        unsigned char encoding;
    
        // 指向当前节点的指针
        unsigned char *p;
    
    } zlentry;

连锁更新

### 调试redis {#debug_redis}

使用vi，ctags，cscope，gdb进行调试 参见[我是这样看源码的](/2016/02/28/view_source)

因为我们要调试，所以还是[使用源码进行安装](/2015/12/28/redis_basic#hello)

1. 启动服务端redis-server
2. 启动客户端redis-cli
3. gdb attach到服务进程上`gdb -p pid`，这种方式可以调试redis接收请求的过程，但是必须redis服务器启动起来。所以如果观察redis的启动流程正确的姿势应该是gdb可执行文件
4. 打断点，对不同数据结构进行断点调试
ps: mac上的gdb还需设置下: http://jingyan.baidu.com/article/925f8cb8fa362ec0dde0561a.html

执行过程如下：
    
    cd /Users/lichuangjian/soft/redis-3.0-annotated
    make install
    gdb ./src/redis-server
    b 3952
    r

![gdb_redis](/images/soft/gdb_redis.png)

### redis启动流程 {#redis_start}

redis.c中有两个全局变量
struct RedisServer * server、 struct RedisCommand *commandTable

1. 初始化redisServer的配置initServerConfig()  

2. 如果指定了配置文件，加载配置文件，修改默认配置loadServerConfig()

3. 打开监听端口listenToPort()

4. 打开unix本地端口anetUnixServer()

5. 创建数据库并进行初始化（默认是16个）

6. 创建pubSub相关数据结构

7. 初始化server的其他一些配置

8. 为 serverCron() 创建时间事件aeCreateTimeEvent()

9. 为 TCP 连接关联连接应答（accept）处理器aeCreateFileEvent()

10. 为本地套接字关联应答处理器aeCreateFileEvent()

11. 如果AOF功能打开，打开或创建aof文件

12. 如果服务器以cluster方式运行，初始化cluster, clusterInit()

13. 初始化复制功能有关的脚本缓存replicationScriptCacheInit()

14. 初始化脚本系统scriptingInit()

15. 初始化慢查询功能slowlogInit()

16. 初始化BIO系统bioInit()

17. 从aof或者rdb文件中载入数据

18. 启动集群？

19. 运行事件处理器。

Q&A ?

1. 如何将其设置为守护进程的。

### redis接受请求 {#redis_accept_request}

### redis多机  {#redis_distributed}

### redis使用场景 {#redis_situation}

### 参考 {#ref}

[redis设计与实现]<http://redisbook.com/>

[mit公开课算法导论:跳跃表]<http://open.163.com/movie/2010/12/7/S/M6UTT5U0I_M6V2TTJ7S.html>
