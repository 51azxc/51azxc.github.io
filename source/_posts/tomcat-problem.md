title: "tomcat部分问题"
date: 2015-04-20 23:53:39
tags: "tomcat"
catgories: ["java","tomcat"]
---

### Tomcat出现processWorkerExit问题

> [eclipse中tomcat自动部署时自动停止问题processWorkerExit](http://hi.baidu.com/hellodragon109/item/bdd485e981f2c1058d3ea8b7)

1、问题描述
  在eclipse或者集成eclipse的其他开发工具中，在tomcat中部署了项目debug模式启动项目，项目启动之后修改项目java源代码，eclipse会自动部署项目到tomcat中。但在tomcat自动重启时会自动停止到`processWorkerExit(w, completedAbruptly);`这一行代码上。
  
2、问题出现原因
  原因是因为在`java.util.concurrent.ThreadPoolExecutor`类中的`runWorker(Worker w)`方法上有未捕获的异常信息
3、解决方案
  去掉java->debug->suspend execution on uncaught exceptions 选项钱的对勾就行了
  
----

### Tomcat JNDI连接池

> [Tomcat 7.0 JNDI连接池配置](http://blog.csdn.net/jokes000/article/details/7463345)
> [tomcat下jndi的三种配置方式](http://blog.csdn.net/lgm277531070/article/details/6711177)

将所用数据库的jdbc驱动包放到tomcat目录下的`lib`文件夹内

#### 全局配置
在tomcat目录下的conf文件夹内找到`context.xml`，在`Context`节点下添加如下代码
```xml
<Resource name="jdbc/test"   
          auth="Container"   
          type="javax.sql.DataSource"   
          driverClassName="com.mysql.jdbc.Driver"   
          url="jdbc:mysql://localhost:3306/test"   
          username="root"   
          password="root"   
          maxActive="20"   
          maxIdle="10"   
          maxWait="10000"/>
```

然后在WEB项目中中的`web.xml`中添加如下代码:
```xml
<resource-ref>  
  <description>JNDI DataSource</description>
  <!-- 节点名字要与Resource name一致 -->
  <res-ref-name>jdbc/test</res-ref-name>  
  <res-ref-type>javax.sql.DataSource</res-ref-type>  
  <res-auth>Container</res-auth>  
</resource-ref>
```
可以编写一个类测试一下
```java
public void test() throws NamingException, SQLException{
    Context ctx = new InitialContext();
    DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/test");
    Connection conn = ds.getConnection();
    if(conn!=null){
      System.out.println("success");
    }
}
```

#### 局部配置
在WEB项目中的`META-INF`文件下新建`context.xml`文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context>
    <Resource auth="Container" 
              driverClassName="com.mysql.jdbc.Driver"
              maxActive="100"
              maxIdle="40"
              maxWait="12000"
              name="jdbc/test"
              username="root"
              password="root"
              type="javax.sql.DataSource"
              url="jdbc:mysql://localhost:3306/BookDB?characterEncoding=UTF-8" />
</Context>
```
接着编辑`web.xml`与**全局配置**中的一样

----

### 同时启动多个Tomcat

> [两个一样的tomcat不能同时启动解决方法](http://blog.csdn.net/newizan/article/details/37343205)

需要使用免安装版的压缩包

1. 在系统的环境变量中增加`CATALINA_HOME2`及`CATALINA_BASE2`，值为新的Tomcat的目录。
2. 修改新的tomcat中的startup.bat，把其中的`CATALINA_HOME`改为`CATALINA_HOME2`
3. 修改新的tomcat中的catalina.bat，把其中的`CATALINA_HOME`改为`CATALINA_HOME2`，`CATALINA_BASE`改为`CATALINA_BASE2`
4. 修改conf/server.xml文件
```xml
<!-- 把端口改为没有是使用的端口 -->
<Server port="8005" shutdown="SHUTDOWN">
<!-- 把端口改为没有是使用的端口 -->
<Connector port="8080" maxHttpHeaderSize="8192" maxThreads="150" minSpareThreads="25" maxSpareThreads="75" enableLookups="false" redirectPort="8443" acceptCount="100" connectionTimeout="20000" disableUploadTimeout="true" />
<!-- 把端口改为没有是使用的端口 -->
<Connector port="8009" enableLookups="false" redirectPort="8443" protocol="AJP/1.3" />
```
至此如无意外已经可以同时启动两台Tomcat了，如果需要更多台，则按上述步骤继续添加新的环境变量以及修改端口。

----

### 修改 tomcat 内存

> [修改 tomcat 内存](http://www.cnblogs.com/quietwalk/archive/2012/11/05/2755199.html)

在Jetty 的VM参数中设置：
```bash
-Xms256m -Xmx512m -XX:MaxNewSize=256m -XX:MaxPermSize=256m
```

在tomcat运行环境中设置：
window环境 startup.bat第一行
```bash
SET CATALINA_OPTS= -Xms256m -Xmx512m -XX:MaxNewSize=256m -XX:MaxPermSize=256m 
```
修改catalina.bat文件,只需要在文件的头部加上`set JAVA_OPTS=-Xms512m -Xmx512m -Xss1024k`，数值分别对应了初始化的**最小内存**，**最大内存**，**线程内存大小**。如果JDK的版本是5.0之后的，线程内存可以不用设置。

对于容器下运行了多个WEB应用时，尽量将相同的JAR包转移到TOMCAT的lib下，此外还需要在`JAVA OPTS`后加上如下配置：
```bash
-XX:PermSize=16m -XX:MaxPermSize=128m
```
即为：`JAVA_OPTS=’-Xms256m –Xmx512m -XX:PermSize=128m -XX:MaxPermSize=512m’`
此配置表示JAVA永久保存区域（即不会被虚拟机回收）初始大小为16M，最大为128M。

修改内存后，可启动TOMCAT，输入`http://127.0.0.1:8080`，进入Status，会提示输入登录的用户名和密码，用户可以在conf/tomcat-user.xml中节点`<tomcat-users>`添加如下配置(配置完后需要重启TOMCAT)，
```xml
<role rolename="manager"/>
<user username="admin" password="admin" roles="manager"/>
```
登录后即可看到TOMCAT当前的空闲内存和最大内存。

tomcat报**Exception in thread "http-8080-36" java.lang.OutOfMemoryError: PermGen space**异常的解决：

`PermGen space`的全称是`Permanent Generation space`,是指内存的永久保存区域,

这块内存主要是被JVM存放Class和Meta信息的,Class在被Loader时就会被放到PermGen space中,
它和存放类实例(Instance)的Heap区域不同,GC(Garbage Collection)不会在主程序运行期对
PermGen space进行清理，所以如果你的应用中有很多CLASS的话,就很可能出现PermGen space错误,
这种错误常见在web服务器对JSP进行pre compile的时候。如果你的WEB APP下都用了大量的第三方jar, 其大小
超过了jvm默认的大小(4M)那么就会产生此错误信息了。
解决方法： 手动设置MaxPermSize大小

修改TOMCAT_HOME/bin/catalina.sh
在`echo "Using CATALINA_BASE: $CATALINA_BASE"`上面加入以下行：
```bash
JAVA_OPTS="-server -XX:PermSize=128m -XX:MaxPermSize=256m”
```

建议：将相同的第三方jar文件移置到tomcat/shared/lib目录下，这样可以达到减少jar 文档重复占用内存的目的
