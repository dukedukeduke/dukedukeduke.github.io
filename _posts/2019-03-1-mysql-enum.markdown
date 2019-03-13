---
layout: post
title:  "mysql enum 类型"
date:   2019-03-1 18:27:02 +0800
comments: true
tags:
- mysql
- enunm
---

### 参考
https://dev.mysql.com/doc/refman/5.7/en/enum.html

https://segmentfault.com/q/1010000003709270

### 介绍
MySQL 支持enum类型，实际存储得是tinyint。可读性和效率兼具。
举个例子：
首先看mysql这边的设置

```
CREATE TABLE t_order (
    order_id int,
    products varchar(64),
    status ENUM('canceled', 'finished', 'delivering')
);    

insert into t_order(order_id, products, status)
values (1, "笔记本电脑", "canceled"), (2, "华为手机", "finished"), (3, "小米手环", "delivering");

mysql> select * from t_order;
+----------+-----------------+------------+
| order_id | products        | status     |
+----------+-----------------+------------+
|        1 | 笔记本电脑        | canceled   |
|        2 | 华为手机          | finished   |
|        3 | 小米手环          | delivering |
+----------+-----------------+------------+
3 rows in set (0.00 sec)

```

程序端只要按字符串访问就可以

### More

An ENUM is a string object with a value chosen from a list of permitted values that are enumerated explicitly in the column specification at table creation time. It has these advantages:
Compact data storage in situations where a column has a limited set of possible values. The strings you specify as input values are automatically encoded as numbers. See Section 11.8, “Data Type Storage Requirements” for the storage requirements for ENUM types.
Readable queries and output. The numbers are translated back to the corresponding strings in query results.

Creating and Using ENUM Columns
An ENUM column can have a maximum of 65,535 distinct elements.
An enumeration value must be a quoted string literal. For example, you can create a table with an ENUM column like this:

```
CREATE TABLE shirts (
    name VARCHAR(40),
    size ENUM('x-small', 'small', 'medium', 'large', 'x-large')
);
INSERT INTO shirts (name, size) VALUES ('dress shirt','large'), ('t-shirt','medium'),
  ('polo shirt','small');
SELECT name, size FROM shirts WHERE size = 'medium';
+---------+--------+
| name      | size       |
+---------+--------+
| t-shirt     | medium|
+---------+--------+
UPDATE shirts SET size = 'small' WHERE size = 'large';
COMMIT;
```

Inserting 1 million rows into this table with a value of 'medium' would require 1 million bytes of storage, as opposed to 6 million bytes if you stored the actual string 'medium' in a VARCHAR column.

Each enumeration value has an index:
- The elements listed in the column specification are assigned index numbers, beginning with 1.
- The index value of the empty string error value is 0. This means that you can use the following SELECT statement to find rows into which invalid ENUM values were assigned:

```
mysql> SELECT * FROM tbl_name WHERE enum_col=0;
```

- The index of the NULL value is NULL.
- The term “index” here refers to a position within the list of enumeration values. It has nothing to do with table indexes.

For example, a column specified as ENUM('Mercury', 'Venus', 'Earth') can have any of the values shown here. The index of each value is also shown

Value | index
---|---
NULL | NULL
'' | 0
'Mercury' | 1
'Venus' | 2
'Earth' | 3

