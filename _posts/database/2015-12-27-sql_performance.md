---
layout: post
title: 数据库查询性能优化
categories: db
tags: performance sql
---

#### 慢日志

查看是否启用慢日志

    show variables like 'log_slow_queries';

查看慢于多少秒的sql会记录到慢日志中

    show variables like ‘long_query_time’;

通过mysql的配置文件my.cnf，可以修改慢日志的相关配置

#### 合理使用索引

*   explain来解释和分析sql查询语句
![explain](/images/database/explain.png)

|字段|说明|
|-|-|
|select_type|SIMPLE表示只是简单查询|
|table|列表示所查询的表|
|type为const|表示查询结果最多匹配一行，const查询很快，只有在查询的列使用了主键索引或者唯一索引的情况下，进行常数值比较时查询的type才为const|
|possible_keys|指查询能够使用到的索引，PRIMARY表示查询可以是哟过主键索引|
|key|显示的是mysql实际使用的索引|
|key_len|mysql决定使用的索引的长度，该列的类型为bigint类型，所以长度为8|
|ref|显示与索引列比较的列，这里为常数，及const|

*   当查询的列不是独立的，而是`表达式`或者`函数`的一部分时，这种情况下，mysql将无法使用该列的索引,所以合理的做法是在查之前,就计算出该值

*   当查询的列需要进行`模糊匹配`时，即使该列建了索引，也需要进行全表扫描

*   最早的mysql一次查询只能够使用其中一个索引，5.0即以后，mysql引入了索引合并（index merge）的策略，但是完全可以通过建立多列索引来避免索引合并给数据库带来额外的开销,以达到更好的性能。

#### 最左前缀原则

*   必须按照顺序（即查询条件中必须出现按照索引声明顺序的列才生效），如idx_site_status_channel

查询条件中 只有site 或者  site，status （status，site）或者site，channel，status 都会走这个索引，从左到右。

例子：

select * from task_info where status = 3 and stie='CA'; 会用到索引

select * from task_info where channel = 3 and status = 'CA'; 不会用到

*   如果查询中某个列使用了范围查询，则其右边所有的列都无法使用索引进行查询优化

    
### 计算qps,tps
 
如果数据库中存在比较多的myisam表，则计算还是questions 比较合适。
如果数据库中存在比较多的innodb表，则计算以com_*数据来源比较合适。
 
#### 方法一
 
基于 questions  计算qps,基于  com_commit  com_rollback 计算tps
 
`questions` = show global status like 'questions';
 
`uptime` = show global status like 'uptime';
 
`com_commit` = show global status like 'com_commit';
 
`com_rollback` = show global status like 'com_rollback';

1.`qps` =questions/uptime 
 
2.`tps` = (com_commit + com_rollback)/uptime
 
 
#### 方法二
 
基于 com_* 的status 变量计算tps ,qps

使用如下命令：
 
`show global status where variable_name in('com_select','com_insert','com_delete','com_update');`
 
获取间隔1s 的 com_*的值，并作差值运算
 
`del_diff` = (int(mystat2['com_delete'])   - int(mystat1['com_delete']) ) / diff
 
`ins_diff` = (int(mystat2['com_insert'])    - int(mystat1['com_insert']) ) / diff
 
`sel_diff` = (int(mystat2['com_select'])    - int(mystat1['com_select']) ) / diff
 
`upd_diff` = (int(mystat2['com_update'])   - int(mystat1['com_update']) ) / diff
 
#### 参考    

[1]<http://coolshell.cn/articles/1846.html>

[2]<http://blog.itpub.net/22664653/viewspace-767265/>
