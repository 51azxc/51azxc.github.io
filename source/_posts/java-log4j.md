title: "log4j配置"
date: 2015-04-27 18:01:19
tags: ["log","Java"]
categories: "Java"
---

> [Log4j使用指南](http://www.cnblogs.com/licheng/archive/2008/08/23/1274566.html)
> [如何使用Log4j？](http://www.blogjava.net/rickhunter/articles/28133.html)
> [玩转log4j](http://www.cnblogs.com/shenliang123/archive/2012/05/02/2479286.html)
> [Tomcat下log4j设置文件路径和temp目录](http://www.cnblogs.com/dkblog/archive/2007/07/27/1980873.html)
> [log4j输出多个自定义日志文件](http://wenku.baidu.com/link?url=LTbE_lIz7Myn7GZtJ9PQlZJl_mHyYGJoVu7BSAIMy0eVKbGf4WL-IibBPJb0j0Sf183sf3A2o08Nao2pddGHTk3r5Oq-VnQhgQw_tuGLQxu)

log4j的配置文件支持key=value格式的properties文件以及xml文件。

#### 日志优先级
它的日志优先级别有: **OFF**、**FATAL**、**ERROR**、**WARN**、**INFO**、**DEBUG**、**ALL**或者自定义级别。一般使用的从低到高为**DEBUG**、**INFO**、**WARN**、**ERROR**.假如在一个级别为q的Logger中发生一个级别为p的日志请求，如果p>=q,那么请求将被启用。这是Log4j的核心原则。 比如在这里定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来。
#### 输出源
一个输出源被称做一个**Appender**。 **Appender**包括console（控制台）, files（文件）, GUI components（图形的组件）, remote socket servers（socket 服务）, JMS（java信息服务）, NT Event Loggers（NT的事件日志）, and remote UNIX Syslog daemons（远程UNIX的后台日志服务）。它也可以做到异步记录。
一个logger可以设置超过一个的appender。 用addAppender 方法添加一个appender到一个给定的logger。对于一个给定的logger它每个生效的日志请求都被转发到该logger所有的appender上和该logger的父辈logger的appender上。
Log4j提供的appender有以下几种：

**org.apache.log4j.ConsoleAppender**（控制台）
**org.apache.log4j.FileAppender**（文件）
**org.apache.log4j.DailyRollingFileAppender**（每天产生一个日志文件）
**org.apache.log4j.RollingFileAppender**（文件大小到达指定尺寸的时候产生新文件）
**org.apache.log4j.WriterAppender**（将日志信息以流格式发送到任意指定的地方）

#### 样式设置
##### 布局样式
**org.apache.log4j.HTMLLayout**（以HTML表格形式布局），
**org.apache.log4j.PatternLayout**（可以灵活地指定布局模式），
**org.apache.log4j.SimpleLayout**（包含日志信息的级别和信息字符串），
**org.apache.log4j.TTCCLayout**（包含日志产生的时间、线程、类别等等信息）

##### 格式
%m 输出代码中指定的消息
%p 输出优先级，即`DEBUG`，`INFO`，`WARN`，`ERROR`，`FATAL`
%r 输出自应用启动到输出该log信息耗费的毫秒数
%c 输出所属的类目，通常就是所在类的全名
%t 输出产生该日志事件的线程名
%n 输出一个回车换行符，Windows平台为"rn"，Unix平台为"n"
%d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921
%l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(Test Log4.java:10) 

示例:
```ini
# log4j.rootLogger=INFO,stdout
# 指定多个输出源
log4j.logger.com.framework=INFO,stdout,R

# 指定控制台输出
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
# 输出级别为INFO
log4j.appender.stdout.Threshold=INFO
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[%p] %d{yyyy-MM-dd HH:mm:ss} %m%n

# 指定输出日志到文件
log4j.appender.R=org.apache.log4j.RollingFileAppender
# 指定文件目录，这里为tomcat主目录,需要在环境变量中配置
log4j.appender.R.File=${catalina.base}/logs/ilog.txt
# 大于20MB则另起文件
log4j.appender.R.MaxFileSize=20MB
log4j.appender.R.Threshold=INFO
log4j.appender.R.MaxBackupIndex=1
log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=[%p] %d{yyyy-MM-dd HH:mm:ss} %m%n
```

在代码中调用日志:
```java
public class LogTest {
  Logger log = Logger.getLogger(LogTest.class);
   public static void main(String[] args) {
     log.info("test");
   }
}
```

