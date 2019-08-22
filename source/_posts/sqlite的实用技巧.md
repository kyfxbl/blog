title: sqlite的实用技巧
date: 2015-02-05 20:02
categories: database 
---
![sqlite](http://pic.kyfxbl.com/sqlite.jpeg)
介绍sqlite的一些小技巧，DDL改变某一列，以及利用事务提升批量insert的性能
<!--more-->

# alter table改变一列

sqlite不支持alter table的时候修改某列的定义，只能全表DML，所以如果需要改变某一列，做法是：

1、先建一张临时表，把原来表中的数据复制进去

2、删除旧表

3、新增表

4、从临时表中把数据复制回新表

5、删除临时表

```
NSString *sql1 = @"create table tb_users_temp as select * from tb_users";
NSString *sql2 = @"drop table tb_users;";
NSString *sql3 = @"CREATE TABLE IF NOT EXISTS tb_users (...);";
NSString *sql4 = @"insert into tb_users select * from tb_users_temp;";
NSString *sql5 = @"drop table tb_users_temp";
```

# 使用事务提升sqlite insert的性能

昨天发现sqlite插入性能很低，搜索了一下发现，其实sqlite的插入可以做到每秒50000条，但是处理事务的速度慢

看下面这个官方FAQ的第19条，大体的意思是对于客户端来说，其实sqlite处理insert是足够快的，但是处理事务确实很慢
[sqlite FAQ#19](http://www.sqlite.org/faq.html#q19)

我原本的代码没有使用事务，所以每条insert语句都默认为一个事务。解决的办法是加上事务，效果浮夸，执行SQL的时间从10秒缩短到了0.07秒

发现了这个以后，我就尝试把可能的地方都加上事务，但是原本程序有一处逻辑，是执行一大堆insert，如果主键冲突就自然无视。但是如果把这堆sql变成事务，就会影响正确数据的插入，所以又把insert语句改成insert or ignore：

```
insert or ignore into test (id, key) values (20001, 'kyfxbl');
```

然后再放到一个事务里，效率大大提升
