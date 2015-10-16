---
layout: post
title: Mysql优化小技巧
tags:  Mysql
categories: php
---

#Mysql优化小技巧

##索引碎片与维护

> 在长期的数据更新过程中, 索引文件和数据文件,都将产生空洞，形成碎片我们可以通过一个nop操作(不产生对数据实质影响的操作), 来修改表。比如: 表的引擎为InnoDB , 可以 `alter table tableName engine InnoDB`也可以` optimize table tableName `,也可以修复。

**注意**： 修复表的数据及索引碎片，就会把所有的数据文件重新整理一遍，对于这个过程，如果表的行数比较大，也是非常耗费资源的操作。所以，不能频繁的修复。

**建议**：如果表的Update操作很频率，可以按 周/月 来修复，如果不频繁，可以更长的周期来做修复

##重复索引和冗余索引的思考

> 重复索引:：是指 在同1个列(如name)， 或者 顺序相同的几个列(name,age)， 建立了多个索引。称为重复索引，重复索引没有任何帮助，只会增大索引文件，拖慢更新速度，必须去除。冗余索引：冗余索引是指2个索引所覆盖的列有重叠， 称为冗余索引，比如 x,m列， 加索引  index x(x)，  index xm(x, m)。x、xm索引， 两者的x列重叠了， 这种情况,称为冗余索引，甚至可以把 index mx(m, x) 索引也建立， mx、xm  也不是重复的，列的顺序不一样。

##in型子查询的陷阱

在ecshop商品表中，查询6号栏目的商品（6号是一个大栏目）
最直观的：     
`mysql> select goods_id,cat_id,goods_name from goods where cat_id in (select cat_id from ecs_category where parent_id=6);`

误区： 给我们的感觉是， 先查到内层的6号栏目的子栏目，如7,8,9,11然后外层，cat_id in (7,8,,9,11)
实际上： 如下图, goods表全扫描， 并逐行与category表对照，看parent_id=6是否成立
![](http://myyphp.github.io/public/img/posts/mysql_optimize1.png)

![](http://myyphp.github.io/public/img/posts/mysql_optimize2.png)

**原因： mysql的查询优化器，针对In型做优化，被改成了exists的执行效果，当goods表越大时， 查询速度越慢**

改进： 用连接查询来代替子查询
   ` explain select goods_id,g.cat_id,g.goods_name from  goods as g inner join (select cat_id from ecs_category where parent_id=6) as t using(cat_id) \G`


1. 内层 select cat_id from ecs_category where parent_id=6 ; 用到Parent_id索引，返回4行，形成结果,设为 t

2. 第2次查询， t 和 goods 通过 cat_id 相连， 因为cat_id在 goods表中有索引，所以相当于用7,8,911，快速匹配上goods的行。

##exists子查询
题： 查询有商品的栏目.按上面的理解，我们用join来操作，如下：
    `mysql> select c.cat_id, cat_name from ecs_category as c inner join  goods as  g on c.cat_id=g.cat_id group by cat_name;`



1. 优化1： 在group时， 用带有索引的列来group， 速度会稍快一些,另外，用int型 比 char型分组，也要快一些

2. 优化2： 在group时， 我们假设只取了A表的内容，group by 的列，尽量用A表的列，会比B表的列要快

3. 优化3： 从语义上去优化

   `select cat_id,cat_name from ecs_category where exists(select *from  goods where  goods.cat_id=ecs_category.cat_id) `

##rom子查询

注意：内层from语句查到的临时表，是没有索引的。所以： from的返回内容要尽量少max|min优化技巧

有如下的地区表：
![](http://myyphp.github.io/public/img/posts/mysql_optimize3.png)

我们查min(id)，id是主键，查min(id)非常快,但是，pid上没有索引， 现在要求查询3113地区的min(id)
`select min(id) from area where pid=69;`

试想 id是有顺序的，(默认索引是升续排列)， 因此,如果我们沿着id的索引方向走，那么 第1个 pid=69的索引结点，他的id就正好是最小的id。
`select  id  from area use index(primary) where pid=69 limit 1;`

##count() 优化

误区： myisam的count()非常快

答: 是比较快，但仅限于查询表的”所有行”比较快，因为Myisam对行数进行了存储。一旦有条件的查询, 速度就不再快了。尤其是where条件的列上没有索引。

假如，id<100的商家都是我们内部测试的，我们想查查真实的商家有多少?

`select count(*) from tableName where id>=100;`

小技巧：

`select count(*) from tableName ; 快`
`select count(*) from tableName where id < 100; 快`
`select count(*) from tableName - select count(*) from tableName where id<100; 快`
`select (select count(*) from tableName ) - (select count(*) from tableName where id<100)`

##group小技巧

注意：分组用于统计，而不用于筛选数据.

比如： 统计平均分，最高分，适合； 但用于筛选重复数据，则不适合.。以及用索引来避免临时表和文件排序以A,B表连接为例 ，主要查询A表的列， 那么 `group by` ,`order by` 的列尽量相同，而且列应该显示声明为A的列

##union优化
注意： `union all` 不过滤 效率提高，如非必须，请用`union all` ，因为 `union`去重的代价非常高，放在PHP程序里去重更合适
