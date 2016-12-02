title: "iOS sqlite3相关使用"
date: 2015-04-15 18:22:51
tags: ["sql", "date"]
categories: ["iOS","Objective-C"]
---

### sqlite3相关使用

> [IOS SQLite数据库](http://www.w3cschool.cc/ios/ios-sqlite.html)
> [sqlite3使用简介](http://www.cnblogs.com/kfqcome/archive/2011/06/27/2136999.html)
> [ios初学SQLite3（创建、插入、查询、更新数据库和表）](http://blog.csdn.net/mad1989/article/details/9322307)

首先需要添加`libsqlite3.dylib`库
```objc
+(DBManager *)getSharedInstance
{
    if (!sharedInstance) {
        sharedInstance = [[super allocWithZone:NULL] init];
        [sharedInstance createDB];
    }
    return sharedInstance;
}

-(BOOL)createDB
{
    BOOL isSuccess = YES;
    NSString *docsDir;
    NSArray *dirPaths;
    //get the documents directory
    dirPaths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    docsDir = dirPaths[0];
    //build the path to the database file
    databasePath = [[NSString alloc] initWithString:[docsDir stringByAppendingPathComponent:@"test.db"]];
    NSFileManager *filemgr = [NSFileManager defaultManager];
    if ([filemgr fileExistsAtPath:databasePath] == NO) {
        const char *dbpath = [databasePath UTF8String];
        if (sqlite3_open(dbpath, &database) == SQLITE_OK) {
            char *errMsg;
            const char *sql_stmt = "create table if not exists test(regno integer primary key, name text)";
            if (sqlite3_exec(database, sql_stmt, NULL, NULL, &errMsg) != SQLITE_OK) {
                isSuccess = NO;
                NSLog(@"Failed to create table");
                sqlite3_free(errMsg);
            }
            sqlite3_close(database);
            return isSuccess;
        }else{
            isSuccess = NO;
            NSLog(@"Failed to open/create database");
        }
    }
    return isSuccess;
}

-(bool)saveData:(NSString *)name
{
    const char *dbpath = [databasePath UTF8String];
    if (sqlite3_open(dbpath, &database) == SQLITE_OK) {
        NSString *insertSQL = [NSString stringWithFormat:@"insert into test(regno,name)values(NULL,\"%@\")", name];
        const char *insert_stmt = [insertSQL UTF8String];
        sqlite3_prepare_v2(database, insert_stmt, -1, &statement, NULL);
        if (sqlite3_step(statement) == SQLITE_DONE) {
            return YES;
        }
        sqlite3_reset(statement);
    }
    return NO;
}

-(BOOL)deleteAllData
{
    const char *dbpath = [databasePath UTF8String];
    if (sqlite3_open(dbpath, &database) == SQLITE_OK) {
        NSString *deleteSQL = @"delete from test";
        const char *insert_stmt = [deleteSQL UTF8String];
        sqlite3_prepare_v2(database, insert_stmt, -1, &statement, NULL);
        if (sqlite3_step(statement) == SQLITE_DONE) {
            return YES;
        }
        sqlite3_reset(statement);
    }
    return NO;
}

-(NSArray *) findAll
{
    const char *dbpath = [databasePath UTF8String];
    if (sqlite3_open(dbpath, &database) == SQLITE_OK) {
        NSString *querySQL = @"select name from test";
        const char *query_stmt = [querySQL UTF8String];
        NSMutableArray *resultArray = [[NSMutableArray alloc] init];
        if (sqlite3_prepare_v2(database, query_stmt, -1, &statement, NULL) == SQLITE_OK) {
            while (sqlite3_step(statement) == SQLITE_ROW) {
                NSString *name = [[NSString alloc] initWithUTF8String:(const char *) sqlite3_column_text(statement, 0)];
                [resultArray addObject:name];
            }
            return resultArray;
        }
    }
    return nil;
}
```

相关方法

* sqlite3_open()：打开数据库

在操作数据库之前，首先要打开数据库。这个函数打开一个sqlite数据库文件的连接并且返回一个数据库连接对象。这个操作同时程序中的第一个调用的sqlite函数，同时也是其他sqlite api的先决条件。许多的sqlite接口函数都需要一个数据库连接对象的指针作为它们的第一个参数。
```cpp
int sqlite3_open(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);

int sqlite3_open16(
  const void *filename,   /* Database filename (UTF-16) */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);

int sqlite3_open_v2(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb,         /* OUT: SQLite db handle */
  int flags,              /* Flags */
  const char *zVfs        /* Name of VFS module to use */
);
```
假如这个要被打开的数据文件不存在，则一个同名的数据库文件将被创建。如果使用`sqlite3_open`和`sqlite3_open_v2`的话，数据库将采用UTF-8的编码方式，`sqlite3_open16`采用UTF-16的编码方式。
如果sqlite数据库被成功打开（或创建），将会返回SQLITE_OK,否则将会返回错误码。`Sqlite3_errmsg()`或者`sqlite3_errmsg16`可以用于获得数据库打开错误码的英文描述
```cpp
const char *sqlite3_errmsg(sqlite3*);
const void *sqlite3_errmsg16(sqlite3*);
```

参数说明：

__filename__：需要被打开的数据库文件的文件名，在sqlite3_open和sqlite3_open_v2中这个参数采用UTF-8编码，而在sqlite3_open16中则采用UTF-16编码

__ppDb__：一个数据库连接句柄被返回到这个参数，即使发生错误。唯一的一场是如果sqlite不能分配内存来存放sqlite对象，ppDb将会被返回一个NULL值。

__flags__：作为数据库连接的额外控制的参数，可以是SQLITE_OPEN_READONLY，SQLITE_OPEN_READWRITE和SQLITE_OPEN_READWRITE|SQLITE_OPEN_CREATE中的一个，用于控制数据库的打开方式，可以和SQLITE_OPEN_NOMUTEX，SQLITE_OPEN_FULLMUTEX， SQLITE_OPEN_SHAREDCACHE，以及SQLITE_OPEN_PRIVATECACHE结合使用，具体的详细情况可以查阅文档

* Sqlite3_prepare()
这个函数将sql文本转换成一个准备语句（prepared statement）对象，同时返回这个对象的指针。这个接口需要一个数据库连接指针以及一个要准备的包含SQL语句的文本。它实际上并不执行（evaluate）这个SQL语句，它仅仅为执行准备这个sql语句
```cpp
int sqlite3_prepare(
  sqlite3 *db,            /* Database handle */
  const char *zSql,       /* SQL statement, UTF-8 encoded */
  int nByte,              /* Maximum length of zSql in bytes. */
  sqlite3_stmt **ppStmt,  /* OUT: Statement handle */
  const char **pzTail     /* OUT: Pointer to unused portion of zSql */
);

int sqlite3_prepare_v2(
  sqlite3 *db,            /* Database handle */
  const char *zSql,       /* SQL statement, UTF-8 encoded */
  int nByte,              /* Maximum length of zSql in bytes. */
  sqlite3_stmt **ppStmt,  /* OUT: Statement handle */
  const char **pzTail     /* OUT: Pointer to unused portion of zSql */
);
```
参数：

db：数据指针

zSql：sql语句，使用UTF-8编码

nByte：如果nByte小于0，则函数取出zSql中从开始到第一个0终止符的内容；如果nByte不是负的，那么它就是这个函数能从zSql中读取的字节数的最大值。如果nBytes非负，zSql在第一次遇见’/000/或’u000’的时候终止

pzTail：上面提到zSql在遇见终止符或者是达到设定的nByte之后结束，假如zSql还有剩余的内容，那么这些剩余的内容被存放到pZTail中，不包括终止符

ppStmt：能够使用`sqlite3_step()`执行的编译好的准备语句的指针，如果错误发生，它被置为NULL，如假如输入的文本不包括sql语句。调用过程必须负责在编译好的sql语句完成使用后使用`sqlite3_finalize()`删除它。

说明

如果执行成功，则返回`SQLITE_OK`，否则返回一个错误码。推荐在现在任何的程序中都使用`sqlite3_prepare_v2`这个函数，`sqlite3_prepare`只是用于前向兼容

*  sqlite3_setp()
这个过程用于执行有前面sqlite3_prepare创建的准备语句。这个语句执行到结果的第一行可用的位置。继续前进到结果的第二行的话，只需再次调用sqlite3_setp()。继续调用sqlite3_setp()知道这个语句完成，那些不返回结果的语句（如：INSERT，UPDATE，或DELETE），sqlite3_step()只执行一次就返回
```cpp
int sqlite3_step(sqlite3_stmt*);
```
返回值

函数的返回值基于创建sqlite3_stmt参数所使用的函数，假如是使用老版本的接口sqlite3_prepare()和sqlite3_prepare16()，返回值会是 SQLITE_BUSY， SQLITE_DONE， SQLITE_ROW， SQLITE_ERROR 或 SQLITE_MISUSE，而v2版本的接口sqlite3_prepare_v2()和sqlite3_prepare16_v2()则会同时返回这些结果码和扩展结果码。

对所有V3.6.23.1以及其前面的所有版本，需要在sqlite3_step()之后调用sqlite3_reset()，在后续的sqlite3_ step之前。如果调用sqlite3_reset重置准备语句失败，将会导致sqlite3_ step返回SQLITE_MISUSE，但是在V3. 6.23.1以后，sqlite3_step()将会自动调用sqlite3_reset。
```cpp
int sqlite3_reset(sqlite3_stmt *pStmt);
```
sqlite3_reset用于重置一个准备语句对象到它的初始状态，然后准备被重新执行。所有sql语句变量使用sqlite3_bind*绑定值，使用sqlite3_clear_bindings重设这些绑定。Sqlite3_reset接口重置准备语句到它代码开始的时候。sqlite3_reset并不改变在准备语句上的任何绑定值，那么这里猜测，可能是语句在被执行的过程中发生了其他的改变，然后这个语句将它重置到绑定值的时候的那个状态。

* sqlite3_column()
这个过程从执行sqlite3_step()执行一个准备语句得到的结果集的当前行中返回一个列。每次sqlite3_step得到一个结果集的列停下后，这个过程就可以被多次调用去查询这个行的各列的值。对列操作是有多个函数，均以sqlite3_column为前缀
```cpp
const void *sqlite3_column_blob(sqlite3_stmt*, int iCol);
int sqlite3_column_bytes(sqlite3_stmt*, int iCol);
int sqlite3_column_bytes16(sqlite3_stmt*, int iCol);
double sqlite3_column_double(sqlite3_stmt*, int iCol);
int sqlite3_column_int(sqlite3_stmt*, int iCol);
sqlite3_int64 sqlite3_column_int64(sqlite3_stmt*, int iCol);
const unsigned char *sqlite3_column_text(sqlite3_stmt*, int iCol);
const void *sqlite3_column_text16(sqlite3_stmt*, int iCol);
int sqlite3_column_type(sqlite3_stmt*, int iCol);
sqlite3_value *sqlite3_column_value(sqlite3_stmt*, int iCol);
```
第一个参数为从sqlite3_prepare返回来的prepared statement对象的指针，第二参数指定这一行中的想要被返回的列的索引。最左边的一列的索引号是0，行的列数可以使用sqlite3_colum_count()获得。这些过程会根据情况去转换数值的类型，sqlite内部使用sqlite3_snprintf()去自动进行这个转换

* sqlite3_finalize
```cpp
int sqlite3_finalize(sqlite3_stmt *pStmt)
```
这个过程销毁前面被sqlite3_prepare创建的准备语句，每个准备语句都必须使用这个函数去销毁以防止内存泄露。在空指针上调用这个函数没有什么影响，同时可以准备语句的生命周期的任一时刻调用这个函数：在语句被执行前，一次或多次调用sqlite_reset之后，或者在sqlite3_step任何调用之后不管语句是否完成执行

*  sqlite3_close
这个过程关闭前面使用sqlite3_open打开的数据库连接，任何与这个连接相关的准备语句必须在调用这个关闭函数之前被释放

`sqlite3_exec`是sqlite3_prepare_v2，sqlite3_step()和sqlite3_finalize()的封装，能让程序多次执行sql语句而不要写许多重复的代码。

`sqlite3_exec`接口执行0或多个UTF-8编码的，分号分割的sql语句，传到第二个参数中。如果sqlite3_exec的第三个参数回调函数指针不为空，那么它会为每个来自执行的SQL语句的结果行调用（也就是说回调函数会调用多次，上面例子中会返回2个结果行，因而会被执行2次），第4个参数是传给回调函数的第一个参数，如果回调函数指针为空，那么回调不会发生同时结果行被忽略。

如果在执行sql语句中有错误发生，那么当前的语句的执行被停止，后续的语句也被跳过。第五个参数不为空的时候，它被分配内存并写入了错误信息，所以在`sqlite3_exec`后面需要调用`sqlite3_free`去释放这个对象以防止内存泄露

回调函数：
```cpp
int (*callback)(void*,int,char**,char**),  /* Callback function */
```
第一个参数通过sqlite3_exec的第第四个参数传入的
第二个参数是结果行的列数
第三个参数是行中列数据的指针
第四个参数是行中列名称的指针

----

#### SQLite函数

> 来自[SQLite学习手册(内置函数)](http://www.cnblogs.com/stephen-liu74/archive/2012/01/13/2322027.html)

| 方法 | 说明 |
| ---- | ---- |
| avg(z) | 获取平均值 |
| count(z) | 获取行数，如果为"*"的话则包含了NULL的行 |
| group_concat(z[,y]) | 按指定的分隔符(y,默认为",")连接成一字符串 |
| max(z) | 返回最大值 |
| min(z) | 返回最小值 |
| sum(z) | 返回总和 |
| total(z) | 返回总和，如果为NULL则返回0 |
| abs(z) | 返回绝对值 |
| changes() | 返回最近(insert/update/delete)受影响的行数,与`sqlite3_changes()`相同 |
| coalesce(z,x,···) | 返回第一个为非NULL的值 |
| ifnull(z,x) | 返回第一个为非NULL的值 |
| length(z) | 返回长度 |
| lower(z) | 将字符串转换为小写 |
| ltrim(z[,x]) | 如果没有可选参数x，该函数将移除参数z左侧的所有空格符。如果有参数x，则移除z左侧的任意在x中出现的字符。最后返回移除后的字符串 |
| max(z,x,···) | 返回参数中最大值，如果有参数为null,返回null |
| min(z,x,···) | 返回参数中的最小值，如果有参数为null,返回null |
| nullif(z,x) | 如果函数参数相同，返回NULL，否则返回第一个参数 |
| random() | 返回整型的伪随机数 |
| replace(z,x,c) | 返回z中x替换为c的字符串,z保持不变 |
| round(z[,x]) | 返回z的四舍五入值，x为小数点后的数位，默认为0 |
| ltrim(z[,x]) | 如果没有可选参数x，该函数将移除参数z右侧的所有空格符。如果有参数x，则移除z右侧的任意在x中出现的字符。最后返回移除后的字符串 |
| substr(X,Y[,Z]) | 返回函数参数X的子字符串，从第Y位开始(X中的第一个字符位置为1)截取Z长度的字符，如果忽略Z参数，则取第Y个字符后面的所有字符。如果Z的值为负数，则从第Y位开始，向左截取abs(Z)个字符。如果Y值为负数，则从X字符串的尾部开始计数到第abs(Y)的位置开始 |
| total_changes() | 返回至连接打开起(insert/update/delete)受影响的行数,与`sqlite3_total_changes()`相同 |
| trim(x[,y]) | 如果没有可选参数Y，该函数将移除参数X两侧的所有空格符。如果有参数Y，则移除X两侧的任意在Y中出现的字符。最后返回移除后的字符串 |
| upper(z) | 返回大写的参数字符串 |
| typeof(z) | 返回函数参数数据类型的字符串表示形式，如"Integer、text、real、null"等 |

日期函数
 SQLite主要支持以下四种与日期和时间相关的函数，如：
    1). date(timestring, modifier, modifier, ...)
    2). time(timestring, modifier, modifier, ...)
    3). datetime(timestring, modifier, modifier, ...)
    4). strftime(format, timestring, modifier, modifier, ...)
    以上所有四个函数都接受一个时间字符串作为参数，其后再跟有0个或多个修改符。其中strftime()函数还接受一个格式字符串作为其第一个参数。strftime()和C运行时库中的同名函数完全相同。至于其他三个函数，date函数的缺省格式为："YYYY-MM-DD"，time函数的缺省格式为："HH:MM:SS"，datetime函数的缺省格式为："YYYY-MM-DD HH:MM:SS"。   
    
 * strftime函数的格式信息：
  
| 格式 | 说明 |
| ---- | ---- |
| %d | 	day of month: 00 |
| %f | 	fractional seconds: SS.SSS |
| %H | 	hour: 00-24 |
| %j | 	day of year: 001-366 |
| %J | 	Julian day number |
| %m | 	month: 01-12 |
| %M | 	minute: 00-59 |
| %s | 	seconds since 1970-01-01 |
| %S | 	seconds: 00-59 |
| %w | 	day of week 0-6 with Sunday==0 |
| %W | 	week of year: 00-53 |
| %Y | 	year: 0000-9999 |
| %% | 	% |

    需要额外指出的是，其余三个时间函数均可用strftime来表示，如：
    date(...)         strftime('%Y-%m-%d', ...)
    time(...)         strftime('%H:%M:%S', ...)
    datetime(...)   strftime('%Y-%m-%d %H:%M:%S', ...)
    
   * 时间字符串的格式：
    见如下列表：
    1). YYYY-MM-DD
    2). YYYY-MM-DD HH:MM
    3). YYYY-MM-DD HH:MM:SS
    4). YYYY-MM-DD HH:MM:SS.SSS
    5). HH:MM
    6). HH:MM:SS
    7). HH:MM:SS.SSS
    8). now
    5)到7)中只是包含了时间部分，SQLite将假设日期为2000-01-01。8)表示当前时间。
   
   * 修改符：
    见如下列表：
    1). NNN days
    2). NNN hours
    3). NNN minutes
    4). NNN.NNNN seconds
    5). NNN months
    6). NNN years
    7). start of month
    8). start of year
    9). start of day
    10).weekday N     
    1)到6)将只是简单的加减指定数量的日期或时间值，如果NNN的值为负数，则减，否则加。7)到9)则将时间串中的指定日期部分设置到当前月、年或日的开始。10)则将日期前进到下一个星期N，其中星期日为0。注：修改符的顺序极为重要，SQLite将会按照从左到右的顺序依次执行修改符。
    
* 示例：
```
    --返回当前日期。
    sqlite> SELECT date('now');  
    2012-01-15    
    --返回当前月的最后一天。
    sqlite> SELECT date('now','start of month','1 month','-1 day');
    2012-01-31
    --返回从1970-01-01 00:00:00到当前时间所流经的秒数。
    sqlite> SELECT strftime('%s','now');
    1326641166    
    --返回当前年中10月份的第一个星期二是日期。
    sqlite> SELECT date('now','start of year','+9 months','weekday 2');
    2012-10-02   
```

----

#### sqlite3_exec 事物

> 来自[sql操作sqlite3_exec](http://blog.csdn.net/theogo/article/details/25195633)

在执行SQL语句之前和SQL语句执行完毕之后加上 
```cpp
sqlite3_exec(db, "BEGIN;", 0, 0, &zErrMsg); 
//执行SQL语句 
sqlite3_exec(db, "COMMIT;", 0, 0, &zErrMsg);
```
这样SQLite将把全部要执行的SQL语句先缓存在内存当中，然后等到COMMIT的时候一次性的写入数据库，这样数据库文件只被打开关闭了一次，效率自然大大的提高。

----

#### sqlite 建表相关属性

* timestap

> 来自[SQLite3中TimeStamp的使用问题](http://www.cnblogs.com/GDLMO/archive/2010/07/19/1780920.html)

sqlite中的timestamp需要转换时区才能得到正确的数据，创建表的时候可以指定默认值
```sql
TimeStamp NOT NULL DEFAULT (datetime('now','localtime')
```

* id自增

> 来自[sqlite3自增key设定(创建自增字段)](http://www.cnblogs.com/zhw511006/archive/2010/09/08/1821596.html)

指定字段为`INTEGER PRIMARY KEY AUTOINCREMENT`然后每次插入时插入`NULL`即可

----

#### sqlite 相关查询

> 来自
> [sql语句insert ignore into 和replace into区别](http://blog.csdn.net/yangkun0824/article/details/9029819)
> [SQLite - UPSERT *not* INSERT or REPLACE](http://stackoverflow.com/questions/418898/sqlite-upsert-not-insert-or-replace)

* insert into 数据库会检查主键，如果主键出现重复，则会报错
* insert ignore into 如果出现相同的记录，会忽略插入
* replace into 数据库需要有primary key，或者unique索引，如果有相同数据，则替换。否则和insert into效果一样

