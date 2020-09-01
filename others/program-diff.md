---
title: 编程语言的差别速记
categories: Program
date: 2020-08-19 10:58:44
tags: [c++, lua]
---

编程语言有很多，总是学了忘，忘了又再学，但是实际上很多概念是共通的，只是细节上的差别。本文就是打算标注出这些细节，方便记忆。
<!-- more -->

***

## 基本类型

* **Lua :**  
  `nil boolean number string function userdata thread table`

* **Cpp :**  
  `bool char short int float double void`

***

## 条件表达式的默认值

* **Lua :**  
  `nil` 和 `false` 为假，其他都为真。

* **Cpp :**  
  0 值 (`int`、`float`、unscoped `enum`) 和 空指针 为假， 其他为真。

***

## 注释

* **Lua :**
  
      local a = 0;      -- 单行注释
      local b = 0;      --[[
                            多行注释
                            。。。
                        ]]

* **Cpp :**  
  
      int a = 0;        // 单行注释
      int b = 0;        /*
                             多行注释
                             。。。
                        */

***

## 操作符

* **Lua :**
  * **/** : 浮点除法，总是把操作数转成浮点数，结果也总是浮点数
  * **//** : 向下取整除法
  * **^** : 乘方
  * **~** : 按位取反，按位异或
  * **~=** : 不等于
  * **and or not** : 逻辑与或非，返回其中一个操作数，短路
  * **..** : 字符串连接
  * **#** : 取长度

* **Cpp :**  
  * **^** : 按位异或
  * **~** : 按位取反
  * **!=** : 不等于
  * **&& || !** : 逻辑与或非，返回bool值，短路

***

## 问号表达式

* **Lua :**  
  `(exp1 and exp2) or exp3`

* **Cpp :**  
  `exp1 ? exp2 : exp3`

***

## for 循环

* **Lua :**  
  `for i = init, limit, step do block end`  
  `for var_1, ..., var_n in explist do block end`

* **Cpp :**  
  `for (int i=0; i<10; ++i) { }`  
  `for (int i : vector) { }`  

***

## 函数定义

* **Lua :**
  
      function cla:f() body end         -- self.xxx 
      function f() body end
      local f = function() body end

      function f(name)
          name = name or "Tom"          -- 参数默认值
      end

* **Cpp :**  

      void f(){}
      void f(const char* name = "Tom") {}    -- 参数默认值

***

## 