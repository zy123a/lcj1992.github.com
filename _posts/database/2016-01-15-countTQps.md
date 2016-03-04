---
layout: post
title: 计算tps,qps
categories: db
tags: tps qps mysql
---

如果数据库中存在比较多的myisam表，则计算还是questions 比较合适。
如果数据库中存在比较多的innodb表，则计算以com_*数据来源比较合适。

### 方法一
基于 questions  计算qps,基于  com_commit  com_rollback 计算tps

`questions = show global status like 'questions'`;

`uptime = show global status like 'uptime'`;

`qps=questions/uptime`

`com_commit = show global status like 'com_commit';`

`com_rollback = show global status like 'com_rollback';`

`uptime = show global status like 'uptime';`

`tps=(com_commit + com_rollback)/uptime`


### 方法二
基于 com_* 的status 变量计算tps ,qps
使用如下命令：

`show global status where variable_name in('com_select','com_insert','com_delete','com_update');`

获取间隔1s 的 com_*的值，并作差值运算

`del_diff = (int(mystat2['com_delete'])   - int(mystat1['com_delete']) ) / diff`

`ins_diff = (int(mystat2['com_insert'])    - int(mystat1['com_insert']) ) / diff`

`sel_diff = (int(mystat2['com_select'])    - int(mystat1['com_select']) ) / diff`

`upd_diff = (int(mystat2['com_update'])   - int(mystat1['com_update']) ) / diff`

参考

[1]<http://blog.itpub.net/22664653/viewspace-767265/>