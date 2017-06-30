---
title: MySQL性能优化
date: 2016-11-09 15:48:37
# 类别
categories:
  - 技术
# 标签
tags:
  - 运维
author: liudong
---
作者：liudong at 2016-11-09 15:48:37
##1、为查询优化你的查询
大多数的MySQL服务器都开启了查询缓存。这是提高性最有效的方法之一，而且这是被MySQL的数据库引擎处理的。当有很多相同的查询被执行了多次的时候，这些查询结果会被放到一个缓存中，这样，后续的相同的查询就不用操作表而直接访问缓存结果了。

```
// 查询缓存不开启
$r = mysql_query("SELECT username FROM user WHERE     signup_date >= CURDATE()");

// 开启查询缓存
$today = date("Y-m-d");
$r = mysql_query("SELECT username FROM user WHERE signup_date >= '$today'");
```
<!--more-->
区别： CURDATE() ，MySQL的查询缓存对这个函数不起作用。所以，像 NOW() 和 RAND() 或是其它的诸如此类的SQL函数都不会开启查询缓存，因为这些函数的返回是会不定的易变的。所以，你所需要的就是用一个变量来代替MySQL的函数，从而开启缓存。

##2、EXPLAIN 你的SELECT查询
使用EXPLAIN关键字可以让你知道MySQL是如何处理你的SQL语句的。

有表关联的查询，如下列：
```
select username, group_name
from users u
joins groups g on (u.group_id = g.id)
```
发现查询缓慢，然后在group_id字段上增加索引，则会加快查询

##3、当只要一行数据时使用LIMIT 1
当你查询表的有些时候，你已经知道结果只会有一条结果，单因为你可能需要去fetch游标，或是你也许会去检查返回的记录数。
在这种情况下，加上LIMIT 1 可以增加性能。这样一样， MySQL数据库引擎会在找到一条数据后停止搜索，而不是继续往后查找下一条符合记录的数据。
下面的示例，只是为了找一下是否有“成都”的用户，很明显，后面的会比前面的更有效率。（请注意，第一条中是Select *，第二条是Select 1）
```
// 没有效率的：
$r = mysql_query("SELECT * FROM user WHERE country = 'China'");
if (mysql_num_rows($r) > 0) {
    // ...
}

// 有效率的：
$r = mysql_query("SELECT 1 FROM user WHERE country = 'China' LIMIT 1");
if (mysql_num_rows($r) > 0) {
// ...
}
```

##4、为搜索字段建索引
索引并不一定就是给主键或是唯一的字段。如果在你的表中，有某个字段你总要会经常用来做搜索，那么，请为其建立索引吧。
```
添加PRIMARY KEY（主键索引）
mysql>ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )

添加UNIQUE(唯一索引)
mysql>ALTER TABLE `table_name` ADD UNIQUE (
`column`
)

添加INDEX(普通索引)
mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column` )


添加FULLTEXT(全文索引)
mysql>ALTER TABLE `table_name` ADD FULLTEXT (
`column`
)

添加多列索引
mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )
```
##5、在Join表的时候使用相当类型的列，并将其索引
如果你的应用程序有很多JOIN查询，你应该确认两个表中Join的字段是被建过索引的。这样，MySQL内部会启动为你优化Join的SQL语句的机制。
而且，这些被用来Join的字段，应该是相同的类型的。例如：如果你要把DECIMAL字段和一个INT字段JOIN在一起，MYSQL就无法使用他们的索引。对于那些STRING类型，还需要有相同的字符集才行（两个表的字符集有可能不一样）

##6、千万不要ORDER BY RAND()
##7、避免SELECT *
从数据库里读出越多的数据，那么查询就会变得越慢。并且，如果你的数据库服务器和WEB服务器是两台独立的服务器的话，这还会增加网络传输的负载。
```
// 不推荐
$r = mysql_query("SELECT * FROM user WHERE user_id = 1");
$d = mysql_fetch_assoc($r);
echo "Welcome {$d['username']}";

// 推荐
$r = mysql_query("SELECT username FROM user WHERE user_id = 1");
$d = mysql_fetch_assoc($r);
echo "Welcome {$d['username']}";
```
##8、永远为两张表设置一个ID
为数据库里的每张表都设置一个ID作为其主键，而最好的是一个INT型（推荐使用UNSIGNED），并设置上自动增长的AUTO INCREMENT标志。
就算是你 users 表有一个主键叫 “email”的字段，你也别让它成为主键。使用 VARCHAR 类型来当主键会使用得性能下降。另外，在你的程序中，你应该使用表的ID来构造你的数据结构。
而且，在MySQL数据引擎下，还有一些操作需要使用主键，在这些情况下，主键的性能和设置变得非常重要，比如，集群，分区

##9、使用 ENUM 而不是 VARCHAR ？
ENUM 类型是非常快和紧凑的。在实际上，其保存的是 TINYINT，但其外表上显示为字符串。这样一来，用这个字段来做一些选项列表变得相当的完美。

如果你有一个字段，比如“性别”，“国家”，“民族”，“状态”或“部门”，你知道这些字段的取值是有限而且固定的，那么，你应该使用 ENUM 而不是 VARCHAR。

##10、从 PROCEDURE ANALYSE() 取得建议 ？
PROCEDURE ANALYSE() 会让 MySQL 帮你去分析你的字段和其实际的数据，并会给你一些有用的建议。只有表中有实际的数据，这些建议才会变得有用，因为要做一些大的决定是需要有数据作为基础的。

例如，如果你创建了一个 INT 字段作为你的主键，然而并没有太多的数据，那么，PROCEDURE ANALYSE()会建议你把这个字段的类型改成 MEDIUMINT 。或是你使用了一个 VARCHAR 字段，因为数据不多，你可能会得到一个让你把它改成 ENUM 的建议。这些建议，都是可能因为数据不够多，所以决策做得就不够准。

##11、尽可能的使用 NOT NULL
摘自MySQL官方文档
“NULL columns require additional space in the row to record whether their values are NULL. For MyISAM tables, each NULL column takes one bit extra, rounded up to the nearest byte.”

##12、把IP地址存成 UNSIGNED INT
很多使用者都会创建一个 VARCHAR(15) 字段来存放字符串形式的IP而不是整形的IP。如果你用整形来存放，只需要4个字节，并且你可以有定长的字段。而且，这会为你带来查询上的优势，尤其是当你需要使用这样的WHERE条件：IP between ip1 and ip2。

##13、拆分大的 DELETE 或 INSERT 语句
如果有一个大的处理，你定你一定把其拆分，使用 LIMIT 条件是一个好的方法。
下面是一个示例：
```
while (1) {
//每次只做1000条
mysql_query("DELETE FROM logs WHERE log_date <= '2009-11-01' LIMIT 1000");
if (mysql_affected_rows() == 0) {
    // 没得可删了，退出！
    break;
}
// 每次都要休息一会儿
usleep(50000);
```
##14、当查询较慢的时候，可用Join来改写一下该查询来进行优化
```
mysql> select sql_no_cache * from guang_deal_outs where deal_id in (select id from guang_deals where id = 100017151) ;
 Empty set (18.87 sec)

mysql> select sql_no_cache a.* from guang_deal_outs a inner join guang_deals b on a.deal_id = b.id where b.id = 100017151;
 Empty set (0.01 sec)

原因
mysql> desc select sql_no_cache * from guang_deal_outs where deal_id in (select id from guang_deals where id = 100017151) ;
+----+--------------------+-----------------+-------+---------------+---------+---------+-------+----------+-------------+
| id | select_type        | table           | type  | possible_keys | key     | key_len | ref   | rows     | Extra       |
+----+--------------------+-----------------+-------+---------------+---------    +---------+-------+----------+-------------+
|  1 | PRIMARY            | guang_deal_outs | ALL   | NULL          | NULL    |     NULL    | NULL  | 18633779 | Using where |
|  2 | DEPENDENT SUBQUERY | guang_deals     | const | PRIMARY       | PRIMARY |     4       | const |        1 | Using index |
+----+--------------------+-----------------+-------+---------------+---------    +---------+-------+----------+-------------+
2 rows in set (0.04 sec)

mysql> desc select sql_no_cache a.* from guang_deal_outs a inner join guang_deals b on a.deal_id = b.id where b.id = 100017151;
+----+-------------+-------+-------+----------------------    +----------------------+---------+-------+------+-------------+
| id | select_type | table | type  | possible_keys        | key                      | key_len | ref   | rows | Extra       |
+----+-------------+-------+-------+----------------------    +----------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | b     | const | PRIMARY              | PRIMARY                  | 4       | const |    1 | Using index |
|  1 | SIMPLE      | a     | ref   | idx_guang_dlout_dlid |     idx_guang_dlout_dlid | 4       | const |    1 |             |
+----+-------------+-------+-------+----------------------    +----------------------+---------+-------+------+-------------+
 2 rows in set (0.05 sec)
```
##15、子查询时用exists而不是用in
```
不推荐in
select * from guang_deal_outs where deal_id in (select id from guang_deals where id = 100017151);
推荐exists
select * from guang_deal_outs where exists (select * from guang_deals where id = 100017151);
```