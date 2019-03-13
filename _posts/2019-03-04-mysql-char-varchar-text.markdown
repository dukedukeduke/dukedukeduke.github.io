---
layout: post
title:  "MySQL中char varchar和text的区别"
date:   2019-03-04 18:27:02 +0800
comments: true
tags:
- mysql
- char
- varchar
- text
---

### 参考
https://blog.csdn.net/weixin_41674292/article/details/80884508

https://dev.mysql.com/doc/refman/8.0/en/char.html

### 区别
#### CHAR
存储定长数据方便(0~255)，存储效率极高，必须在括号里定义长度，可以有默认值。
比如char（10）,即不论存储的数据长度是否达到了10字节，都要占去10个字节空间（自动用空格填充），且在检索的时候会自动将空格隐藏掉，
所以检索出来的数据记得用trim子类的函数过滤空格.
The length of a CHAR column is fixed to the length that you declare when you create the table. 
The length can be any value from 0 to 255.

#### varchar
存储变长数据，但效率没有char高，必须在括号里定义长度，可以有默认值。保存数据的时候，不进行空格填充。
如果数据存在空格时，当值保存和检索时尾部的空格仍会保留。另外，varchar类型的实际字节长度是它的值的实际长度+1，这一个字节用于保存实际使用了多大的长度。
varchar(n), n表示n个字符，无论汉字和英文，MySql都能存入 n 个字符，仅实际字节长度有所区别。
The max length of a varchar is subject to the max row size in MySQL, which is 64KB (not counting BLOBs):
VARCHAR(65535) However, note that the limit is lower if you use a multi-byte character set:
VARCHAR(21844) CHARACTER SET utf8;Please stop using CHARACTER SET utf8 in examples. It should be CHARACTER SET utf8mb4 
(if you want allUnicode text to be stored properly

#### text
存储可变长度的非Unicode数据，最大长度为2^31-1个字符。text不能有默认值，存储或检索过程中，不存在大小写转换，后面如果指定长度，不会报错误，
但是这个长度是不起作用的，意思就是你插入数据的时候，超过你指定的长度还是可以正常插入

#### 区别
数据的检索效率是：char > varchar > text

### 存储举例
Value | CHAR(4) | STORAGE REQUIRED | VARCHAR(4) | STORAGE REQUIRED
---|---|---|---|---
'' |'    ' |4 bytes |'' |1 byte
'ab' |'ab  ' |4 bytes |'ab' |3 bytes
'abcd' |'abcd' |4 bytes |'abcd' |5 bytes
'abcdefgh' |'abcd' |4 bytes |'abcd' |5 bytes
