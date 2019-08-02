---
title: mysql 字符集处理
categories: mysql
date: 2019-08-01 15:19:05
tags: [mysql, database]
---

在 mysql 中，字符集问题其实并不显眼，很多时候都不需要纠结，但是回忆了下这些年，字符集还是会时不时出来捣乱，所以干脆总结下，免得总是过一段时间又去翻官方文档。
<!--more-->

------
## 服务端字符集

主要用于数据存储。   
配置，数据库，表，列都可以分别指定字符集，不指定时默认使用上一级设置。  
服务端字符集的设定没有什么特殊要求，只要能保证客户端所用字符集小于服务端字符集，不至于在转换过程中导致内容丢失就行。  

具体参考：<http://dev.mysql.com/doc/refman/5.7/en/charset-syntax.html>


------
## 客户端字符集

用于客户端程序和MySql服务器间的通讯。  
相关配置项有 `character_set_client` `character_set_connection` `character_set_result`。  
服务端会将客户端传来的数据从 `character_set_client` 编码转到 `character_set_connection` 编码，执行结果又会用 `character_set_result` 编码返回。

客户端可以自行设定字符集，主要方式有：

* `mysql_query(mysql, "SET NAMES 'charset_name' [COLLATE 'collation_name']")`  
    这个调用等同于：  
    
        SET character_set_client = charset_name;
        SET character_set_results = charset_name;
        SET character_set_connection = charset_name;

    值得注意的是这个操作 *不会改变* `mysql_real_escape_string` 所使用的字符集。

* `mysql_options(mysql, MYSQL_SET_CHARSET_NAME, 'charset_name')`  
    实际上这个调用就是让 MySql 服务器执行 `SET NAMES` 语句。  
    不同的是，这个调用必须在 `mysql_real_connect` 之前执行，而且直接决定了 `mysql_real_escape_string` 所使用的字符集。

* `mysql_set_character_set(mysql, 'charset_name')`
    这个调用在 `mysql_real_connect` 之后执行，并且也能够改变 `mysql_real_escape_string` 所用字符集。 

具体参考：<http://dev.mysql.com/doc/refman/5.7/en/charset-connection.html>


------
## 转译

*查询前* 需要对字符串和二进制数据进行转译。  
主要的作用是替换一些特殊字符，比如 `\0` `'` `"` `\n` `\ ` 等以达到防止 sql 注入的作用。  
之所以需要编码信息，是为了防止错误识别这些要被替换的特殊字符，比如 `0xbf5c` 在 GBK 编码中就是是个有效字符。(`0x5c = \ `)

*查询时* MySql 会检查字符串的内容和指定字符集是否匹配，不匹配的情况下查询会出错。  
（这里对这种匹配规则不是很明确，不一致但是属于子集超集关系的字符集也可能通过，只是可能因为兼容程度不同导致乱码）  
MySql 并不对二进制数据进行字符集检查，所以二进制数据不存在这种错误情况。

*由上述可知，客户端连接应该指定和参与查询的字符串相同的字符集。*

相关函数说明：

> [`unsigned long mysql_real_escape_string(MYSQL *mysql, char *to, const char *from, unsigned long length)`][1]  
>
> * 参数 to 的长度最少为：length*2+1。因为最坏的情况下每个字节都需要转义，再加上最后的 \0 。
> * 返回转义后字符串的长度，不包括 \0 。
>
> 示例：  

```cpp
>     MYSQL *handler = ::mysql_init(nullptr);
>     ::mysql_options(handler, MYSQL_SET_CHARSET_NAME, "utf8"); // 指定编码
>     ::mysql_real_connect(...);
>     const char *from = "待转义字符串";
>     int length = strlen(from);
>     char *to = new char[length*2+1];
>     ::mysql_real_escape_string(handler, to, from, length); // 转义
```



[1]: http://dev.mysql.com/doc/refman/5.6/en/mysql-real-escape-string.html




