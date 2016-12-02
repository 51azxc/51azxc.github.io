title: "hibernate其他知识点补遗"
date: 2015-05-13 15:21:59
tags: ["orm", "c3p0"]
categories: ["java", "hibernate"]
---

### c3p0配置

> [Hibernate整合C3P0实现连接池](http://www.cnblogs.com/best/archive/2013/05/09/3069839.html)
> [hibernate与c3p0](http://wanghuidong.iteye.com/blog/835861)
> [hibernate中c3p0的配置](http://blog.csdn.net/fdgaq/article/details/7570618)

```xml
<!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
<property name="acquireIncrement">3</property>
<!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->
<property name="acquireRetryAttempts">30</property>
<!--两次连接中间隔时间，单位毫秒。Default: 1000 -->
<property name="acquireRetryDelay">1000</property>
<!--连接关闭时默认将所有未提交的操作回滚。Default: false -->
<property name="autoCommitOnClose">false</property>
<!--c3p0将建一张名为Test的空表，并使用其自带的查询语句进行测试。
如果定义了这个参数那么属性preferredTestQuery将被忽略。
你不能在这张Test表上进行任何操作，它将只供c3p0测试使用。Default: null-->
<property name="automaticTestTable">Test</property>
<!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。
但是数据源仍有效保留，并在下次调用getConnection()的时候继续尝试获取连接。
如果设为true，那么在尝试获取连接失败后该数据源将申明已断开并永久关闭。Default: false-->
<property name="breakAfterAcquireFailure">false</property>
<!--当连接池用完时客户端调用getConnection()后等待获取新连接的时间，超时后将抛出
SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
<property name="checkoutTimeout">100</property>
<!--通过实现ConnectionTester或QueryConnectionTester的类来测试连接。类名需制定全路径。
Default: com.mchange.v2.c3p0.impl.DefaultConnectionTester-->
<property name="connectionTesterClassName"></property>
<!--指定c3p0 libraries的路径，如果（通常都是这样）在本地即可获得那么无需设置，默认null即可
Default: null-->
<property name="factoryClassLocation">null</property>
<!--Strongly disrecommended. Setting this to true may lead to subtle and bizarre bugs.
（文档原文）作者强烈建议不使用的一个属性-->
<property name="forceIgnoreUnresolvedTransactions">false</property>
<!--每60秒检查所有连接池中的空闲连接。Default: 0 -->
<property name="idleConnectionTestPeriod">60</property>
<!--初始化时获取三个连接，取值应在minPoolSize与maxPoolSize之间。Default: 3 -->
<property name="initialPoolSize">3</property>
<!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
<property name="maxIdleTime">60</property>
<!--连接池中保留的最大连接数。Default: 15 -->
<property name="maxPoolSize">15</property>
<!--JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。
但由于预缓存的statements属于单个connection而不是整个连接池。
所以设置这个参数需要考虑到多方面的因素。
如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0-->
<property name="maxStatements">100</property>
<!--maxStatementsPerConnection定义了连接池内单个连接所拥有的最大缓存statements数。Default: 0 -->
<property name="maxStatementsPerConnection"></property>
<!--c3p0是异步操作的，缓慢的JDBC操作通过帮助进程完成。扩展这些操作可以有效的提升性能
通过多线程实现多个操作同时被执行。Default: 3-->
<property name="numHelperThreads">3</property>
<!--当用户调用getConnection()时使root用户成为去获取连接的用户。主要用于连接池连接非c3p0
的数据源时。Default: null-->
<property name="overrideDefaultUser">root</property>
<!--与overrideDefaultUser参数对应使用的一个参数。Default: null-->
<property name="overrideDefaultPassword">password</property>
<!--密码。Default: null-->
<property name="password"></property>
<!--定义所有连接测试都执行的测试语句。在使用连接测试的情况下这个一显著提高测试速度。注意：
测试的表必须在初始数据源的时候就存在。Default: null-->
<property name="preferredTestQuery">select id from test where id=1</property>
<!--用户修改系统配置参数执行前最多等待300秒。Default: 300 -->
<property name="propertyCycle">300</property>
<!--因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的
时候都将校验其有效性。建议使用idleConnectionTestPeriod或automaticTestTable
等方法来提升连接测试的性能。Default: false -->
<property name="testConnectionOnCheckout">false</property>
<!--如果设为true那么在取得连接的同时将校验连接的有效性。Default: false -->
<property name="testConnectionOnCheckin">true</property>
<!--用户名。Default: null-->
<property name="user">root</property>
<!--连接池中保留的最小连接数。-->
<property name="minPoolSize" value="10" />
<!--连接池中保留的最大连接数。Default: 15 -->
<property name="maxPoolSize" value="100" />
<!--最大空闲时间,1800秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
<property name="maxIdleTime" value="1800" />
<!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
<property name="acquireIncrement" value="3" />
<property name="maxStatements" value="1000" />
<property name="initialPoolSize" value="10" />
<!--每60秒检查所有连接池中的空闲连接。Default: 0 -->
<property name="idleConnectionTestPeriod" value="60" />
<!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->
<property name="acquireRetryAttempts" value="30" />
```

----------

### hibernate 乐观锁

> [Hibernate 乐观锁（Optimistic Locking）](http://blog.sina.com.cn/s/blog_67c40caf0100jazk.html)

hibernate基于数据版本（Version）记录机制实现。为数据增加一个版本标识，一般是通过为数据库表增加一个“version”字段来实现。 读取出数据时，将此版本号一同读出，之后更新时，对此版本号加一。此时，将提交数据的版本数据与数据库表对应记录的当前版本信息进行比对，如果提交的数据 版本号大于数据库表当前版本号，则予以更新，否则认为是过期数据。
```java
@Entity  
public class Conductor {  
    @Id  
    @GeneratedValue  
    private Integer id;  

    private String name;  

    @Version  
    private Long version;
    
    //setter/getter
}

Session session1=openSession();  
Session session2=openSession();  
Conductor stu1=(Conductor)session1.createQuery("from Conductor as a where a.name='Bob'").uniqueResult();  
Conductor stu2=(Conductor)session2.createQuery("from Conductor as a where a.name='Bob'").uniqueResult();  
  
//这时候，两个版本号是相同的  
System.out.println("v1="+stu1.getVersion()+"--v2="+stu2.getVersion());  
  
Transaction tx1=session1.beginTransaction();  
stu1.setName("session1");  
tx1.commit();  
//这时候，两个版本号是不同的，其中一个的版本号递增了  
System.out.println("v1="+stu1.getVersion()+"--v2="+stu2.getVersion());  
  
Transaction tx2=session2.beginTransaction();  
stu2.setName("session2");  
  
tx2.rollback();  
session2.close();  
session1.close();
```

----------

### 注解实现联合主键

> [Hibernate注解映射联合主键的三种主要方式](http://blog.csdn.net/robinpipi/article/details/7655388)
> [hibernate注解方式实现复合主键](http://www.blogjava.net/relax/archive/2009/09/18/295587.html)

 联合主键用Hibernate注解映射方式主要有三种：
第一、将联合主键的字段单独放在一个类中，该类需要实现java.io.Serializable接口并重写equals和hascode，再将该类注解为`@Embeddable`,最后在主类中(该类不包含联合主键类中的字段)保存该联合主键类的一个引用，并生成set和get方法，并将该引用注解为`@Id`
```java
@Entity  
@Table(name="JLEE01")  
public class Jlee01 implements Serializable{
    private String address ;  
    private int age ;  
    private String email ;  
    private String phone ;  
@Id
    private JleeKey01 jleeKey ;
```
主键类
```java
@Embeddable  
public class JleeKey01 implements Serializable{  
    private long id;
    private String name;
    
    //setter/getter
  
    @Override  
    public boolean equals(Object o) {  
        if(o instanceof JleeKey01){  
            JleeKey01 key = (JleeKey01)o ;  
            if(this.id == key.getId() && this.name.equals(key.getName())){  
                return true ;  
            }  
        }  
        return false ;  
    }  
      
    @Override  
    public int hashCode() {  
        return this.name.hashCode();  
    }
}  
```

第二、将联合主键的字段单独放在一个类中，该类需要实现java.io.Serializable接口并重写equals和hascode，最后在主类中(该类不包含联合主键类中的字段)保存该联合主键类的一个引用，并生成set和get方法，并将该引用注解为`@EmbeddedId`
```java
@Entity  
@Table(name="JLEE02")  
public class Jlee02 {  
  
    private String address ;  
    private int age ;  
    private String email ;  
    private String phone ;  
    @EmbeddedId
    private JleeKey02 jleeKey ;
// 主键类：JleeKey02.java为普通java类即可。
```

第三、将联合主键的字段单独放在一个类中，该类需要实现java.io.Serializable接口并要重写equals和hashcode.最后在主类中(该类包含联合主键类中的字段)将联合主键字段都注解为`@Id`,并在该类上方将上这样的注解：`@IdClass`(联合主键类.class)
```java
@Entity  
@Table(name="JLEE03")  
@IdClass(JleeKey03.class)  
public class Jlee03 {  
    @Id
    private long id ;
    @Id  
    private String name ;
```

----------

### hibernate.current_session_context_class

> [hibernate.current_session_context_class](http://justsee.iteye.com/blog/1061576)

从3.0.1版本开 始，Hibernate增加了`SessionFactory.getCurrentSession()`方法。一开始，它假定了采用JTA事务，JTA事务 定义了当前session的范围和上下文(scope and context)。Hibernate开发团队坚信，因为有好几个独立的JTA TransactionManager实现稳定可用，不论是否被部署到一个J2EE容器中，大多数(假若不是所有的）应用程序都应该采用JTA事务管理。 基于这一点，采用JTA的上下文相关session可以满足你一切需要。 

更好的是，从3.1开始，`SessionFactory.getCurrentSession()`的后台实现是可拔插的。因此，我们引入了新的扩展接口(org.hibernate.context.CurrentSessionContext)和新的配置参数(`hibernate.current_session_context_class`)，以便对什么是“当前session”的范围和上下文(scope and context)的定义进行拔插。

请参阅org.hibernate.context.CurrentSessionContext接口的Javadoc,那里有关于它的契约的详细讨论。它定义了单一的方法，`currentSession()`，特定的实现用它来负责跟踪当前的上下文session。Hibernate内置了此接口的三种实现。

`org.hibernate.context.JTASessionContext` - 当前session根据JTA来跟踪和界定。这和以前的仅支持JTA的方法是完全一样的。详情请参阅Javadoc。

`org.hibernate.context.ThreadLocalSessionContext` - 当前session通过当前执行的线程来跟踪和界定。详情也请参阅Javadoc。

`org.hibernate.context.ManagedSessionContext` - 当前session通过当前执行的线程来跟踪和界定。但是，你需要负责使用这个类的静态方法将Session实例绑定、或者取消绑定，它并不会打开(open)、flush或者关闭(close)任何Session。

前两种实现都提供了“每数据库事务对应一个session”的编程模型，也称作每次请求一个session。Hibernate session的起始和终结由数据库事务的生存来控制。假若你在纯粹的 Java SE之上采用自行编写代码来管理事务,而不使用JTA，建议你使用Hibernate Transaction API来把底层事务实现从你的代码中隐藏掉。如果你使用JTA，请使用JTA借口来管理Transaction。如果你在支持CMT的EJB容器中执行代码，事务边界是声明式定义的，你不需要在代码中进行任何事务或session管理操作。

`hibernate.current_session_context_class`配置参数定义了应该采用哪个org.hibernate.context.CurrentSessionContext实现。注意，为了向下兼容，如果未配置此参数，但是存在org.hibernate.transaction.TransactionManagerLookup的配置，Hibernate会采用org.hibernate.context.JTASessionContext。一般而言，此参数的值指明了要使用的实现类的全名，但那三种内置的实现可以使用简写，即**jta**、**thread**和**managed**。

 

1、`getCurrentSession()`与`openSession()`的区别？

在 SessionFactory 启动的时候， Hibernate 会根据配置创建相应的 CurrentSessionContext ，在 `getCurrentSession()` 被调用的时候，实际被执行的方法是`CurrentSessionContext.currentSession()` 。在 `currentSession()` 执行时，如果当前 Session 为空，currentSession 会调用SessionFactory的openSession 。所以`getCurrentSession()` 对于 Java EE 来说是更好的获取 Session 的方法。

* 采用`getCurrentSession()`创建的session会绑定到当前线程中，而采用`openSession()`创建的session则不会
* 采用`getCurrentSession()`创建的session在commit或rollback时会自动关闭，而采用`openSession()`创建的session必须手动关闭

2、使用`getCurrentSession()`需要在hibernate.cfg.xml文件中加入如下配置：

* 如果使用的是本地事务（jdbc事务）
```xml
<property name="hibernate.current_session_context_class">thread</property>
```
* 如果使用的是全局事务（jta事务）
```xml
<property name="hibernate.current_session_context_class">jta</property>
```
* 如果使用的是session的管理机制（不太常用）
```xml
<property name="hibernate.current_session_context_class">managed</property>
```
