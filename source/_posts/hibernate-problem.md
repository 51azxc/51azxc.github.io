title: "Hibernate 填坑"
date: 2015-04-21 22:12:32
tags: ["orm"]
categories: ["java", "hibernate"]
---

### LOB creation as createClob() method threw error

> [Disabling contextual LOB creation as createClob() method threw error : java.lang.reflect.InvocationTargetException](http://hi.baidu.com/forloop/item/ed2e29b077f1a6f063388e52)

如果运行时出现下面提示的话：
```
INFO [pool-2-thread-1] - HHH000400: Using dialect: org.hibernate.dialect.MySQL5Dialect
INFO [pool-2-thread-1] - HHH000424: Disabling contextual LOB creation as createClob() method threw error : java.lang.reflect.InvocationTargetException
INFO [pool-2-thread-1] - HHH000268: Transaction strategy: org.hibernate.engine.transaction.internal.jdbc.JdbcTransactionFactory
```

可以忽略也可以将驱动降级，例如`Oracle`,将class15替换成class14，`Mysql`，将mysql-connector-java-5.1.21.jar 替换成 mysql-connector-java-5.1.6.jar,以此类推

----

### createSQLQuery is not valid without active transaction

> [如果你报createSQLQuery is not valid without active transaction,请看这里 ](http://blog.csdn.net/yinjian520/article/details/8666695)


使用 Hibernate 的大多数应用程序需要某种形式的“上下文相关的”会话，特定的会话在整个特定的上下文范围内始终有效。然而，对不同类型的应用程序而言，要为什么是组成这种“上下文”下一个定义通常是困难的；不同的上下文对“当前”这个概念定义了不同的范围。
在 3.0 版本之前，使用 Hibernate 的程序要么采用自行编写的基于 `ThreadLocal` 的上下文会话，要么采用HibernateUtil 这样的辅助类，要么采用第三方框架（比如 Spring 或 Pico），它们提供了基于代理（proxy）或者基于拦截器（interception）的上下文相关的会话。从 3.0.1 版本开始，Hibernate 增加了SessionFactory.getCurrentSession() 方法。一开始，它假定了采用 JTA 事务，JTA 事务定义了当前 session 的范围和上下文（scope 和 context）。因为有好几个独立的 JTA TransactionManager 实现稳定可用，不论是否被部署到一个 J2EE 容器中，大多数（假若不是所有的）应用程序都应该采用 JTA 事务管理。基于这一点，采用 JTA 的上下文相关的会话可以满足你一切需要。

hibernate 增加配置
```xml
<property name="hibernate.current_session_context_class">thread</property> 
```
将`getCurrentSession()`返回的session绑定到当前运行线程中。


