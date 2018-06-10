title: "Spring xml配置"
date: 2015-05-15 11:29:03
tags: ["Spring", "xml"]
categories:  ["Java", "Spring"]
---

### Spring使用Annotation时需要注意的地方

> [Spring使用Annotation时需要注意的地方](http://www.blogjava.net/fengzhisha0914/articles/343648.html)

使用Annotation注解形式时,在Spring的配置文件中需要加入新的xsd文件引用:

<!-- more -->

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--错误形式
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/context/spring-context-2.5.xsd
    http://www.springframework.org/schema/context">
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
这三句的顺序不能自己随便写.....必须按照下面的顺序写.否则会抛出一个异常-->

<!--正确形式-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
      http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context-2.5.xsd">

    <context:annotation-config />
    <bean id="userDAO" class="com.dao.impl.UserDAOImpl" />
    <bean id="userService" class="com.service.UserService">
       <!--<property name="userDAO" ref="userDAO"></property>-->
    </bean>

</beans>
```
异常
```
org.springframework.beans.factory.xml.XmlBeanDefinitionStoreException: Line 17 in XML document from class path resource [applicationContext.xml] is invalid; nested exception is org.xml.sax.SAXParseException: cvc-complex-type.2.4.c: The matching wildcard is strict, but no declaration can be found for element 'context:annotation-config'.
```

----------

### Spring Hibernate SessionFactory packagesToScan Bug

> [Spring Hibernate SessionFactory packagesToScan Bug](http://myrev.iteye.com/blog/493527)

如果配置了`<property name="packagesToScan" value="com.entity.*" /> `属性，需要扫描的实体类需要在指定包下的下一层位置，不能直接放在`com.entity.`中，或者不指定最后的`*`号

----------

### Spring depends-on

> [Spring depends-on](http://blog.163.com/tangyang_personal/blog/static/4622961320083272317605/)

`depend-on`用来表示一个Bean的实例化依靠另一个Bean先实例化。如果在一个bean A上定义了depend-on B那么就表示：A 实例化前先实例化 B。
这种情况下，A可能根本不需要持有一个B对象.
Spring允许Bean和Bean依赖的Bean（合作者）上同时定义`depends-on`。比如`A depends-on B && B depends-on C && C depends-on D`。
但是Spring不允许`A depends-on B && B depends-on A`的情况。看下面的例子，由于D又依赖回A，这种在依赖关系中形成了一个闭环，Spring将无法处理这种依赖关系。
一个Bean可以同时`depends-on`多个对象如，`A depends-on D,C,B`。可以使用“，”或“；”定义多个depends-on的对象

----------

### Spring中bean的scope详解

> [Spring中bean的scope详解](http://blog.csdn.net/fhx007/article/details/7016694)

`scope`就是用来配置spring bean的作用域，它标识bean的作用域

**singleton作用域(scope 默认值)**
当一个bean的作用域设置为`singleton`, 那么Spring IOC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。换言之，当把一个bean定义设置为`singleton`作用域时，Spring IOC容器只会创建该bean定义的唯一实例。这个单一实例会被存储到单例缓存（singleton cache）中，并且所有针对该bean的后续请求和引用都将返回被缓存的对象实例，这里要注意的是`singleton`作用域和GOF设计模式中的单例是完全不同的，单例设计模式表示一个ClassLoader中 只有一个class存在，而这里的`singleton`则表示一个容器对应一个bean，也就是说当一个bean被标识为`singleton`时候，spring的IOC容器中只会存在一个该bean。
```xml
<bean id="class" class="class" scope="singleton"/>
<!-- or -->
<bean id="class" class="class" singleton="true"/>
```

**prototype**
`prototype`作用域部署的bean，每一次请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）都会产生一个新的bean实例，相当与一个new的操作，对于prototype作用域的bean，有一点非常重要，那就是Spring不能对一个prototype bean的整个生命周期负责，容器在初始化、配置、装饰或者是装配完一个prototype实例后，将它交给客户端，随后就对该prototype实例不闻不问了。不管何种作用域，容器都会调用所有对象的初始化生命周期回调方法，而对prototype而言，任何配置好的析构生命周期回调方法都将不会被调用。 清除prototype作用域的对象并释放任何prototype bean所持有的昂贵资源，都是客户端代码的职责。（让Spring容器释放被singleton作用域bean占用资源的一种可行方式是，通过使用 bean的后置处理器，该处理器持有要被清除的bean的引用。）
```xml
<bean id="class" class="class" scope="prototype"/>
<!-- or -->
<bean id="class" class="class" singleton="false"/>
```

**request**
`request`表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前`HTTP request`内有效
`request`、`session`、`global session`使用的时候首先要在初始化web的web.xml中做如下配置：
如果你使用的是Servlet 2.4及以上的web容器，那么你仅需要在web应用的XML声明文件web.xml中增加下述`ContextListener`即可：
```xml
<web-app>
   ...
  <listener>
    <listener-class>org.springframework.web.context.request.RequestContextListener     </listener-class>
  </listener>
   ...
</web-app>
```
接着既可以配置bean的作用域了
```xml
<bean id="class" class="class" scope="request"/>
```

**session**
`session`作用域表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前`HTTP session`内有效

**global session**
``global session``作用域类似于标准的HTTP Session作用域，不过它仅仅在基于portlet的web应用中才有意义。Portlet规范定义了全局Session的概念，它被所有构成某个portlet web应用的各种不同的portlet所共享。在global session作用域中定义的bean被限定于全局portlet Session的生命周期范围内。如果你在web中使用global session作用域来标识bean，那么web会自动当成session类型来使用

----------

### `<context:component-scan>`使用说明

> [`<context:component-scan>`使用说明](http://blog.csdn.net/chunqiuwei/article/details/16115135)

在xml配置了这个标签后，spring可以自动去扫描base-pack下面或者子包下面的java文件，如果扫描到有`@Component`,`@Controller`,`@Service`等这些注解的类，则把这些类注册为bean

**注意**：如果配置了`<context:component-scan>`那么`<context:annotation-config/>`标签就可以不用再xml中配置了，因为前者包含了后者。另外`<context:annotation-config/`>还提供了两个子标签`<context:include-filter>`,`<context:exclude-filter>`。默认的`<context:component-scan>`标签有个属性`use-default-filters`默认为`true`，必须指定为`false`上面两个标签才有用，`<context:include-filter>`意为需要包含哪些被扫描的类，`<context:exclude-filter>`表示需要被排除的哪些类。

----------

### 配置Spring数据源

> [配置Spring数据源](http://yonguo.iteye.com/blog/115221)

**DBCP数据源**
```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"       
        destroy-method="close">       
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />      
    <property name="url" value="jdbc:mysql://localhost:3309/sampledb" />      
    <property name="username" value="root" />      
    <property name="password" value="1234" />      
</bean>
```
BasicDataSource提供了`close()`方法关闭数据源，所以必须设定`destroy-method=”close”`属性，以便Spring容器关闭时，数据源能够正常关闭。除以上必须的数据源属性外，还有一些常用的属性：

* `defaultAutoCommit`：设置从数据源中返回的连接是否采用自动提交机制，默认值为 `true`;
* `defaultReadOnly`：设置数据源是否仅能执行只读操作， 默认值为 `false`；
* `maxActive`：最大连接数据库连接数，设置为0时，表示没有限制；
* `maxIdle`：最大等待连接中的数量，设置为0时，表示没有限制；
* `maxWait`：最大等待秒数，单位为毫秒， 超过时间会报出错误信息；
* `validationQuery`：用于验证连接是否成功的查询SQL语句，SQL语句必须至少要返回一行数据， 如你可以简单地设置为：“select count(*) from user”；
* `removeAbandoned`：是否自我中断，默认是`false`；
* `removeAbandonedTimeout`：几秒后数据连接会自动断开，在`removeAbandoned`为`true`，提供该值；
* `logAbandoned`：是否记录中断事件， 默认为 `false`；

**C3P0数据源**
```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"       
        destroy-method="close">      
    <property name="driverClass" value=" oracle.jdbc.driver.OracleDriver "/>
    <property name="jdbcUrl" value=" jdbc:oracle:thin:@localhost:1521:ora9i "/>
    
    <property name="user" value="admin"/>      
    <property name="password" value="1234"/>      
</bean>
```
ComboPooledDataSource和BasicDataSource一样提供了一个用于关闭数据源的close()方法，这样我们就可以保证Spring容器关闭时数据源能够成功释放。
C3P0拥有比DBCP更丰富的配置属性，通过这些属性，可以对数据源进行各种有效的控制：

* `acquireIncrement`：当连接池中的连接用完时，C3P0一次性创建新连接的数目；
* `acquireRetryAttempts`：定义在从数据库获取新连接失败后重复尝试获取的次数，默认为30；
* `acquireRetryDelay`：两次连接中间隔时间，单位毫秒，默认为1000；
* `autoCommitOnClose`：连接关闭时默认将所有未提交的操作回滚。默认为false；
* `automaticTestTable`： C3P0将建一张名为Test的空表，并使用其自带的查询语句进行测试。如果定义了这个参数，那么属性preferredTestQuery将被忽略。你 不能在这张Test表上进行任何操作，它将中为C3P0测试所用，默认为null；
* `breakAfterAcquireFailure`：获取连接失败将会引起所有等待获取连接的线程抛出异常。但是数据源仍有效保留，并在下次调用`getConnection(`)的时候继续尝试获取连接。如果设为`true`，那么在尝试获取连接失败后该数据源将申明已断开并永久关闭。默认为`false`；
* `checkoutTimeout`：当连接池用完时客户端调用`getConnection()`后等待获取新连接的时间，超时后将抛出`SQLException`，如设为0则无限期等待。单位毫秒，默认为0
* `connectionTesterClassName`： 通过实现ConnectionTester或QueryConnectionTester的类来测试连接，类名需设置为全限定名。默认为 com.mchange.v2.C3P0.impl.DefaultConnectionTester；
* `idleConnectionTestPeriod`：隔多少秒检查所有连接池中的空闲连接，默认为0表示不检查；
* `initialPoolSize`：初始化时创建的连接数，应在minPoolSize与maxPoolSize之间取值。默认为3；
* `maxIdleTime`：最大空闲时间，超过空闲时间的连接将被丢弃。为0或负数则永不丢弃。默认为0；
* `maxPoolSize`：连接池中保留的最大连接数。默认为15；
* `maxStatements`：JDBC的标准参数，用以控制数据源内加载的PreparedStatement数量。但由于预缓存的Statement属 于单个Connection而不是整个连接池。所以设置这个参数需要考虑到多方面的因素，如果maxStatements与 maxStatementsPerConnection均为0，则缓存被关闭。默认为0；
* `maxStatementsPerConnection`：连接池内单个连接所拥有的最大缓存Statement数。默认为0；
* `numHelperThreads`：C3P0是异步操作的，缓慢的JDBC操作通过帮助进程完成。扩展这些操作可以有效的提升性能，通过多线程实现多个操作同时被执行。默认为3；
* `preferredTestQuery`：定义所有连接测试都执行的测试语句。在使用连接测试的情况下这个参数能显著提高测试速度。测试的表必须在初始数据源的时候就存在。默认为null；
* `propertyCycle`： 用户修改系统配置参数执行前最多等待的秒数。默认为300；
* `testConnectionOnCheckout`：因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的时候都将校验其有效性。建议使用`idleConnectionTestPeriod`或`automaticTestTable`等方法来提升连接测试的性能。默认为false；
* `testConnectionOnCheckin`：如果设为true那么在取得连接的同时将校验连接的有效性。默认为false.

读配置文件的方式引用属性：
```xml
<bean id="propertyConfigurer"     
        class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">      
    <property name="location" value="/WEB-INF/jdbc.properties"/>      
</bean>      
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"       
        destroy-method="close">      
    <property name="driverClassName" value="${jdbc.driverClassName}" />      
    <property name="url" value="${jdbc.url}" />      
    <property name="username" value="${jdbc.username}" />      
    <property name="password" value="${jdbc.password}" />      
</bean>
```
在jdbc.properties属性文件中定义属性值：
```
jdbc.driverClassName= com.mysql.jdbc.Driver
jdbc.url= jdbc:mysql://localhost:3309/sampledb
jdbc.username=root
jdbc.password=1234 
```

**获取JNDI数据源**
```xml
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
    <property name="jndiName" value="java:comp/env/jdbc/bbt"/>      
</bean>
```
Spring 2.0为获取J2EE资源提供了一个jee命名空间，通过jee命名空间，可以有效地简化J2EE资源的引用。下面是使用jee命名空间引用JNDI数据源的配置：
```xml
<beans xmlns=http://www.springframework.org/schema/beans    
xmlns:xsi=http://www.w3.org/2001/XMLSchema-instance    
xmlns:jee=http://www.springframework.org/schema/jee    
xsi:schemaLocation="http://www.springframework.org/schema/beans     
http://www.springframework.org/schema/beans/spring-beans-2.0.xsd     
http://www.springframework.org/schema/jee    
http://www.springframework.org/schema/jee/spring-jee-2.0.xsd">      
    <jee:jndi-lookup id="dataSource" jndi-name=" java:comp/env/jdbc/bbt"/>      
</beans>
```
