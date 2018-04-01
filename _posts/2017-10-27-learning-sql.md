# 数据库之SQL提高

最近在工作中写sql语句时，遇到了以前在学校学习sql过程中没见过的知识，在此记录。

## `with as` 用法

以前写sql，在遇到查询条件复杂的情况，更多用到的是嵌套查询，尽管也能完成查询任务，但相比我最近新学到的`with as`就要麻烦很多了。

先来看看两种写法的对比：
```sql
/*嵌套的写法*/
select *, enddates-startdates as day
from (
	SELECT date(to_char(ty.a,'9999') || '-03-01')-1  AS enddates, date(to_char(ty.a,'9999') || '-02-01') AS startdates, ty.a as year
FROM (
        SELECT s.a
        FROM generate_series(1980, 2010) as s(a)
     ) as ty
) as adate ;

/*with-as的写法*/
with ty
as
(
    SELECT s.a
    FROM generate_series(1980, 2010) as s(a)
),
adate
as
(
    SELECT date(to_char(ty.a,'9999') || '-03-01')-1  AS enddates, date(to_char(ty.a,'9999') || '-02-01') AS startdates, ty.a as year
    FROM ty
)
select *, enddates-startdates as day
from adate;
```

从上面两种写法一眼就能看出`with as`写法相比嵌套写法的优点：
1. 比嵌套写法更简单（更符合我个人的思维模式），相比来说，如果嵌套的层数过多，那就太复杂了；简单还体现在，如果底层的结果被用在了不止一个地方，那还会造成代码冗余！
2. 方便阅读，代码通常还得给其他人看，而嵌套的写法最大的缺点就是不易阅读和理解
3. 便于维护，这其实是方便阅读带来的好处，但这一点也是非常重要的。
4. `with as`写法相比嵌套写法的查询效率要高，上面的例子中`with as`就比嵌套快`0.001s`（用的PostgreSQL数据库查询，嵌套语句耗时0.003s，`with as`语句0.002s）！！
5. 以上4点是我硬凑起来的……无他，我就觉得`with as`写法比嵌套写法好太多了~！只恨到现在才知道……

### 写法

- `with as`写法的格式：
```sql
with tmp_table_name1
as
(
    /*第一个查询---实际使用时不要在一句sql的中间写注释*/
    ……
),
tmp_table_name2
as
(
     /*第二个查询---实际使用时不要在一句sql的中间写注释*/
    ……
),
tmp_table_name3
as
(
     /*第三个查询---实际使用时不要在一句sql的中间写注释*/
    ……
)
select *
from tmp_table_name3;
```

写法实际上在前面的例子中已经体现了，这里只提醒几个注意点：
- 最后一句（也就是嵌套写法最外面的查询）的前面不要有"`, tmp_table_name as`",尤其是那个**逗号**，千瓦不能有，我才不会告诉你我跳下去这个坑几次……
- 与嵌套写法类似，`with as`中的最后一句除了可以是查询外，还可以是插入（`insert`）、删除（`delete`）、修改（`update`）。

----
### （10.30更新）

今天刚好又用到了`with as`与`insert`结合的方式，顺便附在这里：

```sql
-- with as 与insert结合使用
with
sdata as
(
		select stationid from china_stations limit(723)
),
adata as
(
		select * from sixtysum, sdata where stationid = station
)
insert into sixtysum_723
(
		station, time, rain
)
select station, time, rain from adata;
```
----
## `row_number() over()`用法

`row_number() over()`实际上是两个函数`row_number()`和`over()`组合起来用的，它们又叫分析函数（开窗函数？）,下面这篇文章解释的很好，我就偷个懒了，嘿嘿……
> 引用自[SQL 关于row_number() over()](http://www.cnblogs.com/shuangnet/archive/2013/04/12/3016898.html)
> 注：引用文章只做学习用，如有侵权，请留言，我会尽快删除。


先建表（dbo.PeopleInfo）：

```sql
CREATE TABLE [dbo].[PeopleInfo](
    [id] [int] IDENTITY(1,1) NOT NULL,
    [name] [nchar](10) COLLATE Chinese_PRC_CI_AS NULL,
    [Gender] [nchar](10) COLLATE Chinese_PRC_CI_AS NULL,
    [numb] [nchar](10) COLLATE Chinese_PRC_CI_AS NULL,
    [phone] [nchar](10) COLLATE Chinese_PRC_CI_AS NULL,
    [FenShu] [int] NULL
) ON [PRIMARY]
```
向表中插入数据：

```sql
insert into peopleinfo([name],Gender,numb,phone,fenshu) values ('李欢','男','3223','1365255',80)
insert into peopleinfo([name],Gender,numb,phone,fenshu) values ('李欢','男','322123','1',90)
insert into peopleinfo([name],Gender,numb,phone,fenshu) values ('李名','男','3213112352','13152',56)
insert into peopleinfo([name],Gender,numb,phone,fenshu) values ('李名','女','32132312','13342563',60)
insert into peopleinfo([name],Gender,numb,phone,fenshu) values ('王华','女','3223','1365255',80)
```
查询出所有插入的数据：

```sql
select * from  dbo.PeopleInfo
```
结果如图：
![](https://www.veir.site/upload/2017/10/64aife6ta2i1ooub8n9crqtch9.png)


**例子一**：只用order by 不用 partition by 的sql语句如下：

```sql
--不用partition by
select [name],gender,fenshu, row_number() over(order by fenshu desc) as num from dbo.PeopleInfo
```
结果如图：
![](https://www.veir.site/upload/2017/10/r74grsfmjugbto2io6m1v70nhv.png)


**例子二**：用order by 也用 partition by 的sql语句如下：

```sql 
select [name],gender,fenshu, row_number() over(partition by Gender order by fenshu desc) as num from dbo.PeopleInfo
```
结果如图：
![](https://www.veir.site/upload/2017/10/r40bum0gr8g51q8c6a3v1qhmdv.png)


比较例子一和例子二的结果图很容易就明白partition by的用处了，以例子二为例就是先用partition by把性别`Gender`分成两个区一个男一个女，然后再用order by 把每个区里的分数`fenshu`从大到小排序。

## 函数的使用：

第一节的例子里我用到了一个函数`generate_series()`，这个函数使用来生成一段连续的数字序列的，在实际工作中很有用处。

还有一个函数:`regexp_split_to_table()`,这个函数用来将一个字符串按给定的字符分割为一个查询集合，与java中的`String.split()`函数功能相似。

例子：
```sql
select a
from regexp_split_to_table('2005, 2006, 2007', E'(, *)+') as s(a);
```
结果：
![](https://www.veir.site/upload/2017/10/q438icda8oirdp3unqt9oh91rv.jpg)

上面两个函数在`PostgreSQL`数据库中有，`MySQL`中就没有。
所以，以后多注意一些数据库自带的函数，往往能极大提升工作效率

## 其他

在查询中不要过多的依赖"`in()`",如果`in()`中的参数过多，会导致查询效率非常低。

## 总结

要学习的还有很多啊……