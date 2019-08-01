---
title: mysql 重连处理
categories: mysql
date: 2019-08-01 15:57:14
tags: mysql, database
---

mysql 的连接有的时候会发生被动断开的情况：

* mysql 客户端与 mysql 服务器之间的网络出现问题
* mysql 服务器重启
* mysql 连接长时间没有操作，被 mysql 服务器断开。

在服务端程序中，我们希望 mysql 的连接能尽量保持不断开，如果断开就需要重连。
<!--more-->

## 保持连接

mysql 服务器会断开长时间没有操作的连接，这个时间长短取决于配置中的 wait_timeout 和 interactive_timeout 字段，默认是 8 小时。
mysql 客户端可以每隔一段时间调用一次 [mysql_ping][1] 以保持连接。

## 重连

一种方式是当发现连接断开时，销毁该连接，重新创建。使用这种方式的好处是知道连接什么时候被断开，这样就可以在重连后恢复某些状态，比如表锁、用户变量、字符集等等。特别是当我们使用 [Prepared Statements][2] 来操作数据库的时候，重连后需要重新执行 mysql_stmt_prepare 。

如果我们不需要连接保持这些状态，那就可以使用 mysql 本身实现的[重连机制][3]：

    MYSQL *mysql = ::mysql_init(nullptr); 
    ::mysql_real_connect(...);
    char reconnect = 1;
    ::mysql_options(mysql, MYSQL_OPT_RECONNECT, (char *)&reconnect));

注意：跟其它选项不同，设置 MYSQL_OPT_RECONNECT 的代码是放在 mysql_real_connect 后面的。这是因为在 mysql 5.0.19 之前，mysql_real_connect 会重置该选项（mysql->reconnect = 0 ）。

查看源码会发现，重连实际上是在 `cli_advanced_command` 中触发的。 
mysql_ping 和 mysql_real_query 最终都会调用到该函数，所以开启重连机制后就不需要其它额外操作了。

摘抄自 mysql 源码中的 mysql_init 实现：
> By default we don't reconnect because it could silently corrupt data (after
reconnection you potentially lose table locks, user variables, session
variables (transactions but they are specifically dealt with in
mysql_reconnect()).
This is a change: < 5.0.3 mysql->reconnect was set to 1 by default.
How this change impacts existing apps:
\- existing apps which relyed on the default will see a behaviour change;
they will have to set reconnect=1 after mysql_real_connect().
\- existing apps which explicitely asked for reconnection (the only way they
could do it was by setting mysql.reconnect to 1 after mysql_real_connect())
will not see a behaviour change.
\- existing apps which explicitely asked for no reconnection
(mysql.reconnect=0) will not see a behaviour change.


  [1]: http://dev.mysql.com/doc/refman/5.6/en/mysql-ping.html
  [2]: http://dev.mysql.com/doc/refman/5.6/en/statement-caching.html
  [3]: http://dev.mysql.com/doc/refman/5.6/en/auto-reconnect.html
