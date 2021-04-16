---
title: "基于MySQL的rownum伪列功能实现"
date: 2021-03-16T23:21:17+08:00
draft: false
tags: ["Databasee"]
categories: ["coding 1024"]
featured_image: ""
description: ""
---

<br>

## rownum伪列功能

伪列的含义就是说：实际上该列并不存在，但是你可以直接像正常的列一样使用它。rownum伪列是Oracle用来标记返回数据行号的。Oracle中没有limit，通过在where条件中使用rownum伪列，可以达到跟MySQL中limit类似的效果。

使用时需要注意以下几点：

1. rownum的生成时机。跟rownum窗口函数类似，是在最后的order by之前生成。
2. where条件中的“rownum>正整数”将会导致select结果为空

rownum最常见的应用是topN，即前几名的场景，但是要区分以下两种sql，其效果是完全不一样的。

```sql
-- 对t表的前N行数据按成绩排序
select rownum, t.* from t where rownum < N order by grade;
-- 取前N名
select rownum, T.* from (select * from t order by grade) T where rownum < N;
```

## 架构与设计

### 架构

![basic_framework](https://cdn.jsdelivr.net/gh/longf2021/myImage@main/mysql_base/basic_framework.jpg)  

如上图所示，这是我画的一个架构图，很简单的一个基于MySQL的分布式数据库的实现方案（maybe中间件？）。其中，gate是整个系统的入口，负责根据meta data，将sql转发到底下的MySQL上。

我们的任务就是在这个架构上实现类似Oracle中的rownum伪列的功能。

### 设计

总体思想：取到数据之后，在order by之前，为每行加上行号，并根据where条件中的rownum表达式，完成数据的过滤。

1. 需要拉取的数据太多，但实际上只需要输出少量数据

   - 使用流模式去拉取数据

     *暂未实现*

   - 将rownum改写成一个limit

     将rownum转换为limit，下推到每个MySQL上，汇聚数据以后，再在gate上对数据进行添加行号的操作。比如，`select rownum,t.* from t where rownum<10;` 改写为 `select t.* from t limit 10` 下推;

     具体而言，就是将rownum<=x order by xxx转换为(limit x) order by xxx，这样就是在每个mysql上先进行limit，然后对limit的结果进行排序。每个mysql的结果汇集到gate上后，再执行一遍limit，再进行order by排序。

     将rownum转换为limit可以分为以下这些场景：

     - rownum=x时，当x>1时不返回数据；当x=1返回第一行，可以转换为limit 1。 
     - rownum!=x 时，当x<1返回全部数据，当x>=1等价与rownum<x，可以转换为limit x-1。
     - rownum <= x 时，当x<=0时转为limit 0，返回空集；x>0时转换为limit x，返回前x行)
     - rownum < x时，当x<=1转换为limit 0，返回空集；x>1时转换为limit x-1，返回前x-1行)
     - rownum>x时，当x<1，返回全部数据， x>=1返回空集
     - rownum>=x时，当x<=1，返回全部数据，x>1返回空集

     以rownum=1为例：

     - 在没有order by的情况，oracle的文档描述是说结果是不确定的。我们转换为limit 1，是会下推的每个mysql上的，在每个mysql上取一行数据到gate上，然后在返回最先到达gate的那行数据，因此，我们的行为也是不确定的。
     - 在加了order by的情况下，Oracle的rownum是在order by之前进行，所以就有了where rownum<x order by xxx和 (order by xxx) where rownum<x 的区别，括号强行让rownum在order by之后进行。对于前者，直接转换为limit，就改变了语义（变成了先排序，在加rownum），所以需要将提升order by子句；对于后者，直接转化为limit即可。
     - 因为索引的关系，rownum<=x order by xxx在oracle中的结果是不确定的。我们转换为(limit x) order by xxx后，要做两遍limit，一遍在每个mysql上，另外一遍在gate上，结果也是不确定的，但不确定的原理是不一样的。

2. 对于不分shard的表来说，性能可以再优化

   可以直接对rownum进行改写，将改写的SQL发往底下的MySQL上。这样的好处是省去了gate对取回来的数据再次做加序号排序的工作。具体的改写方法可以参考附录一。

3. 复杂条件的处理

   - in/not in表达式

     主要对in的内容有要求：排除小于1的数，最小值不为1则输出空集，最小值为1时，则输出到连续的最大值；

     对于not in：小于1的不起任何作用，列表中包含1则输出空集；不包含1时则输出的最小值，比方是x(x>1)，则改为rownum < x，等价limit x-1

     如：

     ```sql
     rownum in(1,2,3) -- 返回1,2,3 
     rownum in(0,1,2,3,5,6,7) -- 返回1,2,3 
     rownum in(2,3,4,5,7,8,9) -- 返回空
     ```

   - [not] between ... and ..

     相当于`limit m,n` m也要求从小于1

     between min and max

     如果没有not，当between不合法时，即min>max时，返回空集；合法时，min>1时，返回空集，min<=1时相当与limit max。

     当有not，不合法时，返回全集；合法时，min<=1，返回空集，min>1时，相当与limit min-1

## 实现

1. 改写rownum的format函数

   需要完成从原始sql中去除所有rownum。对于select子句中的rownum，按照附录一的方法，改写成自定义变量；对于where条件中的rownum，将其合并成为一个limit。

   需要注意：

   - 输出列中可能包含多个rownum
   - rownum可能会涉及到复杂的表达式
   - rownum的优先级高于同级的order by

2. 对数据进行fillRownum的操作

   这主要是为了处理分布式场景。也就是，从底下MySQL中获取到数据以后，对数据加上伪列的过程

## 需要注意的地方

1. 不支持rownum伪列和limit混用

2. 不支持rownum四则运算，如rownum=rownum+1, (rownum+1)*10<100等，以及一些其他的复杂比较函数如like/not like等

3. 复杂条件支持测试不完善，可能会有坑

4. 不支持or条件包含rownum，如`select rownum,t.* from t where id > 7 or rownum < 3`

   按照我们的处理方法，转换为limit后，sql就会变为`select t.* from t where id > 8 limit 2;` 结果明显是不对的，也就是说or条件中的rownum不能随便转换为limit。可以看附录二的例子。

## 附录一

 在单个mysql中模拟oracle rownum一个的方法：

```sql
select @rn:=@rn+1 as rownum,t.* from (Original SQL) t,(select @rn:=0) r;
```

利用MySQL里面的自定义变量，每次从原来SQL中取一条数据，让rn自增。

## 附录二

```sql
create table t(id int primary key, name varchar(20));
insert into t(id,name) values(1,'c_9');
insert into t(id,name) values(2,'c_8');
insert into t(id,name) values(3,'c_7');
insert into t(id,name) values(4,'c_6');
insert into t(id,name) values(5,'c_5');
insert into t(id,name) values(6,'c_4');
insert into t(id,name) values(7,'c_3');
insert into t(id,name) values(8,'c_2');
insert into t(id,name) values(9,'c_1');

select rownum,t.* from t where id  > 7 or rownum < 3;
-- 在oracle中的执行上面这条sql的结果如下
-- id为1和2的行满足rownum的条件，id为8和9的行满足id>7的条件
+--------+----+------+
| ROWNUM | ID | NAME |
+--------+----+------+
|       1|  1 | c_9  |
+--------+----+------+
|       2|  2 | c_8  |
+--------+----+------+
|       3|  8 | c_2  |
+--------+----+------+
|       4|  9 | c_1  |
+--------+----+------+

-- 如果改成limit，在MySQL上执行的效果大概是这样
select @rn:=@rn+1 rownum,t.* from (select * from t where id > 8 limit 2) t,(select @rn:=0) r;
+--------+----+------+
| rownum | id | name |
+--------+----+------+
|      1 |  9 | c_1  |
+--------+----+------+
```

<br> 

<center>  ·End·  </center>
