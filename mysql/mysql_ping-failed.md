---
title: mysql_ping 失敗
categories: mysql
date: 2020-03-16 15:19:05
tags: [mysql, database]
---

台湾同事在接入【登入登出日志】后发现 mysql 服务器的连接数一直往上升，进一步分析确定：

*每调用一次【登入登出日志】的存储过程，mysql 服务器的连接数就多一个。*
<!--more-->

查看台湾服务器相关代码后发现，他们并没有处理*多结果返回*的情况，而写入日志的存储过程恰好返回了多个结果。

mysql 的文档对这个情况有非常具体的描述：
>Executing a multiple-statement string can produce multiple result sets or row-count indicators. Processing these results involves a different approach than for the single-statement case: After handling the result from the first statement, it is necessary to check whether more results exist and process them in turn if so. To support multiple-result processing, the C API includes the mysql_more_results() and mysql_next_result() functions. These functions are used at the end of a loop that iterates as long as more results are available. <u>Failure to process the result this way may result in a dropped connection to the server.</u>  
详见：<https://docs.oracle.com/cd/E17952_01/mysql-5.6-en/c-api-multiple-queries.html>

实际上 mysql 建议所有可能调用存储过程的连接一定要记得处理多结果的情况。
>If your program uses CALL statements to execute stored procedures, the CLIENT_MULTI_RESULTS flag <u>must be</u> enabled. This is because each CALL returns a result to indicate the call status, in addition to any result sets that might be returned by statements executed within the procedure. Because CALL can return multiple results, process them using a loop that calls mysql_next_result() to determine whether there are more results.  
详见：<https://docs.oracle.com/cd/E17952_01/mysql-5.6-en/mysql-real-connect.html>

2020.8.1 记：vt 服务器中的 center server 出现公会数据在周六0点后无法继续写入的问题。经查发现是其中一个业务使用 ostringstream.clean 导致多条语句出现在一次查询中，以致后续查询都没有生效也没有报错。虽然内网没有重现，但更新这个bug后外网也不再出现类似故障，应该可以基本认定是没有处理 *多结果返回* 导致。
