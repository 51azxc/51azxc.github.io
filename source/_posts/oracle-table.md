title: "Oracle表"
date: 2015-04-21 20:38:41
tags: ["sql", "tablespace"]
categories: ["database", "oracle"]
---

#### 创建表时先判断是否已存在

> [oracle（PL/SQL）表操作：创建表时检查数据库是否存在该表，若存在删除再创建 ](http://blog.csdn.net/cnham/article/details/5388016)

使用sql 2005 来执行“创建表之前判断表是否存在 如果有就删除表，再创建”的操作语句非常简单
```sql
if exists (
select * from sysobjects 
where id = OBJECT_ID('STUDENTS]') and OBJECTPROPERTY(id, 'IsUserTable') = 1) 
DROP TABLE [STUDENTS]
```
PL/SQL中
```sql
declare 
  cnt number;
begin
  --查询要创建的表是否存在
  select count(*)into cnt from user_tables where table_name='STUDENTS';
  if cnt>0 then
    execute immediate 'drop table STUDENTS';
    dbms_output.put_line('表存在，删除成功!');
  end if;
  ---删除之后再创建该表
  execute immediate 
    'CREATE TABLE STUDENTS(SNAME CHAR (8) NOT NULL) tablespace Users' ;
end;
```

<!-- more -->

----

#### Oracle临时表

> [oracle 关于临时表 创建 删除 拷贝](http://blog.sina.com.cn/s/blog_4b3c1f950102dwbn.html)
> [oracle临时表的用法总结](http://blog.csdn.net/wyzxg/article/details/1882347)

临时表分为 **会话级临时表** 和 **事务级临时表**

* 会话级的临时表因为这这个临时表中的数据和你的当前会话有关系，当你当前SESSION不退出的情况下，临时表中的数据就还存在，而当你退出当前SESSION的时候，临时表中的数据就全部没有了，当然这个时候你如果以另外一个SESSION登陆的时候是看不到另外一个SESSION中插入到临时表中的数据的。即两个不同的SESSION所插入的数据是互不相干的。当某一个SESSION退出之后临时表中的数据就被截断（truncate table，即数据清空）了。
* 事务级临时表是指该临时表与事务相关，当进行事务提交(`commit`)或者事务回滚(`rollback`)的时候，临时表中的数据将自行被截断，其他的内容和会话级的临时表的一致（包括退出SESSION的时候，事务级的临时表也会被自动截断）

临时表创建后，除非 主动删除，是不会自动删除的（除非重启数据库）

建立方法:

* `ON COMMIT PRESERVE ROWS` 定义了创建会话级临时表的方法
```sql
Create Global Temporary Table Table_Name
(Col1 Type1,Col2 Type2...) On Commit Preserve Rows
```
* `ON COMMIT DELETE ROWS` 定义了建立事务级临时表的方法
```sql
Create Global Temporary Table Table_Name
(Col1 Type1,Col2 Type2...) On Commit Delete Rows
```

临时表的不足之处
1. 不支持lob对象
2. 不支持主外键关系

ORACLE临时表和SQLSERVER临时表异同


SQL SERVER临时表

SQL SERVER也可以创建临时表。临时表与永久表相似，但临时表存储在 tempdb 中，当不再使用时会自动删除。
有本地和全局两种类型的临时表，二者在名称、可见性和可用性上均不相同。本地临时表的名称以单个数字符号 (#) 打头；它们仅对当前的用户连接是可见的；当用户从 Microsoft SQL Server 2000 实例断开连接时被删除。全局临时表的名称以数学符号 (##) 打头，创建后对任何用户都是可见的，当所有引用该表的用户从 SQL Server 断开连接时被删除。
例如，如果创建名为 employees 的表，则任何人只要在数据库中有使用该表的安全权限就可以使用该表，除非它已删除。如果创建名为 #employees 的本地临时表，只有您能对该表执行操作且在断开连接时该表删除。如果创建名为 ##employees 的全局临时表，数据表中的任何用户均可对该表执行操作。如果该表在您创建后没有其他用户使用，则当您断开连接时该表删除。如果该表在您创建后有其他用户使用，则 SQL Server在所有用户断开连接后删除该表。


oracle临时表与sqlserver临时表的不同:
1. SQL SERVER临时表是一种”内存表”,表是存储在内存中的.ORACLE临时表除非执行DROP TABLE,否则表定义会保留在数据字典中.
2. SQL SERVER临时表不存在类似ORACLE临时表 事务级别 上的功能.
3. SQL SERVER本地临时表(#) 与 ORACLE的会话级别临时表类似,但是在会话退出的时候,ORACLE不会删除表.
4. SQL SERVER的全局临时表(##) 是指多个连接共享同一片内存.当没有指针引用该内存区域时,SQL SERVER自动释放全局临时表.
5. 由于ORACLE不是一种 内存中的数据库. 所以如果ORACLE类似SQL SERVER 频繁的对临时表进行建立和删除,必定会影响性能.所以ORACLE会保留临时表的定义直到用户DROP TABLE.
6. 在ORACLE中,如果需要多个用户共享一个表(类似SQL SERVER的全局临时表##).则可以利用永久表,并且在表中添加一些可以唯一标识用户的列.利用触发器和视图.当用户退出的时候,根据该登陆用户的唯一信息删除相应的表中的数据. 这种方法给ORACLE带来了一定量的负载. 

----

#### 创建表空间和分配权限

> [oracle-创建表空间和分配权限-011](http://hi.baidu.com/lwyfly/item/52766e1085a830affeded59a)

##### 1. 表空间创建
###### 1.1 查看表空间（sys用户下）
```sql
select * from v$tablespace;
```
###### 1.2 创建表空间

ORACLE可以创建的表空间有三种类型:
1. TEMPORARY: 临时表空间,用于临时数据的存放;
2. UNDO : 还原表空间. 用于存入重做日志文件.
3. 用户表空间: 最重要,也是用于存放用户数据表空间
`TEMPORARY` 和 `UNDO` 表空间是ORACLE 管理的特殊的表空间.只用于存放系统相关数据.
```sql
//如果有重名的表空间会被覆盖。
create tablespace mytablespace datafile 'e:\dbf\mytablespace.dbf' size 5m
autoextend on next  100k maxsize 10m
```
创建表空间mytablespace 存放在e:\dbf 创建文件mytablespace.dbf 自动扩展增量100k最大文件10m

###### 1.3 添加表空间的文件
```sql
ALTER TABLESPACE "MYTABLESPACE" ADD DATAFILE 'E:\DBF\myts02' SIZE 10M AUTOEXTEND ON NEXT 100K MAXSIZE 15M
```
表空间mytablespace 增加添加文件myts02.dbf大小为10m 数据文件已满后自动扩展增量100kb，最大文件大小为15m

##### 2. 创建用户并分配表空间
查看用户
```sql
select * from dba_users;//所有用户
select * from all_users;//所有用户
select * from user_users;//仅自己的
create user fly identified by fly;//创建用户fly,密码fly
alter user fly default tablespace mytablespace;//用户分配的表空间mytablespace
```

##### 3. 分配权限
```sql
grant resource to fly;//给用户资源权限
grant connect to fly;//给用户连接权限
grant create session to fly;//在mytablespace上分配给fly用户使用
grant create table to fly with admin option; //给fly登录权限
grant dba to fly;// dba将系统权限分配给fly
```
Oracle所有的系统权限

alter cluster 修改拥有簇的权限
alter database 修改数据库的权限
alter procedure 修改拥有的存储过程权限
alter profile 修改资源限制简表的权限
alter resource cost 设置佳话资源开销的权限
alter rollback segment 修改回滚段的权限
alter sequence 修改拥有的序列权限
alter session 修改数据库会话的权限
alter sytem 修改数据库服务器设置的权限
alter table 修改拥有的表权限
alter tablespace 修改表空间的权限
alter user 修改用户的权限
analyze 使用analyze命令分析数据库中任意的表、索引和簇
audit any 为任意的数据库对象设置审计选项
audit system 允许系统操作审计
backup any table 备份任意表的权限
become user 切换用户状态的权限
commit any table 提交表的权限
create cluster 为用户创建簇的权限
create database link 为用户创建的权限
create procedure 为用户创建存储过程的权限
create profile 创建资源限制简表的权限
create public database link 创建公共数据库链路的权限
create public synonym 创建公共同义名的权限
create role 创建角色的权限
create rollback segment 创建回滚段的权限
create session 创建会话的权限
create sequence 为用户创建序列的权限
create snapshot 为用户创建快照的权限
create synonym 为用户创建同义名的权限
create table 为用户创建表的权限
create tablespace 创建表空间的权限
create user 创建用户的权限
create view 为用户创建视图的权限
delete snapshot 删除快照中行的权限
delete table 为用户删除表行的权限
delete view 为用户删除视图行的权限
drop profile 删除资源限制简表的权限
drop public cluster 删除公共簇的权限
drop public database link 删除公共数据链路的权限
drop public synonym 删除公共同义名的限
drop rollback segment 删除回滚段的权限
drop tablespace 删除表空间的权限
drop user 删除用户的权限
execute any procedure 执行任意存储过程的权限
execute function 执行存储函数的权限
execute package 执行存储包的权限
execute procedure 执行用户存储过程的权限
force any transaction 管理未提交的任意事务的输出权限
force transaction 管理未提交的用户事务的输出权限
grant any privilege 授予任意系统特权的权限
grant any role 授予任意角色的权限
index table 给表加索引的权限
insert any table 向任意表中插入行的权限
insert snapshot 向快照中插入行的权限
insert table 向用户表中插入行的权限
insert view 向用户视图中插行的权限
lock any table 给任意表加锁的权限
manager tablespace 管理（备份可用性）表空间的权限
references table 参考表的权限
restricted session 创建有限制的数据库会话的权限
select any sequence 使用任意序列的权限
select any table 使用任意表的权限
select snapshot 使用快照的权限
select sequence 使用用户序列的权限
select table 使用用户表的权限
select view 使用视图的权限
unlimited tablespace 对表空间大小不加限制的权限
update any table 修改任意表中行的权限
update snapshot 修改快照中行的权限
update table 修改用户表中的行的权限
update view 修改视图中行的权限

----

###  Windows下启动停止Oracle11g服务

> [Windows下启动停止Oracle11g服务](http://hollenliu.blog.51cto.com/116553/280844)

启动Oracle 11g服务
```bash
@echo off
@ ECHO 启动 Oracle 11g 服务
net start "OracleDBConsoleorcl"
net start "OracleOraDb11g_home1TNSListener"
net start "OracleServiceORCL"
@ ECHO 启动完毕 按任意键继续
pause
exit
```

停止Oracle 11g服务
```bash
@echo off
@ ECHO 停止 Oracle 11g 服务
net stop "OracleDBConsoleorcl"
net stop "OracleOraDb11g_home1TNSListener"
net stop "OracleServiceORCL"
@ ECHO 停止完毕 按任意键继续
pause
exit 
```