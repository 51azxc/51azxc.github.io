title: "MySQL函数命令"
date: 2015-05-05 11:32:10
tags: ["date", "string", "sql"]
categories: ["database", "mysql"]
---

### MySQL函数
#### Every derived table must have its own alias

> [关于子查询 Every derived table must have its own alias 的错误](http://bbs.9izhan.com/thread-11660-1-1.html)

在子查询中必须给查询的结果一个别名，不然会报上述错误
```sql
select * from (select * from table1) as t
```

----

#### SELECT INTO 和 INSERT INTO SELECT 两种表复制语句
> 来自[SELECT INTO 和 INSERT INTO SELECT 两种表复制语句](http://www.cnblogs.com/freshman0216/archive/2008/08/15/1268316.html)

`insert into select`要求目标表Table2必须存在，由于目标表Table2已经存在，所以我们除了插入源表Table1的字段外，还可以插入常量。
```sql
insert into Table2(field1,field2,...) select value1,value2,... from Table1
```

`select into`要求目标表Table2不存在，因为在插入时会自动创建表Table2，并将Table1中指定字段数据复制到Table2中。
```sql
select vale1, value2 into Table2 from Table1
```

----

#### mysql update查询结果

> [mysql update不能直接使用select的结果](http://blog.sina.com.cn/s/blog_3c62c21f01013qch.html)

**SQL Server**中可以直接使用结果集更新
```sql
update table1 set a.field = (select field from table2)
```
在**MySQL**中上述语句是行不通的，如果需要完成上述效果，需要使用`inner join`
```sql
update table1 a inner join table2 b set a.field1 = b.field1 where a.field2 = b.field2
```

----

#### mysql查询区分大小写
> 来自[mysql查询区分大小写](http://www.cnblogs.com/trying/p/3669101.html)

Mysql默认查询是不分大小写的，可以在SQL语句中加入`binary`来区分大小写；
**BINARY**不是函数，是类型转换运算符，它用来强制它后面的字符串为一个二进制字符串，可以理解为在字符串比较的时候区分大小写
```sql
select * from table1 a where a.id = BINARY 'a'
```

----

#### MySQL行转列
##### case when 语句

> [mysql中case when语句的使用示例](http://database.51cto.com/art/201010/229082.htm)

基本用法
```sql
select filed1,
  case 
    when filed2 = 'a' then 'a'
    when filed2 = 'b' then 'b'
    else 'c' 
  end
from table1;
```

```sql
select filed1,
  case field2
    when 'a' then 'a'
    when 'b' then 'b'
    else 'c' 
  end
from table1;
```
复合?
```sql
select filed1,filed2
  case 
    when filed1 = 'a' then 'a'
      when filed2 = 'b' then 'b'
      else 'c' 
  end
from table1;
```

----

##### if sum语句

> [mysql行转列(if + sum) ](http://blog.csdn.net/cdy102688/article/details/14006515)

mysql行转列可以使用`sum`与`if`函数配合完成
所用数据如下

| name | type | score |
| ---- | ---- | ----- |
| a | chinese | 80 |
| a | math | 70 |
| b | chinese | 90 |
| b | math | 100 |

```sql
select name, 
  sum(if(type='chinese',score,0)) as chinese, 
  sum(if(type='math',score,0)) as math
from table1
group by name
```
结果如下

| name | chinese | math |
| ---- | ------- | ---- |
| a | 80 | 70 |
| b | 90 | 100 |

**总结**：if主要是用来创建新列，并将非对应学科的分数写为0，用sum或max配合group by保证取出的值是学科对应的值，这样就可以完成行转列了

----

#### 多行记录合并成一行

> [Oracle和Mysql中将多行记录合并为一行](http://blog.sina.com.cn/s/blog_51a493580101jsaw.html)

MySQL使用函数`group_concat`
```sql
select 
  field1,
  field2,
  group_concat(field3 order by field3 separator "|") 
from table1 
group by field1
```
这里的`separator`指定分隔符为"|"

Oracle使用函数`WMSYS.WM_CONCAT`
```sql
select
  field1,
  WMSYS.WM_CONCAT(field2) as field2
from table1
group by field1
```

----

#### 获取小数点后两位

> [mysql格式化小数保留小数点后两位](http://www.jb51.net/article/44378.htm)
> [MySQL CAST与CONVERT 函数的用法](http://www.nowamagic.net/librarys/veda/detail/2044)
> [mysql数据库，结果保留4位小数，小数点后四位](http://zhidao.baidu.com/link?url=LePIQDrUswcVQX2PYliFLJ2_5haop43H30D1MDO0Z-LCHDMo9S1NaFJhqoqqengUSNbuzJIsbEekGQM1HyUfWa)

* 使用`format`函数
```sql
select format(12345.678,2)
```
返回结果为 12,345.68
此函数整数部分超过三位的时候以逗号分割，并且返回的结果是string类型的。

* 使用`truncate`函数
```sql
select truncate(12345.678,2)
```
返回结果为 12345.67
此函数并不能达到四舍五入的效果

* 使用`convert`函数
```sql
select convert(12345.678,decimal)
```
返回结果为 12346
此函数为转换格式，将所选数字转换为浮点数类型，接收参数主要有 **二进制**`BINARY`,**字符型**`CHAR()`,**日期**`DATE`,**时间**`TIME`,**日期时间型**`DATETIME`,**浮点数**`DECIMAL`,**整数**`SIGNED`**,无符号整数**`UNSIGNED`.

* 使用`round`函数
```sql
select round(12345.678,2)
```
返回结果为 12345.68
符合预期

----

#### **NULL**与空值的区别

> [Mysql探究之null与not null](http://my.oschina.net/junn/blog/161769)

首先，我们要搞清楚“空值” 和 “NULL” 的概念： 
1. 空值是不占用空间的
2. mysql中的**NULL**其实是占用空间的

**NOT NULL**的字段是不能插入“NULL”的，只能插入“空值”(即`''`)
**NULL** 其实并不是空值，而是要占用空间，所以mysql在进行比较的时候，**NULL** 会参与字段比较，所以对效率有一部分影响,而且对表索引时不会存储**NULL**值的，所以如果索引的字段可以为**NULL**，索引的效率会下降很多

判断不为空
```sql
select * from table1 where field1 <> ''
```
判断不为**NULL**
```sql
select * from table1 where field1 is not null
```

----

#### **IFNULL**,**NULLIF**与**ISNULL**的区别

> [MySql 里的IFNULL、NULLIF和ISNULL用法](http://www.cnblogs.com/JuneZhang/archive/2010/08/26/1809306.html)

* `ifnull(expr1,expr2)`: 如果`expr1`不为**NULL**时，返回``expr1`,否则返回`expr2`
* `nullif(expr1,expr2)`: 如果`expr1=expr2`，则返回**NULL**,否则返回`expr1`
* `isnull(expr)`: 如果`expr`为**NULL**，则返回1，否则返回0

----

#### 操作字符串函数
> 来自[Mysql字符串截取函数SUBSTRING的用法说明](http://www.jb51.net/article/27458.htm)

* `left(被截取字符串，截取长度)`： 从左开始截取字符串
* `right(被截取字符串，截取长度)`: 从右开始截取字符串
* `substring(被截取字段，[从第几位开始截取]，截取长度)`: 截取字符串，中括号内的参数为可选，如果为负数则是从字符串右边开始计数
* `substring_index(被截取字段，关键字，关键字出现的次数)`: 按关键字截取字符串,如果关键字出现的次数为负数，则是从字符串右边开始计数

----

#### 日期函数
> 来自
> [MySQL日期时间函数大全](http://www.cnblogs.com/zeroone/archive/2010/05/05/1727659.html)
> [mysql相似于oracle的to_char() to_date()方法](http://blog.sina.com.cn/s/blog_68f4b9f201013vql.html)
> [MYSQL如何计算两个日期间隔天数](http://blog.163.com/i_yuhan/blog/static/19834210020124174495366/)

* `DAYOFWEEK(date)`: 返回日期date是星期几(1=星期天···7=星期六,ODBC标准)
* `WEEKDAY(date)`: 返回日期date是星期几(0=星期一···6= 星期天)
* `DAYOFMONTH(date)`: 返回date是一月中的第几日(在1到31范围内) 
* `DAYOFYEAR(date)`: 返回date是一年中的第几日(在1到366范围内)
* `DAYNAME(date)`: 返回date是星期几(按英文名返回)
* `MONTHNAME(date)`: 返回date是几月(按英文名返回)
* `QUARTER(date)`: 返回date是一年的第几个季度
* `WEEK(date,first)`: 返回date是一年的第几周(first默认值0,first取值1表示周一是周的开始,0从周日开始)
* `YEAR(date)`: 返回date的年份
* `MONTH(date)`: 返回date中的月份
* `HOUR(time)`: 返回time的小时数(范围是0到23)
* `MINUTE(time)`: 返回time的分钟数(范围是0到59)
* `SECOND(time)`: 返回time的秒数(范围是0到59)
* `PERIOD_ADD(P,N)`: 增加N个月到时期P并返回(P的格式YYMM或YYYYMM)
* `PERIOD_DIFF(P1,P2)`: 返回在时期P1和P2之间月数(P1和P2的格式YYMM或YYYYMM) (`P1<P2为负数`)
* `DATE_ADD(date,INTERVAL expr type)`,
  `DATE_SUB(date,INTERVAL expr type)`,
  `ADDDATE(date,INTERVAL expr type) `,
  `SUBDATE(date,INTERVAL expr type)`: 对日期时间进行加减法运算。`ADDDATE()`和`SUBDATE()`是`DATE_ADD()`和`DATE_SUB()`的同义词,date是一个DATETIME或DATE值,expr对date进行加减法的一个表达式字符串,type指明表达式expr应该如何被解释，例如
```sql
select date_add('2015-05-04', INTERVAL -1 DAY)
--返回结果2015-05-03
select date_add('2015-05-04', INTERVAL '1 2:3:4' DAY_SECOND)
--返回结果2015-05-05 02:03:04
select date_add('2015-05-04', INTERVAL '2:3' MINUTE_SECOND)
--返回结果2015-05-04 00:02:03
select date_add('2015-05-04', INTERVAL '-1 10' DAY_HOUR)
--返回结果2015-05-02 14:00:00
```

| type | 意义 | expr |
| ---- | ---- | ---- |
| SECOND | 秒 | SECONDS |
| MINUTE | 分 | MINUTES |
| HOUR | 时 | HOURS |
| DAY | 天 | DAYS |
| MONTH | 月 | MONTHS |
| YEAR | 年 | YEARS |
| MINUTE_SECOND | 分钟:秒 | "MINUTES:SECONDS" |
| HOUR_MINUTE | 小时:分钟 | "HOURS:MINUTES" |
| DAY_HOUR | 天和小时 | "DAYS HOURS" |
| YEAR_MONTH | 年和月 | "YEARS-MONTHS" |
| HOUR_SECOND | 小时, 分钟 | "HOURS:MINUTES:SECONDS" |
| DAY_MINUTE | 天, 小时, 分钟 | "DAYS HOURS:MINUTES" |
| DAY_SECOND | 天, 小时, 分钟, 秒 | "DAYS HOURS:MINUTES:SECONDS" |

`expr`中允许任何标点做分隔符,如果所有是`DATE`值时结果是一个`DATE`值,否则结果是一个`DATETIME`值
如果type关键词不完整,则MySQL从右端取值,`DAY_SECOND`因为缺少小时分钟等于`MINUTE_SECOND`
如果增加`MONTH`、`YEAR_MONTH`或`YEAR`,天数大于结果月份的最大天数则使用最大天数

* `TO_DAYS(date)`: 返回日期date是西元0年至今多少天(不计算1582年以前)
* `FROM_DAYS(N)`:　给出西元0年至今多少天返回DATE值(不计算1582年以前)
* `STR_TO_DATE(date,format)`: 将date字符串转成date格式
* `DATE_FORMAT(date,format)`: 根据format字符串格式化date值

| 标识符 | 意义 |
| ------ | ---- |
| %M | 月名字(January……December) |
| %W | 星期名字(Sunday……Saturday) |
| %D | 有英语前缀的月份的日期(1st, 2nd, 3rd, 等等） |
| %Y | 年, 数字, 4 位 |
| %y | 年, 数字, 2 位 |
| %a | 缩写的星期名字(Sun···Sat) |
| %d | 月份中的天数, 数字(00···31) |
| %e | 月份中的天数, 数字(0……31) |
| %m | 月, 数字(01……12) |
| %c | 月, 数字(1……12) |
| %b | 缩写的月份名字(Jan……Dec) |
| %j | 一年中的天数(001……366) |
| %H | 小时(00……23) |
| %k | 小时(0……23) |
| %h | 小时(01……12) |
| %I | 小时(01……12) |
| %l | 小时(1……12) |
| %i | 分钟, 数字(00……59) |
| %r | 时间,12 小时(hh:mm:ss [AP]M) |
| %T | 时间,24 小时(hh:mm:ss) |
| %S | 秒(00……59) |
| %s | 秒(00……59) |
| %p | AM或PM |
| %w | 一个星期中的天数(0=Sunday···6=Saturday） |
| %U | 星期(0……52), 这里星期天是星期的第一天 |
| %u | 星期(0……52), 这里星期一是星期的第一天 |
| %% | 字符% |

```sql
select date_format('2015-05-04 17:13:40','%W %M %Y %H:%i:%s')
--返回结果 Monday May 2015 17:13:40
```

* `TIME_FORMAT(time,format)`: 和`DATE_FORMAT()`类似,但`TIME_FORMAT`只处理小时、分钟和秒(其余符号产生一个NULL值或0)
* `CURDATE()`与`CURRENT_DATE()`: 以'YYYY-MM-DD'或YYYYMMDD格式返回当前日期值(根据返回值所处上下文是字符串或数字) 
* `CURTIME()`与`CURRENT_TIME()`:以'HH:MM:SS'或HHMMSS格式返回当前时间值(根据返回值所处上下文是字符串或数字) 
* `NOW()`,`SYSDATE()`与`CURRENT_TIMESTAMP()`: 以'YYYY-MM-DD HH:MM:SS'或YYYYMMDDHHMMSS格式返回当前日期时间(根据返回值所处上下文是字符串或数字) 
* `UNIX_TIMESTAMP()`与`UNIX_TIMESTAMP(date)`: 返回一个Unix时间戳(从'1970-01-01 00:00:00'GMT开始的秒数,date默认值为当前时间)
* `FROM_UNIXTIME(unix_timestamp)`: 以'YYYY-MM-DD HH:MM:SS'或YYYYMMDDHHMMSS格式返回时间戳的值
* `FROM_UNIXTIME(unix_timestamp,format)`: 以format字符串格式返回时间戳的值
* `SEC_TO_TIME(seconds)`: 以'HH:MM:SS'或HHMMSS格式返回秒数转成的TIME值
* `TIME_TO_SEC(time)`: 返回time值有多少秒
* `datediff(date1,date2)`: 计算两个日期之间间隔的天数

----

#### MySQL批量更新数据，有则更新，无则插入

> [ON DUPLICATE KEY UPDATE重复插入时更新](http://lobert.iteye.com/blog/1604122)

`insert on duplicate key update`
在`INSERT`语句中指定了`ON DUPLICATE KEY UPDATE`，并且插入行后会导致在一个`UNIQUE`索引或`PRIMARY KEY`中出现重复值，则执行旧行`UPDATE`
您可以在`UPDATE`子句中使用`VALUES(col_name)`函数从`INSERT...UPDATE`语句的INSERT部分引用列值。换句话说，如果没有发生重复关键字冲突，则`UPDATE`子句中的`VALUES(col_name)`可以引用被插入的`col_name`的值。本函数特别适用于多行插入。`VALUES()`函数只在`INSERT...UPDATE`语句中有意义，其它时候会返回NULL。
```sql
INSERT INTO table (a,b,c) VALUES (1,2,3),(4,5,6) ON DUPLICATE KEY UPDATE c=VALUES(a)+VALUES(b);
```

`replace`
我们在使用数据库时可能会经常遇到这种情况。如果一个表在一个字段上建立了唯一索引，当我们再向这个表中使用已经存在的键值插入一条记录，那将会抛出一个主键冲突的错误。当然，我们可能想用新记录的值来覆盖原来的记录值。如果使用传统的做法，必须先使用DELETE语句删除原先的记录，然后再使用INSERT插入新的记录。而在MySQL中为我们提供了一种新的解决方案，这就是`REPLACE`语句。**使用REPLACE插入一条记录时，如果不重复，REPLACE就和INSERT的功能一样，如果有重复记录，REPLACE就使用新记录的值来替换原来的记录值。**
使用REPLACE的最大好处就是可以将DELETE和INSERT合二为一，形成一个原子操作。这样就可以不必考虑在同时使用**DELETE**和**INSERT**时添加事务等复杂操作了。
在使用REPLACE时，表中必须有唯一索引，而且这个索引所在的字段不能允许空值，否则REPLACE就和INSERT完全一样的。
在执行REPLACE后，系统返回了所影响的行数，如果返回1，说明在表中并没有重复的记录，如果返回2，说明有一条重复记录，系统自动先调用了DELETE删除这条记录，然后再记录用INSERT来插入这条记录。如果返回的值大于2，那说明有多个唯一索引，有多条记录被删除和插入。
REPLACE的语法和INSERT非常的相似，如下面的REPLACE语句是插入或更新一条记录。
```sql
REPLACE INTO users (id,name,age) VALUES(123, 'a', 10);
```
REPLACE也可以使用SET语句
```sql
REPLACE INTO users SET id = 123, name = 'a', age = 10;
```

----

####中文排序

> [让MySQL支持中文排序的实现方法](http://www.jb51.net/article/28876.htm)

使用`CONVERT`来转换字符集
```sql
select * from mytable order by CONVERT(chineseColumnName USING gbk);
```