问题
--

最近，在用SSH框架完成一个实践项目时，碰到了一个莫名其妙的Bug困扰了我好久，最后终于解决，记录如下。

> **问题**：同学在测试系统的时候突然发现，数据库保存的账户本来应该是admin，结果该同学用Admin账户居然登录成功了……

……EXM？？？这样也行？好吧，我还是查找这个Bug发生的原因吧。然后就是各种排查程序的过程，找来找去也没发现什么问题。终于想到，不用hql，自己写sql语句在数据库里面直接查询试试，结果果然发现了问题所在：

```
select * from user where username = 'admin' and password = 'admin';
select * from user where username = 'Admin' and password = 'admin';
```
用上面的两条sql语句分表查询，出来的结果居然是一样的！……！！去搜索引擎搜索关键词：MySQL  查询   大小写，果然找到问题了！MySQL查询是不区分大小写的！这可真的是惊呆我了，虽然知道一般情况下，关键字是不区分大小写的，但是没想到连要查询的参数都是不区分大小写的！！再尝试下面的sql语句，果然还是一样的结果。

```
select * from user where username = 'ADMIN' and password = 'admin';
```

解决方案
----

网上搜索到一篇相关的文章，写的挺好的，这里直接贴上该文章解释吧：

1. Mysql默认的字符检索策略：utf8_general_ci，表示不区分大小写；utf8_general_cs表示区分大小写，utf8_bin表示二进制比较，同样也区分大小写 。（注意：在Mysql5.6.10版本中，不支持utf8_genral_cs！！！！）
	
	创建表时，直接设置表的collate属性为utf8_general_cs或者utf8_bin；如果已经创建表，则直接修改字段的Collation属性为utf8_general_cs或者utf8_bin。

	```
	-- 创建表：
	CREATE TABLE testt(
	id INT PRIMARY KEY,
	name VARCHAR(32) NOT NULL
	) ENGINE = INNODB COLLATE =utf8_bin;

	-- 修改表结构的Collation属性
	ALTER TABLE TABLENAME MODIFY COLUMN COLUMNNAME VARCHAR(50) BINARY CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL;
	```
2. 直接修改sql语句，在要查询的字段前面加上binary关键字即可。
	

	```
	-- 在每一个条件前加上binary关键字
	select * from user where binary username = 'admin' and binary password = 'admin';

	-- 将参数以binary('')包围
	select * from user where username like binary('admin') and password like binary('admin');
 
	```
		
> **注意**：在我的这次项目中用的是hibernate框架，使用的并不是sql，而是hql语句，使用 `from User where binary username = ? and binary password = ?;`结果报错，再试 `from User where username like binary(?) and password like binary(?);`才没有报错。原因暂时不知。


----
> 部分引用自文章：http://www.imooc.com/article/14190