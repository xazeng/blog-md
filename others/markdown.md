---
title: markdown 基础
categories: 未分类
date: 2019-08-01 15:58:44
tags: software, markdown
---

Markdown是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。  
本文主要记录一些基本用法，供参考和记忆。
<!--more-->

***
## 兼容 HTML

不需要任何标注就可以使用 HTML 标签。

区块标签（比如：`<div>`、`<table>`、`<p>`、`<pre>` 等）必须在前后加上空行，且要求开始和结束标签不能用 tab 或空格来缩进。  
区段标签（比如：`<span>`、`<a>`、`<img>` 等）则不受限制，可以在 Markdown 语法里任意使用。  
值得注意的是 Markdown 语法在区块标签内是无效的，在区段标签内有效。

尽量避免使用 style 标签，因为有些网站（比如 github）出于某种考虑会禁用这个标签。  
实际上应该尽量避免直接使用 html 标签，因为使用 Markdown 的目的就在于简化格式语法，而不是让它更复杂。

***
## 特殊字符自动转换

某些特殊字符（比如：`<`、`&`）在 HTML 中必须以实体形式存在，这种实体形式在 Markdown 的语法中也是有效的。  
当然 Markdown 允许直接使用这些特殊字符。

***
## 段落和换行

用前后至少一个空行来区分段落，段落内允许换行（两个空格加回车）。  
普通段落不要用空格或者 tab 来首行缩进，因为空格和 tab 会被当成 Markdown 语法处理。

***
## 标题

    # 一级标题
    ## 二级标题
    ### 三级标题
    #### 四级标题
    ##### 五级标题
    ###### 六级标题

***
## 引言区块

在每行前面加上 `>` 就能产生一个区块引言。  
Markdown 允许只在整个段落的第一行最前面加上 `>` 。  
引言的区块内可以使用其它 Markdown 语法，包括引言语法。

    > ## This is a header.
    > 
    > 1.   This is the first list item.
    > 2.   This is the second list item.
    > 
    > Here's some example code:
    > 
    >     return shell_exec("echo $input | $markdown_script");

***
## 列表

* #### 有序列表
    序号不影响结果的数字的顺序。

        1. 文本1
        2. 文本2
        3. 文本3

* #### 无序列表
    星号、加号和减号都可以作为无序列表标记

        * 文本
        * 文本
        * 文本

列表项目可以包含多个段落。  
列表项目下的段落、引言、和代码区块都必须先缩排 4 个空格或者一个 tab。

***
## 代码区块

一个空行，加上每行缩进 4 个空格或者一个 tab ，就构成一个代码区块。
    
        do
        {
            for (int i=0; i<10; i++)
            {
                printf("%02d\n", i);
            }
        }while (false);

***
## 分隔线
    
    ***
    * * *
    *****
    -----
    ---------------------------------------

***
## 链接

* #### 行内
    
        This is [an example](http://example.com/ "Title") inline link. // 绝对路径
        See my [About](/about/) page for details. // 相对路径
        Example link: <http://www.baidu.com>.  //

* #### 参考

        This is [an example][id] reference-style link.
        
        // 文档任意处，建议统一放在文档最后
        [id]: http://example.com/  "Optional Title Here"

    这里的 id 可以是数字，也可以是字母。

***
## 强调

使用一个 `*` 或者 `_` 包围的文本会被转成用 `<em>` 标签包围。  
使用两个 `*` 或者 `_` 包围的文本会被转成用 `<strong>` 标签包围。

    *em*
    **strong**

***
## 程序代码

行内标记程序代码，可以用反引号 `` ` `` 包围。
    
    Use the `printf()` function.
    ``There is a literal backtick (`) here.`` // 代码内有反引号
    A single backtick in a code span: `` ` ``. // 单个反引号

***
## 图片

    ![Alt text](/path/to/img.jpg "optional title")
    
    ![Alt text][id]
    [id]: url/to/image  "Optional title attribute"

***
## 相关链接

[Markdown: Syntax][1]  
[Markdown 語法說明][2]  
[Markdown 语法说明][3]  
[知乎：怎样引导新手使用 Markdown][4]  

  [1]: http://daringfireball.net/projects/markdown/syntax
  [2]: http://markdown.tw/
  [3]: http://www.ituring.com.cn/article/504
  [4]: http://www.zhihu.com/question/20409634

