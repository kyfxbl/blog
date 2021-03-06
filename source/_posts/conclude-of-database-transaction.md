title: 数据库事务小结
date: 2013-09-23 10:52
categories: database 
---
![transaction](http://pic.kyfxbl.com/transaction.jpeg)
最近在看jdbc specification，昨天刚看到Transactions那节。有些概念感觉比较模糊，所以就搜索了相关的信息，总结一下。不仅数据库有事务，业务上也有事务的感念。本文单指数据库事务
<!--more-->

# 事务的概念 

事务就是将一组操作视为一个整体，具有ACID的性质： 

原子性（Atomic）：事务中的操作要么全部成功，要么全部失败 

一致性（Consistent）：事务处理前后，数据状态保持一致 

隔离性（Isolated）：一个事务的处理，不影响另一个事务的处理 

持久性（Durable）：事务处理成功后，应该在数据库里持久化 

前面讲的概念是从网上找的，举例子会比较简单，就是到处都能看到的那个银行转账的例子：A转100块钱给B。此过程包含有2个操作： A-100； B+100；这2个操作必须一起成功，或者一起失败，这个就是最简单的事务 

# 事务边界 

事务边界的概念是为了描述，事务何时开始，以及何时结束。有2种： 

显式事务：以begin transaction开始，以commit或者rollback结束 

隐式事务：每次当前事务commit或者rollback之后，就自动开启新事务 

这个似乎和数据库产品的具体实现有关 

# 事务并发与事务隔离 

如果对于一个数据库，只有一个连接对其进行操作，那么就不存在并发的问题。数据库中的所有记录都是该连接独占的 

但是在实际中，有大量的连接同时操作数据库，所以就涉及到并发的问题。因此才需要设置事务隔离，使得不同的事务不会互相影响 

可以这样认为： 事务相互的隔离程度越高，往往意味着更多的等待，因此并发能力就越弱；相反，事务的并发能力越强，事务相互的隔离程度就越弱，比较容易出现错误数据的问题 

这是无法避免的，当前无法做到并发能力和数据正确性两全其美，只能“权衡选择”一个折中的方式 

# 事务并发可能引起的问题 

对于表t：

| *id* | *name* | *age* |
| 1 | mike | 20 |

## 脏读（dirty read） 

事务A：
```
select t.age where t.id = 1;
```
事务B：
```
update t set age = 21 where t.id = 1;
rollback;
```

当事务B的update语句执行，但是回滚之前，事务A读取到了21的值；然后事务B回滚了，age值回滚为20。事务A读取到的是一个错误的值，这种情况叫脏读 

## 不可重复读（unrepeatable read） 

事务A：
```
select t.age where t.id = 1;
select t.age where t.id = 1;
```
事务B：
```
update t set age = 21 where t.id = 1;
commit;
```

事务A首先读出值20，然后事务B执行并commit，事务A读出21。在同一个事务A里，读出不同的值，这种情况叫不可重复读 

## 幻读（phantom read） 

事务A：
```
select t.name where t.age = 20;
select t.name where t.age = 20;
```

事务B：
```
insert into t values ("2","jim","20");
commit;
```

事务A读出1条记录，然后事务B执行并commit，事务A读出2条记录。在同一个事务A里，先后查询得到不同的记录条数，这种情况叫幻读 

# 事务隔离级别 

为了避免上述的3种（不好的）情况，可以设置数据库的事务隔离级别 

事务隔离级别越低，就越无法避免上述情况，但是由于加锁少，也就越不影响并发性能。相反，事务隔离级别越高，越能避免上述情况的发生，但是对并发性能的影响就越大 

所以，设置事务隔离级别，实际上是对数据正确性，和并发性能的一种权衡取舍

| *隔离级别* | *脏读* | *不可重复读* | *幻读* |
| 读未提交（read uncommitted） | 允许 | 允许 | 允许 |
| 读已提交（read committed） | 不允许 | 允许 | 允许 |
| 可重复读（repeatable read） | 不允许 | 不允许 | 允许 |
| 可串行（serializable） | 不允许 | 不允许 | 不允许 |

# 在java中处理事务

对于java应用，数据库操作通过jdbc实现。CRUD都是通过方法间接实现的，事务的控制也转移到代码里。所以把数据库事务，不准确地称为“java transaction” 

java transaction有2种，一种是jdbc事务，一种是JTA事务 

jdbc事务的相关接口是java.sql.Connection，有
```
setAutoCommit();
commit();
rollback();
```
等事务相关方法 

JTA事务比较复杂点，跟本文关系比较远，后面有机会再专门说明 

另外，现在很多应用框架都对事务处理进行了封装，比如hibernate通过session管理事务、spring的声明式事务等。其实都是对jdbc事务和JTA事务的封装