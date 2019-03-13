---
layout: post
title:  "查看MySQL的版本"
date:   2019-03-02 18:27:02 +0800
comments: true
tags:
- mysql
---

1. mysql -V
没有连接到MySQL服务器，查看MySQL的版本

```
develop@server:~$ mysql -V
mysql  Ver 14.14 Distrib 5.6.30-76.3, for debian-linux-gnu (x86_64) using readline 6.3
```

2. mysql -v
这个命令可以查看到更为详细的信息，它会用账号 ODBC，连接上MySQL服务器，默认连接到localhost上的3306端口

```
develop@server:~$ mysql -v
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 158276
Server version: 5.6.30-76.3 Percona Server (GPL), Release 76.3, Revision 3850db5

Copyright (c) 2009-2016 Percona LLC and/or its affiliates
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Reading history-file /home/develop/.mysql_history
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
