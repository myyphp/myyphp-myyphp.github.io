---
layout: post
title: Mysql索引文件本身的数据结构
tags:  Mysql
categories: php
---

#Mysql索引文件本身的数据结构

##1、问题：索引本身是一种数据，也是需要一定的格式来存储到磁盘上的
索引文件如何存储呢？
不同的索引文件，存储的结构不一样，对于 MyiSam 和 Innodb 的索引文件，分别如下：
1. `MyiSam` 存储引擎的索引结构是：B-tree（balance tree）
2. `Innodb` 存储引擎的索引结构是：聚簇索引

##2、什么是B-tree索引
![](http://myyphp.github.io/public/img/posts/mysql_index1.png)

##3、什么是聚簇索引

![](http://myyphp.github.io/public/img/posts/mysql_index2.png)

##4、几点说明
- 可以看出 MyiSam存储引擎 的结构在 维护方面 更具优势，因为 Innodb的存储引擎的结构里的 索引值下面包含了该行记录，索引开销很大；但是在查询方面却有着 无可比拟的性能，不需要做回行处理（读取磁盘文件）

- 如果项目不要使用事务可以有限考虑 MyiSam存储引擎来处理，具有更高效的维护更本

- 在存在主键ID时，在批量插入数据的时候，MyiSam存储引擎，索引列不会进行排序，直接插入；但Innodb存储引擎会在排序后再插入，故会影响性能

##5、测试
- 创建 `myisam` 的表
{% highlight php linenos %}
 
	CREATE TABLE my_isam (
     	 id int(11) NOT NULL DEFAULT '0',
     	 name varchar(20) DEFAULT NULL,
     	 address varchar(30) DEFAULT NULL,
      	 PRIMARY KEY (id)
	) ENGINE=MyISAM DEFAULT CHARSET=utf8;
{% endhighlight %}
- 创建 innodb 的表
{% highlight php linenos %}
	CREATE TABLE `in_nodb` (
          id int(11) NOT NULL DEFAULT '0',
          name varchar(20) DEFAULT NULL,
          address varchar(30) DEFAULT NULL,
          PRIMARY KEY (id)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;
{% endhighlight %}

- 添加数据测试

`myisam`存储引擎：
{% highlight php linenos %}
    insert into  my_isam(id,name,address)  values(8, 'jkeyll', 'shenzhen');
    insert into  my_isam(id,name,address)  values(10, 'keyll', 'shenzhen'); 
    insert into  my_isam(id,name,address) values(3,'tony','hangzhou');
    insert into  my_isam(id,name,address) values(2,'andy','zhenzhou');
{% endhighlight %}

`innodb`存储引擎：
{% highlight php linenos %}
    insert into  in_nodb(id,name,address)  values(8, 'jkeyll', 'shenzhen');
    insert into  in_nodb(id,name,address)  values(10, 'keyll', 'shenzhen'); 
    insert into  in_nodb(id,name,address) values(3,'tony','hangzhou');
    insert into  in_nodb(id,name,address) values(2,'andy','zhenzhou');
{% endhighlight %}

**对比：通过测试数据可以看出，在`myisam`存储引擎在批量插入数据的时候，不会做排序处理，而`innodb`则会先排序在处理**