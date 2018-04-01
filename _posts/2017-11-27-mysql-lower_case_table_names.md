---
layout: post
title: MySQL表名大小写在Windows和Linux上的不同
categories: datebase
description: MySQL表名大小写在Windows和Linux上的不同
keywords: mysql, 表名, 大小写
---

# MySQL表名大小写在Windows和Linux上的不同

今天将一个项目放到公司服务器上，在调试的时候突然发现系统登录不上去了，控制台报错代码“1146”，说表找不到，但是在我本地运行又是没问题的。由于数据非常大，折腾来折腾去搞了一下午。最后发现其实是MySQL本身的坑造成的。

## 原因

MySQL在Windows上的表名是不区分大小写的，而在Linux上严格区分大小写。

> 偏偏公司的初始数据库表名全是小写的，而sql查询里面又大写的，造成一直找不到要查询的表。

## 解决办法

修改Linux上的MySQl配置文件，使它不区分大小写

Linux版本：Ubuntu 14.04
MySQl版本：5.7.20

1. 修改`my.cnf`,命令:`vim /etc/mysql/my.cnf`
2. 在`[mysqld]`后面添加`lower_case_table_names=1 `
3. 保存后，重启mysql即可

## MySql在Linux下的大小写规则问题

1. 数据库名与表名是严格区分大小写的 
2. 表的别名是严格区分大小写的 
3. 列名与列的别名在所有的情况下均是忽略大小写的 
4. 变量名也是严格区分大小写的 