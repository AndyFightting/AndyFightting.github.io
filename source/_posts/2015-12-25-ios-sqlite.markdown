---
layout: post
title: "iOS中的SQLite"
date: 2015-12-25 10:44:52 +0800
comments: true
categories: iOS
---
![image](/myimg/ios/sqlite.png)

[SQLite](http://www.sqlite.org/)是一个轻量级的关系型数据库，在iOS和Android手机中都有用到，C语言的面向过程的函数式编程。[SQLitePersistentObject](https://github.com/woooooojianjie/SQLitePersistentObject)和[FMDB](https://github.com/ccgus/fmdb)框架都是基于SQLite开发的。SQLitePersistentObject更像是个ORM框架，它是由Jeff LaMarche在2008年开发的，所以不是ARC的，要用的请加上**-fno-objc-arc**，且作者已经没有维护了，所以还是推荐使用更加广泛的FMDB。<!--more-->

### *SQLite
添加依赖库libsqlite3.tbd，用SQLite来个原生的增删改查。下面一次性完成了创建并打开数据库，创建表，插入一条数据，查询数据，最后关闭数据库。
```
NSString* docsdir = [NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
NSString* dbpath = [docsdir stringByAppendingPathComponent:@"user.sqlite"];
sqlite3 *database;
//创建并打开数据库*****************
if (sqlite3_open([dbpath UTF8String], &database) != SQLITE_OK) { //SQLITE_OK == 0
   sqlite3_close(database);
   NSAssert(0,@"数据库打开失败"); //若状态不是0就打开失败
}
//创建表格*****************
NSString *createSql = @"CREATE TABLE IF NOT EXISTS student (student_id INTEGER PRIMARY KEY ,student_name TEXT);";
char *createError;
//sqlite3_exec这个方法可以执行那些没有返回结果的操作，例如创建、插入、删除、修改等。这个函数包含了sqlite3_prepare这个函数的操作，目的是将UTF-8格式的SQL语句转换为编译后的语句
if (sqlite3_exec(database, [createSql UTF8String], NULL, NULL, &createError) != SQLITE_OK) {
    sqlite3_close(database);
    NSAssert(0,@"创建表错误：%s", createError);
}
//插入或修改数据*****************  
char *update = "INSERT OR REPLACE INTO student VALUES (?,?)";  
sqlite3_stmt *statement;  
if (sqlite3_prepare_v2(database, update, -1, &statement, nil) == SQLITE_OK) {  
    //将值保存到指定的列,列从1开始！！  
    sqlite3_bind_int(statement, 1, 4);//1是列，4是student_id  
    //第四个参数代表第三个参数中需要传递的长度。对于C字符串来说，-1表示传递全部字符串。第五个参数是一个回调函数，比如执行后做内存清除工作。  
    sqlite3_bind_text(statement, 2, [@"studentName" UTF8String], -1, NULL);  
}    
if (sqlite3_step(statement) != SQLITE_DONE) {  
    NSAssert(0,@"更新数据出错");  
}  
sqlite3_finalize(statement);  
//查询数据库***************** 
NSString *querySql = @"SELECT * FROM student;";
sqlite3_stmt *selectStmt;
if (sqlite3_prepare_v2(database, [querySql UTF8String], -1, &selectStmt, nil) == SQLITE_OK) {
    while (sqlite3_step(selectStmt) == SQLITE_ROW) {
        int studentId = sqlite3_column_int(selectStmt, 0);//后面的数字对应每一列
        char *cString = (char *)sqlite3_column_text(selectStmt, 1);
        NSString* studentName = [[NSString alloc]initWithUTF8String:cString];
        NSLog(@"student_id = %d   student_name = %@",studentId,studentName);
    }
    sqlite3_finalize(selectStmt);
}
sqlite3_close(database);//最后关闭数据库
```
![image](/myimg/ios/select.png)
![image](/myimg/ios/table.png)





