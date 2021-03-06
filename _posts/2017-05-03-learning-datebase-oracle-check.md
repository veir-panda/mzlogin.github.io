---
layout: post
title: Oracle数据库中的约束
categories: datebase
description: Oracle数据库中的约束
keywords: Oracle, check, datebase, 数据库约束
---

# Oracle数据库中的约束
-------------
>注：本文参考自[文章](http://www.cnblogs.com/Ronger/archive/2011/10/10/2205515.html)

上周应聘一家公司的实习生，笔试时遇到一个题：

 - 列出Oracle数据库中有哪些约束，并说明该约束的作用

很遗憾，一不小心就把上学期学得挺好的Oracle忘完了，突然想起来，查阅资料，同时在此记录一下。

----------

为了维护数据的完整性，Oracle提供了5种约束:

 1. NOT NULL  (非空）：约束该列一定要输入值。

 1. UNIQUE Key  （唯一） ：当定义了唯一约束后，该列值是不能重复的，但是可以为null。
 
 2. PRIMARY KEY   （主键） ：用来唯一标示表中的一个列，一个表中的主键约束只能有一个。当定义主键约束后，该列不但不能重复而且不能为NULL。
 
 3. FOREIGN KEY  （外键） ：用于定义主表和从表之间的关系，外键约束要定义在从表上，主表则必须具有主键约束或是unique约束，当定义外键约束后，要求从表的外键列数据必须在主表的主键列存在或是为NULL。
 
 4. CHECK ：用于强制行数据必须满足的条件，假定在sal列上定义了check约束，并要求sal列值在1000～2000之间，如果不在1000～2000之间就会提示出错：check(sal between 1000 and 2000)。