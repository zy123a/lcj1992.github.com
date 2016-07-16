---
layout: post
title: mysql执行计划
categories: db
tags: explain select mysql
---
* [select sql执行顺序](#select)
  * [细解](#sql)
* [explain](#explain)

## select sql执行顺序 {#select}

        (8)SELECT (9)DISTINCT
        <select_list>
        (1)FROM <left_table>
        (3)　<join_type> JOIN <right_table>
        (2)　 ON <join_condition>  
        (4)WHERE <where_condition>
        (5)GROUP BY <group_by_list>
        (6)WITH {CUBE | ROLLUP}
        (7)HAVING <having_condition>
        (10)ORDER BY <order_by_list>
        (11)<LIMIT_specification>


执行顺序:

`笛卡尔积---on---join类型---where---group by---with{cube | rollup}---having---select---distinct---order by---limit`

`limit works on MySQL and PostgreSQL, top works on SQL Server, rownum works on Oracle.`

### 细解 {#sql}

1.  on与where的区别
    两者效果可能一样，但on是连接两表做笛卡尔积时的连接条件，where连接之后的筛选条件。

2.  where 和 having的区别

    1.  where 子句的作用是在对查询结果进行分组前(`group by 之前`)，将不符合where条件的行去掉，即在分组之前过滤数据， ***where条件中不能包含聚集函数***，使用where条件过滤出特定的行。

    2.  having 子句的作用是筛选满足条件的组(`group by 之后`,having是专门搭配group by干活的)，即在分组之后过滤数据，条件中经常包含聚集函数，使用having条件过滤出特定的组，也可以使用多个分组标准进行分组。

3.  join、left join、right join　　
    eg table_a,table_b

    1.  left join 以左为准，右可以为空, 记录数>=table_a总数
    2.  right join 以右为准，左可以为空，记录数>=table_b总数
    3.  如果你有on的连接条件，三个就一样了。

4.  group by

    1.  先对某一列或多列进行分组，然后在分组内进行相应的操作，一般和count，max等聚集函数一起使用。

    2.  聚集函数与列不能同时出现在select子句中，除非这个列是group by子句的分组列,参见下错误例子

    3.  当查询语句中有group by子句时，聚集函数作用的对象是一个分组，而不是整个查询的结果,没有group by的话是整个查询结果

        select DepartmentId,Name,count(Salary) from Employee group by Name;
        Error Code: 1055. Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.Employee.DepartmentId' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

5.  order by

对某一列进行排序,发生在select，distinct之后

6.  limit

取出结果的几个 eg：limit 1,2从1开始的2个。  查询结果是从0 开始的

7.  union和union all

跨库查询，union会去重，union all 不会去重

## 执行计划 {#explain}

explain来解释和分析sql查询语句

    explain format=traditional select * from mtp_book_id in (select max(book_id) from book_goods_info order by id)
![explain](/images/database/explain.png)

ps: format有traditional 和json两种格式

### explain输出字段说明

|字段|对应json格式的key|含义|备注|case|
|-|-|-|-|-|
|id|select_id|查询语句的id|mysql按照id从小到大的顺序进行解释，实际上执行使按照id从大到小的顺序||
|select_type|none|选择的类型|详见[select_type说明](#select_type)||
|table|table_name|输出列所在的表|||
|partitions|partitions|匹配到的分区|||
|type|access_type|join的类型|表示查询结果最多匹配一行，const查询很快，只有在查询的列使用了主键索引或者唯一索引的情况下，进行常数值比较时查询的type才为const||
|possible_keys|possiable_keys|可能会选择的索引|指查询能够使用到的索引，PRIMARY表示查询可以是哟过主键索引||
|key|key|真实选择的索引|显示的是mysql实际使用的索引||
|key_len|key_length|选择的索引的长度|mysql决定使用的索引的长度，该列的类型为bigint类型，所以长度为8||
|ref|ref|与索引进行比较的列数|显示与索引列比较的列，这里为常数，及const||
|rows|rows|被检查的记录数的估量|||
|filtered|filtered|被查询条件过滤掉的记录数的占比|||
|Extra|none|额外信息|||

#### select_type说明 ｛＃select_type｝

|类型|对应json格式的key|含义|
|-|-|-|
|simple|none|简单的查询（没有使用union或者子查询）|
|primary|none|最外层的select（多层嵌套子查询）|
|union|none|多个union时，第二个和更后的查询|
|dependent union|dependent(true)|多个union时，第二个和更后的查询（依赖于外层的查询）|
|union result|union_result|union的结果|
|subquery|none|子查询时的第一个|
|dependent subquery|dependent(true)|子查询时的第一个，依赖于外层查询|
|derived|none|派生表|
|materialized|materialized_from_subquery||
|uncacheable subquery|cacheable (false)||
|uncacheable union|cacheable (false)||

*   当查询的列不是独立的，而是`表达式`或者`函数`的一部分时，这种情况下，mysql将无法使用该列的索引,所以合理的做法是在查之前,就计算出该值

*   当查询的列需要进行`模糊匹配`时，即使该列建了索引，也需要进行全表扫描

*   最早的mysql一次查询只能够使用其中一个索引，5.0即以后，mysql引入了索引合并（index merge）的策略，但是完全可以通过建立多列索引来避免索引合并给数据库带来额外的开销,以达到更好的性能。
