title: 通过block，消除打开和关闭sqlite数据的冗余代码
date: 2014-01-08 14:35
categories: iOS
---
我们的应用，CRUD操作很多，非常依赖sqlite数据库。一开始使用原生sqlite3 API，比较痛苦。后来用FMDatabase重构了一次，代码简单多了。不过还是美中不足，存在大量重复的open and close代码，所以今天用block方式进一步优化
<!--more-->

以下是示例代码：

```
-(void) testInvokeWithBlock
{
    [self doJobWithBlock:^(FMDatabase* db){
    
        NSString *createTableSql = @"create table tb_kyfxbl_test (name varchar, age number);";
        NSString *insertSql = @"insert into tb_kyfxbl_test values ('zsd', 29);";

        [db executeUpdate:createTableSql];
        [db executeUpdate:insertSql];
    }];
}

-(void) doJobWithBlock:(void (^)(FMDatabase* db))block
{
    NSString* dbFilePath = [YLSGlobalUtils getDatabaseFilePath];
    FMDatabase *db = [[FMDatabase alloc] initWithPath:dbFilePath];

    [db open];

    block(db);

    [db close];
}
```
很薄的一层封装，对原来的代码不构成额外的负担

另外，可以看到，这种形式和javascript中常见的callback几乎一模一样，都把函数作为参数。此函数不在定义的时刻调用，而是在未来的某个时间点调用

区别在于，上面的示例代码是同步的，而js的callback大部分是异步的。如果对示例代码稍加改造：

```
-(void) doJobWithBlock:(void (^)(FMDatabase* db))block
{
    double delayInSeconds = 2.0;
    dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delayInSeconds * NSEC_PER_SEC));

    dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
        NSString* dbFilePath = [YLSGlobalUtils getDatabaseFilePath];
        FMDatabase *db = [[FMDatabase alloc] initWithPath:dbFilePath];

        [db open];

        block(db);

        [db close];
    });
}
```

这样就更像callback function了。所以，回调函数只是语言层面的特性，并非一定是同步或者异步的，事实上即使在javascript里，也存在同步的回调函数。只不过大部分是异步的而已