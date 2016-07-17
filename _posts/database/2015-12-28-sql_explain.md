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
|id|select_id|查询语句的id|mysql按照id从小到大的顺序进行解释，实际上执行是按照id从大到小的顺序||
|select_type|none|选择的类型|详见[select_type说明](#select_type)||
|table|table_name|输出列所在的表|详见[table说明](#table)||
|partitions|partitions|匹配到的分区|||
|type|access_type|join的类型|详见[type说明](#type)||
|possible_keys|possiable_keys|可能会选择的索引|||
|key|key|真实选择的索引|显示的是mysql实际使用的索引||
|key_len|key_length|选择的索引的长度|mysql决定使用的索引的长度||
|ref|ref|与索引进行比较的列数|||
|rows|rows|被检查的记录数的估量|||
|filtered|filtered|被查询条件过滤掉的记录数的占比|||
|Extra|none|额外信息|||

#### select_type说明 ｛#select_type}

|类型|对应json格式的key|含义|
|-|-|-|
|simple|none|`简单的查询（没有使用union或者子查询）`|
|primary|none|`最外层的select（多层嵌套子查询）`|
|union|none|多个union时，第二个和更后的查询|
|dependent union|dependent(true)|多个union时，第二个和更后的查询（依赖于外层的查询）|
|union result|union_result|union的结果|
|subquery|none|子查询时的第一个|
|dependent subquery|dependent(true)|子查询时的第一个，依赖于外层查询|
|derived|none|`派生表的select`|
|materialized|materialized_from_subquery||
|uncacheable subquery|cacheable (false)||
|uncacheable union|cacheable (false)||

#### table

M，N都是explain结果中的id字段值

|类型|含义|
|-|-|
|`<unionM,N>`|第m和第n行的对应的table的union结果|
|`<derivedN>` |第n行的派生表的select|
|`<subqueryN>`| The row refers to the result of a materialized subquery for the row with an id value of N|

#### type

|类型|含义|case|
|-|-|-|
|system|表只有一行（系统表），这是const的特殊情形||
|const|表至多匹配一行，用于和主键或者unique index比较时。非常快，因为只读一次|`SELECT * FROM tbl_name WHERE primary_key=1;`|
|eq_ref|相等,用于primary key或者unique not null索引|`SELECT * FROM ref_table,other_table WHERE ref_table.key_column=other_table.column;`|
|ref|key不能匹配一个记录，而是一些记录|`SELECT * FROM ref_table WHERE key_column=expr;`|
|fulltext|处理fulltext索引时||
|ref_or_null|类似ref，多了个null| `SELECT * FROM ref_table WHERE key_column=expr OR key_column IS NULL;`|
|index_merge|||
| unique_subquery|类似eq_ref，只不过不是＝，而是in|`value IN (SELECT primary_key FROM single_table WHERE some_expr)`|
|index_subquery|类似unique_subquery,只不过作用于非unique索引|`value IN (SELECT key_column FROM single_table WHERE some_expr)`|
|range|使用索引，使用操作符`=, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, or IN()`、与常量进行比较 |`SELECT * FROM tbl_name WHERE key_column = 10;`  `SELECT * FROM tbl_name WHERE key_column BETWEEN 10 and 20;`|
|index|扫描索引树,当查询结果可以被索引包含时，就不扫表了，只扫索引树就ok，这时Extra列会有`using index`标示（`覆盖索引`）||
|all|全表扫描|||

注：

1. `覆盖索引`(Covering Index)：只用到某个索引，且该索引包含查询需要的数据列，也就没有回表操作。
2. 覆盖索引必须要存储索引列的值，而哈希索引、空间索引、和全文索引都不能存储列的值，所以MySQL只能使用B-Tree索引做覆盖索引

case1:

    mysql> explain select * from mtp_book_info order by book_id desc limit 10000,10;
    +----+-------------+---------------+-------+---------------+---------+---------+------+-------+-------+
    | id | select_type | table         | type  | possible_keys | key     | key_len | ref  | rows  | Extra |
    +----+-------------+---------------+-------+---------------+---------+---------+------+-------+-------+
    |  1 | SIMPLE      | mtp_book_info | index | NULL          | PRIMARY | 4       | NULL | 10010 |       |
    +----+-------------+---------------+-------+---------------+---------+---------+------+-------+-------+
    1 row in set (0.05 sec)

case2:

    mysql> explain select  * from mtp_book_info where book_id >= (select book_id from mtp_book_info order by book_id  desc limit 10000, 1) order by book_id desc  limit 10;   +----+-------------+---------------+-------+---------------+---------+---------+------+-------+-------------+
    | id | select_type | table         | type  | possible_keys | key     | key_len | ref  | rows  | Extra       |
    +----+-------------+---------------+-------+---------------+---------+---------+------+-------+-------------+
    |  1 | PRIMARY     | mtp_book_info | range | PRIMARY       | PRIMARY | 4       | NULL | 19764 | Using where |
    |  2 | SUBQUERY    | mtp_book_info | index | NULL          | PRIMARY | 4       | NULL | 10001 | Using index |
    +----+-------------+---------------+-------+---------------+---------+---------+------+-------+-------------+
    2 rows in set (0.04 sec)

case3:

    mysql> explain select * from mtp_book_info as book1 join (select book_id from mtp_book_info order by book_id desc limit 10000, 10) as book2 where  book1.book_id = book2.book_id ;
    +----+-------------+---------------+--------+---------------+---------+---------+---------------+-------+-------------+
    | id | select_type | table         | type   | possible_keys | key     | key_len | ref           | rows  | Extra       |
    +----+-------------+---------------+--------+---------------+---------+---------+---------------+-------+-------------+
    |  1 | PRIMARY     | <derived2>    | ALL    | NULL          | NULL    | NULL    | NULL          |    10 |             |
    |  1 | PRIMARY     | book1         | eq_ref | PRIMARY       | PRIMARY | 4       | book2.book_id |     1 |             |
    |  2 | DERIVED     | mtp_book_info | index  | NULL          | PRIMARY | 4       | NULL          | 10010 | Using index |
    +----+-------------+---------------+--------+---------------+---------+---------+---------------+-------+-------------+
    3 rows in set (0.03 sec)

case4:

    mysql> explain select order_display_id from mtp_book_info  union all select order_display_id from book_goods_info;
    +----+--------------+-----------------+-------+---------------+----------------------+---------+------+-------+-------------+
    | id | select_type  | table           | type  | possible_keys | key                  | key_len | ref  | rows  | Extra       |
    +----+--------------+-----------------+-------+---------------+----------------------+---------+------+-------+-------------+
    |  1 | PRIMARY      | mtp_book_info   | index | NULL          | idx_order_display_id | 8       | NULL | 64174 | Using index |
    |  2 | UNION        | book_goods_info | index | NULL          | idx_order_display_id | 8       | NULL | 64147 | Using index |
    | NULL | UNION RESULT | <union1,2>      | ALL   | NULL          | NULL                 | NULL    | NULL |  NULL |             |
    +----+--------------+-----------------+-------+---------------+----------------------+---------+------+-------+-------------+
    3 rows in set (0.04 sec)

## 底层的数据结构

## 参考 {#ref}

[mysql explain详解]<http://www.cnitblog.com/aliyiyi08/archive/2016/04/21/48878.html>
[官方文档]<https://dev.mysql.com/doc/>
[]
