title: 在ios中使用FMDB
date: 2013-12-13 23:32
categories: iOS 
---
本来应用里很多代码，都是用原生的sqlite3 API，确实感到很不方便，API极不友好。昨天看到唐巧的博客，知道了FMDB，试用一下果然不错，记录一下
<!--more-->

这个开源项目的github地址是：[FMDB](https://github.com/ccgus/fmdb)

安装最方便的方式是用CocoaPods来安装，见官方文档。FMDB把SQL操作分为update和query，所以API不是executeUpdate，就是executeQuery，下面是几个简单的例子：

```
-(void) clearDeleteRecordForTable:(NSString*)tableName withRecords:(NSMutableArray*)deleteIds
{
    NSString *dbFilePath = [YLSGlobalUtils getDatabaseFilePath];
    FMDatabase *db = [FMDatabase databaseWithPath:dbFilePath];

    NSString *sql = @"delete from tb_deleteRecord where table_name = ? and id = ?";

    [db open];
    for(NSString *deleteId in deleteIds){
        [db executeUpdate:sql, tableName, deleteId];
    }
    [db close];
}
```
这个是executeUpdate的例子，挺简单的。下面是一个executeQuery的例子：

```
-(NSMutableArray*) queryDeleteData:(NSString*)tableName
{
    NSMutableArray *result = [NSMutableArray new];

    NSString *dbFilePath = [YLSGlobalUtils getDatabaseFilePath];
    FMDatabase *db = [FMDatabase databaseWithPath:dbFilePath];

    [db open];
    FMResultSet *rs = [db executeQuery:@"select id from tb_deleteRecord where upper(table_name) = upper(?)", tableName];
    while ([rs next]) {
        [result addObject:[rs objectForColumnName:@"id"]];
    }
    [db close];

    return result;
}
```
值得注意的是，以前用原生sqlite3 API，拼接sql语句的时候，placeholder用的是%@，但这个方式是FMDB所反对的：

Thus, you SHOULD NOT do this (or anything like this):

```
[db executeUpdate:[NSString stringWithFormat:@"INSERT INTO myTable VALUES (%@)", @"this has \" lots of ' bizarre \" quotes '"]];

```
Instead, you SHOULD do:

```
[db executeUpdate:@"INSERT INTO myTable VALUES (?)", @"this has \" lots of ' bizarre \" quotes '"];
```
总的来说，FMDB不推荐使用%@拼接sql，而是要求使用?或者:name来占位。这点我很喜欢，因为我本来写的N个bug，都是因为在sql里忘记加引号造成的，比如说：

```
@"select * from table where name = %@"
```
这个sql就写错了，应该写成：

```
@"select * from table where name = '%@'"
```

非常麻烦，一不小心写漏了SQL执行就会报错，现在用FMDB，错误的几率就减小了很多。另外FMResultSet的objectForColumnName等方法也十分方便。下面还有一个更方便的：

```
-(NSMutableArray*) queryUpdateData:(NSString*)tableName
{
    NSMutableArray *result = [NSMutableArray new];

    NSString *dbFilePath = [YLSGlobalUtils getDatabaseFilePath];
    FMDatabase *db = [FMDatabase databaseWithPath:dbFilePath];

    [db open];
    long now = [[NSDate date] timeIntervalSince1970];
    FMResultSet *rs = [db executeQuery:@"select * from %@ where modify_date between ? and ? and (create_date not between ? and ?)", tableName, latestBackupTime, now, latestBackupTime, now];
    while ([rs next]) {
        [result addObject:[rs resultDictionary]];
    }
    [db close];

    return result;
}
```
就是[FMResultSet resultDictionary]方法，可以直接将一行的记录转换成NSDictionary，比原生sqlite3 API好用很多

总的来说，FMDB的API很直观，也很方便。项目切换到FMDB的成本也非常低，强烈推荐试一下