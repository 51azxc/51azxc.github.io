title: "Oracle函数命令"
date: 2015-05-05 17:50:08
tags: ["sql", "date"]
categories: ["Database", "Oracle"]
---

#### 查看oracle版本

> [查看oracle版本命令](http://blog.csdn.net/cnham/article/details/5388016)

```sql
select * from v$instance
select * from product_component_version
```

<!-- more -->

----

#### insert···select···union语句

> [oracle能不能使用insert...select...union语句？](http://zhidao.baidu.com/question/376044972.html)

如果需要插入常量，则需要使用`dual`表,例如
```sql
insert into table1 
select 'a' from dual
union
select 'b' from dual
```

----

#### merge into用法

> [oracle merge into 用法详解](http://blog.csdn.net/edgenhuang/article/details/3587912)

`merge`命令可以在SQL语句中对一个表同时执行`insert`和`update`操作。
```sql
merge into table1 a
using table2 b
on (a.id = b.id)
when matched then
update
set a.name = b.name
when not matched then
insert values(b.id,b.name,b.age)
```
`merge`命令给table1插入table2的数据，其中通过`on`关键字来匹配相关数据，如果相等则更新相关数据，如果不相等则插入新的数据。其中`update`及`insert`语句可以追加`where`子句进行条件筛选。`update`及`insert`语句也不是非必需的，同时也允许在`update`子句包含`delete`语句进行删除操作，`delete`语句必须跟`where`条件。匹配`delete where`条件但不匹配`on`条件的行不会被从表中删除
```sql
merge into table1 a
using table2 b
on (a.id = b.id)
when matched then
update
set a.name = b.name
delete where(a.name!='a')
```

----

#### 按分隔符截取字符串

> [Oracle函数，按分隔符截取字符串](http://bbs.csdn.net/topics/370163558)

使用`regexp_substr`函数利用正则表达式截取
```sql
select regexp_substr('aaa,bb,ccccc,ddd,vvv','[^,]+',1,3)
from dual
;
```
结果为 'ccccc'

----

#### trunc函数

> [Oracle TRUNC函数的正确用法](http://database.51cto.com/art/201004/197703.htm)

**TRUNC(for dates)**
`TRUNC(date[,fmt])`为指定元素而截去的日期值。
`date`: 一个日期值
`fmt`: 日期格式，该日期将由指定的元素格式所截去。忽略它则由最近的日期截去
```sql
select TRUNC(TO_DATE(’24-Nov-1999 08:00 pm’,’dd-mon-yyyy hh:mi am’)) from dual
-- result: 24-Nov-1999 12:00:00 am
select TRUNC(TO_DATE(’24-Nov-1999 08:37 pm’,’dd-mon-yyyy hh:mi am’,’hh’)) from dual
-- result: 24-Nov-1999 08:00:00 am
```
`round (date,''format'')`未指定format时，如果日期中的时间在中午之前，则将日期中的时间截断为12 A.M.(午夜,一天的开始),否则进到第二天。
`trunc(date,''format'')`未指定format时，将日期截为12 A.M.，不考虑是否在中午之前的条件。

**TRUNC(for number)**
`TRUNC(number[,decimals])`函数返回处理后的数值，其工作机制与`ROUND`函数极为类似，只是该函数不对指定小数前或后的部分做相应舍入选择处理，而统统截去。
`number` 待做截取处理的数值
`decimals` 指明需保留小数点后面的位数。可选项，忽略它则截去所有的小数部分
下面是该函数的使用情况：
```sql
select TRUNC（89.985，2） from dual
--result: 89.98
select TRUNC（89.985） from dual
--result: 89
select TRUNC（89.985，-1） from dual
--result: 80
```
注意：第二个参数可以为负数，表示为小数点左边指定位数后面的部分截去，即均以0记

----

#### trim函数

> [oracle trim函数用法详解](http://blog.csdn.net/indexman/article/details/7748766)

基本语法
```sql
TRIM([ { { LEADING | TRAILING | BOTH }
         [ trim_character ]
       | trim_character
       }
       FROM 
     ]
     trim_source
    )
```
参数解释：
`leading`  开头字符
`trailing`  结尾字符
`both`  开头和结尾字符
`trim_character`  去除的字符
`trim_source`  修剪源

`trim`函数用来去除一个字符串的开头或结尾（或两者）的字符。函数返回一个varchar2类型值。该值最大的长度等于`trim_source`的长度。`trim_character`和`trim_source`都可以为以下任意一种数据类型：**CHAR**, **VARCHAR2**， **NCHAR**, **NVARCHAR2**， **CLOB**, **NCLOB**。返回值的类型与`trim_source`的数据类型一致

如果指定`leading`参数，oracle数据库将去除任何等于`trim_character`的开头字符
```sql
select trim(leading 'a' from 'abc') from dual;
-- result: bc
```
如果指定`traling`参数，oracle将去除任何等于`trim_character`的结尾字符
```sql
select trim(trailing 'c' from 'abc') from dual;
-- result: ab
```
如果指定了`both`参数或者三个参数都未指定，oracle将去除任何等于`trim_character`的开头和结尾字符
```sql
select trim(both 'a' from 'abca') from dual;
-- result: bc
```
如果没有指定`trim_character`参数，默认去除的值为空格
```sql
select trim(both from ' abc ') from dual;
-- result: abc
```
如果只指定修剪源`trim_source`，oracle将去除`trim_source`的开头和结尾的空格
```sql
select trim(' abc ') from dual;
-- result: abc
```
如果`trim_source`和`trim_character`有一个为**null**，则trim函数返回**null**
```sql
select trim(both null from 'abca') from dual;
select trim(both 'a' from null) from dual;
-- result: (null)
```

----

#### NULL相关

> [oracle中的NVL,NVL2,NULLIF,COALESCE几个通用函数](http://jenny-86.iteye.com/blog/465725)

* `NVL(expr1,expr2)`: 如果`expr1`为null则返回`expr2`,否则返回`expr1`
* `NVL2(expr1,expr2, expr3)`: 如果`expr1`为null则返回`expr3`，否则返回`expr2`
* `NULLIF(exp1,expr2)`: 如果`expr1`与`expr2`相等则返回null，否则返回`expr1`
* `Coalesce(expr1, expr2, expr3….. exprn)`: 返回第一个不为null的参数，全为空则返回null

----

#### 日期函数

> [oracle_查询date只显示日期不显示时间](http://blog.csdn.net/wocjj/article/details/7490994)
> [oracle 日期显示英文格式](http://blog.csdn.net/meboy88scofiled/article/details/5035045)
> [日期如何增加一年](http://bbs.csdn.net/topics/360053715)

`TO_CHAR(d [, fmt [, 'nlsparams'] ])`函数用于将日期转换成指定格式的字符串
`d`是`Date`类型的变量，`fmt`是我们指定的日期时间格式，如果不显式指定就用 Oracle 的默认值。 `fmt`里常用的跟日期时间有关的占位符如下：
MM 用数字表示的月份(例如，07)
MON 缩写的月份名称(例如，JUL)
MONTH 完整的月份名称(例如，JULY)
DD 日期(例如, 24)
DY 星期几的缩写(例如，FRI)
YYYY 用4位表示的年份(例如, 2008)
YY 用2位表示的年份，取年份的后两位(例如，08)
RR 跟 YY 类似，但两位表示的年份被近似到 1950 到 2049 这个范围里的年份
AM (或 PM) 上下午指示符
HH 12进制表示的时间(1-12)
HH24 24进制表示的时间(0-23)
MI 分钟(0-59)
SS 秒(0-59)

如果需要输出英文格式的日期，可以指定`nlsparams`为`nls_date_language=american`
```sql
select to_char(sysdate,'dd-mon-yyyy','nls_date_language=american') from dual
```

也可以使用`ALTER SESSION SET NLS_DATE_LANGUAGE='AMERICAN'`来指定当前会话的`NLS_DATE_LANGUAGE`属性，但是断开此次会话后，数据库会恢复到默认的格式

`TO_DATE(char [, fmt [, 'nlsparams'] ])` 将字符串转换成指定格式的日期。`char` 是表示日期和时间的字符串。`fmt` 的表示方法和 `TO_CHAR` 函数一样

如果需要给日期增加一年可以使用`add_months`函数
```sql
select add_months(sysdate,12) from dual;
```

----

#### rank函数

> [oracle rank()函数总结](http://keke-wanwei.iteye.com/blog/138632)

`rank`是一个给数据确定等级的函数.

以销售为例,有地区,年,月,销售员,销售额,记录这五个字段.我们可以按地区,年,月,销售额对销售员进行排序,这样对销售员来说就相当于有一个等级概念了,第一名就是销售最高的......,如果我们要找出每个地区,年,月,销售额的前三名销售员.SQL如何写?
```sql
SELECT area_code, YEAR, MONTH, saleroom,saler   
       RANK () OVER 
    (PARTITION BY area_code,year,month ORDER BY area_code,year,month,saleroom ) RANK   
FROM t_sale
```
现在RANK 就是1,2,3,3,3,6,有了这个字段,就很容易得到前三名的销售员了.
新问题:销售额50000块在深圳,2007年5月能排到第几?
```sql
SELECT    
      RANK('SHENZHEN',2007,5,50000)  WITHIN GROUP    
      (ORDER BY area_code,year,month,saleroom) Rank    
FROM T_SALE
```
上面这个SQL就可以搞定了.要注意的是,Rank()里的参数必须为常数,或常值表达式,里面参数的个数,类型也要和order by后字段的类型相对应.
上面就是Rank函数的两个用法.另外还有一个`dense_rank()`,它的用法和`rank()`一样,只是计算等级的方式不同.例如上面的1,2,3,3,3,6.用dense_rank() 就是1,2,3,3,3,4.

----

#### over函数

> [Oracle over() 函数的实际用法](http://database.51cto.com/art/201005/197847.htm)

`over()` 函数是对分析函数的一种条件解释，直接点就是 给分析函数加条件吧。

在网上看见比较常用的就是 与 `sum()`、`rank()` 函数使用。接下来就用分析下两种函数结合over的用法。

以下测试使用scott用户下的emp表数据。
```sql
select a.empno as 员工编号
,a.ename as 员工姓名
,a.deptno as 部门编号
,a.sal as 薪酬
,sum(sal) over (partition by deptno) 按部门求薪酬总和
from scott.emp a;
```

| 部门编号 | 员工姓名 | 员工编码 | 薪酬 | 按部门求薪酬总和 |
| --- | --- | --- | --- | --- |
| 7934 | MILLER | 10 | 1300 | 8750 |
| 7782 | CLARK | 10 | 2450 | 8750 |
| 7839 | KING | 10 | 5000 | 8750 |
| 7369 | SMITH | 20 | 800 | 10875 |

可以从结果上看到sum()函数对部门区分进行了求和统计。其中“partition by”官方点的说法叫做"分区"，其实就是统计的范围条件

----

#### SQL(oracle) 取得分组后最大值记录 

> [SQL(oracle) 取得分组后最大值记录](http://blog.csdn.net/rfb0204421/article/details/7546724)

`row_number() OVER (PARTITION BY COL1 ORDER BY COL2)`表示根据COL1分组，在分组内部根据 COL2排序，而此函数计算的值就表示每组内部排序后的顺序编号（组内连续的唯一的).
与`rownum`的区别在于：使用`rownum`进行排序的时候是先对结果集加入伪列`rownum`然后再进行排序，而此函数在包含排序从句后是先排序再计算行号码

* `row_number()`和`rownum`差不多，功能更强一点（可以在各个分组内从1开时排序）．
* `rank()`是跳跃排序，有两个第二名时接下来就是第四名（同样是在各个分组内）．
* `dense_rank()`是连续排序，有两个第二名时仍然跟着第三名。相比之下`row_number`是没有重复值的
* `lag（arg1,arg2,arg3)`: arg1是从其他行返回的表达式;arg2是希望检索的当前行分区的偏移量。是一个正的偏移量，时一个往回检索以前的行的数目。arg3是在arg2表示的数目超出了分组的范围时返回的值。

```sql
select row_number() over(order by sale/cnt desc) as sort, sale/cnt
from (
select -60 as sale,3 as cnt from dual union
select 24 as sale,6 as cnt from dual union
select 50 as sale,5 as cnt from dual union
select -20 as sale,2 as cnt from dual union
select 40 as sale,8 as cnt from dual
);
```
执行结果:

| SORT | SALE/CNT |
| --- | --- |
| 1 | 10 |
| 2 | 5 |
| 3 | 4 |
| 4 | -10 |
| 5 | -20 |

查询员工的工资,按部门排序
```sql
select ename,sal,row_number() over (partition by deptno order by sal desc) as sal_order from scott.emp;
```
执行结果:

| ENAME | SAL | SAL_ORDER
| --- | --- | --- |
| KING | 5000 | 1 |
| CLARK | 2450 | 2 |
| MILLER | 1300 | 3 |
| SCOTT | 3000 | 1 |

并列排名
```sql
select deptno,sal,dense_rank() over(partition by deptno order by sal) as dense_rank_order from scott.emp order by deptn; 
```
执行结果:

 | DEPTNO | SAL | DENSE_RANK_ORDER |
 | --- | --- | --- |
 | 30 | 950 | 1 |
 | 30 | 1250 | 2 |
 | 30 | 1250 | 2 |
 | 30 | 1500 | 3 |


----

#### listagg、lag及lead函数

> [oracle listagg函数、lag函数、lead函数实例](http://blog.sina.com.cn/s/blog_4cef5c7b01016efp.html)

**listagg**
```sql
listagg (measure_expr[,'delimiter']) with group (order_by_clause ) [over query_parition_clause]
```
`listagg`的作用是将分组范围内的所有行特定列的记录加以合并成行。函数签名中的`measure_expr`为分组中每个列的表达式，而`delimiter`为合并分割符。如果`delimiter`不设置的话，就表示无分割符。中间`within group`后面的`order_by_clause`表示的是进行合并中要遵守的排序顺序。而后面的`over`子句表明`listagg`是具有分析函数analyze funcation特性的。具体采用listagg有三个场景。

* 当无分组的single-list情况下
如果要获取到deptno为30的所有员工横行记录
```bash
SQL> select * from emp where deptno=30;

EMPNO ENAME     JOB        MGR HIREDATE         SAL     COMM DEPTNO
----- ---------- --------- ----- ----------- --------- --------- ------
 7499 ALLEN     SALESMAN  7698 1981-2-20    1600.00   300.00    30

 7521 WARD      SALESMAN  7698 1981-2-22    1250.00   500.00    30

 7654 MARTIN    SALESMAN  7698 1981-9-28    1250.00  1400.00    30

 7698 BLAKE     MANAGER   7839 1981-5-1     2850.00              30

 7844 TURNER    SALESMAN  7698 1981-9-8     1500.00     0.00    30

 7900 JAMES     CLERK     7698 1981-12-3     950.00              30

6 rows selected

--按照empno进行排序

SQL> select listagg(ename,' , ') within group (order byempno) from emp where deptno=30;

LISTAGG(ENAME,',')WITHINGROUP(
------------------------------------------------------------
ALLEN , WARD , MARTIN , BLAKE , TURNER , JAMES
```

* 在有分组条件下的listagg使用
如果要使用分组统计各个部门的所有员工列表。
```bash
SQL> select deptno, listagg(ename,' ,') within group (order by empno) from emp group by deptno;

DEPTNO LISTAGG(ENAME,',')WITHINGROUP(
------ -------------------------------------
   10 CLARK ,KING ,MILLER

   20 SMITH ,JONES ,SCOTT ,ADAMS ,FORD

   30 ALLEN ,WARD ,MARTIN ,BLAKE ,TURNER ,JAMES
```

* 使用over分组情况
如果要统计所有工作十年以上员工和他们相同部门的员工信息，就需要在listagg的基础上加入over分析函数子句
```bash
select deptno, ename, 
       listagg(ename, ' , ') within group (order by empno) 
       over (partition by deptno) as emp_list 
from emp 
where hiredate<=add_months(sysdate,-10*12);

 

DEPTNO ENAME     EMP_LIST

------ ------

   10 CLARK     CLARK , KING , MILLER

   10 KING      CLARK , KING , MILLER

   10 MILLER    CLARK , KING , MILLER

   20 SMITH     SMITH , JONES , SCOTT , ADAMS , FORD

   20 JONES     SMITH , JONES , SCOTT , ADAMS , FORD

   20 SCOTT     SMITH , JONES , SCOTT , ADAMS , FORD

   20 ADAMS     SMITH , JONES , SCOTT , ADAMS , FORD

   20 FORD      SMITH , JONES , SCOTT , ADAMS , FORD

   30 ALLEN     ALLEN , WARD , MARTIN , BLAKE , TURNER , JAMES

   30 WARD      ALLEN , WARD , MARTIN , BLAKE , TURNER , JAMES

   30 MARTIN    ALLEN , WARD , MARTIN , BLAKE , TURNER , JAMES

   30 BLAKE     ALLEN , WARD , MARTIN , BLAKE , TURNER , JAMES

   30 TURNER    ALLEN , WARD , MARTIN , BLAKE , TURNER , JAMES

   30 JAMES     ALLEN , WARD , MARTIN , BLAKE , TURNER , JAMES

 

14 rows selected
```

**lag函数“取到上个月的销售额”**
我们在进行销售数据统计汇总时候，经常遇到这样的需求：“对比上月（上季度同月份或者上年度同月份），我们的销售变化情况如何？”。我们的销售数据通常是对应单月信息，如下所示。
```bash
SQL> select * from sales_qual;

MONT        QUALITIES PRICE
---------- ----------- ------
2011-01          1000 23.40

2011-02          1020 23.40

2011-03          1030 33.40

2011-04          1035 10.30
```
如果要获取到之前月份的信息，没有SQL专门函数就意味着需要使用PL/SQL代码进行反复的迭代获取。现在，我们可以使用lag函数来轻易实现这个功能。
```sql
lag(value_expr [,offset [, default] ) { respect | ignore } nulls ] over ([query_parition_clause] order_by_clause)

lag(value_expr [ { respect | ignore } nulls] [,offset [,default] ]) over ([query_parition_clause] order_by_clause)
```

`lag`函数是一个典型的分析函数。它提供了在不使用自连接的情况下，访问多个数据行的能力。在返回多个结果行的时候，`lag`函数可以访问到向上特定`offset`偏移行的数据。
`value_expr`就是访问到向上数据行进行的操作。`offset`是返回偏移的函数，默认值为**1**。`over`中，可以定义内部分析的顺序列。

如果我们要获取到对应上个月的销售数据，SQL语句如下：
```sql
select mont,qualities, 
       lag(qualities,1) over (order by mont) as "Next Month Qual"
from sales_qual
order by mont;

MONT        QUALITIES Next Month Qual
---------- ----------- ---------------
2011-01          1000

2011-02          1020           1000

2011-03          1030           1020

2011-04          1035           1030
```
之后对销量变化率的处理就方便了，可以进行增长率比对等操作。那么，如果是上一年度或者上一季度的数据呢？我们只需要调节`offset`，从1变化为12或者3就可以了。

最后，对`ignore/respect nulls`子句的使用是什么呢？该子句的作用是确定当`value_exp`r表达式计算出的数值为空null的时候，该列如何进行计算。`ignore nulls`的作用就是忽略上面计算为空的行，采用上上行row的计算结果。`respect nulls`的作用是直接反映为null。`respect nulls`为默认值。
```sql
select * from sales_qual;

MONT        QUALITIES PRICE
---------- ----------- ------
2011-04          1035 10.30

2011-05                12.30

2011-06               

6 rows selected

select mont,qualities, 
       lag(qualities,1)ignore nullsover (order by mont) as "Next Month Qual"
from sales_qual
order by mont;

MONT        QUALITIES Next Month Qual
---------- ----------- ---------------
2011-04          1035           1030

2011-05                          1035

2011-06                          1035

6 rows selected

select mont,qualities, 
       lag(qualities,1) respect nulls over (order by mont) as "Next Month Qual"
from sales_qual
order by mont;

MONT        QUALITIES Next Month Qual
---------- ----------- ---------------
2011-04          1035           1030

2011-05                          1035

2011-06               

6 rows selected
```

**lead函数获取下一个月销售量**
有`lag`的获取上个`offset`处理行的函数，就有`lead`函数处理下一个处理行的函数。lead函数实际上就是`lag`的逆向过程。
相关各项参数与`lag`函数的相同。区别就在于`lead`函数获取的是排序后结果集合的后`offset`数据行记录。
```sql
select mont,qualities, 
       lead(qualities,1) over (order by mont) as "Next Month Qual"
from sales_qual
order by mont;

MONT        QUALITIES Next Month Qual
---------- ----------- ---------------
2011-01          1000           1020

2011-02          1020           1030

2011-03          1030           1035

2011-04          1035
```