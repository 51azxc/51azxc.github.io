title: "spring注解"
date: 2015-05-15 16:58:10
tags: ["mvc","transaction", "annotation"]
categories: ["java", "spring"]
---

### spring 基本注解

> [Spring Annotation 详解](http://blog.csdn.net/zsm653983/article/details/8133113)
> [autowire异常的三个情况](http://zhongzhihua.iteye.com/blog/613305)
> [Spring注释@Qualifier](http://yangchengwanhong.blog.sohu.com/234465733.html)

`@Component`: 是一个泛化的概念，仅仅表示一个组件 (Bean) ，可以作用在任何层次。
`@Service`: 常作用在业务层
`@Constroller`: 通常作用在控制层
`@Repository`: 只能标注在 DAO 类上
`@Scope`: 指定 Bean 的作用域
```java
@Scope("prototype")     
@Repository     
public class Demo { … }
```
`@PostConstruct`: 标注初始化之后执行的回调方法
`@PreDestroy`: 标注销毁之前执行的回调方法
`@Required`:  进行对Bean的依赖检查，判断给定Bean的相应`Setter`方法是否都在实例化的时候被调用了标签提供了 dependency-check 属性用于进行依赖检查。该属性的取值包括以下几种：

* `none`: 默认不执行依赖检查
* `simple`: 对原始基本类型和集合类型进行检查。
* `objects`: 对复杂类型进行检查(除了 simple 所检查类型之外的其他类型)。
* `all`: 对所有类型进行检查。

----------

#### 自动装配
`@Autowired`: 自动装配.Spring 在装配 Bean 的时候，根据指定的自动装配规则，将某个 Bean 所需要引用类型的 Bean 注入进来。 元素提供了一个指定自动装配类型的 autowire 属性，该属性有如下选项：

* `no`: 显式指定不使用自动装配。
* `byName`: 如果存在一个和当前属性名字一致的Bean，则使用该Bean 进行注入。如果名称匹配但是类型不匹配，则抛出异常。如果没有匹配的类型，则什么也不做。
* `byType`: 如果存在一个和当前属性类型一致的 Bean ( 相同类型或者子类型 )，则使用该 Bean 进行注入。byType 能够识别工厂方法，即能够识别 `factory-method` 的返回类型。如果存在多个类型一致的 Bean，则抛出异常。如果没有匹配的类型，则什么也不做。
* `constructor`: 与 byType 类似，只不过它是针对构造函数注入而言的。如果当前没有与构造函数的参数类型匹配的 Bean，则抛出异常。使用该种装配模式时，优先匹配参数最多的构造函数。
* `autodetect`: 根据Bean的自省机制决定采用`byType`还是`constructor`进行自动装配。如果 Bean 提供了默认的构造函数，则采用`byType`;否则采用`constructor`进行自动装配。

在按类型匹配的时候(可能是`byType`、`constructor`、`autodetect`)，同一个类型可能存在多个Bean如果被注入的属性是数组、集合或者Map，这可能没有问题，但是如果只是简单的引用类型，则会抛出异常。解决方法有如下几种：

* 取消该 Bean 的自动装配特性，使用显式的注入。我们可能不希望某个 Bean 被当作其他 Bean 执行自动封装时的候选对象，我们可以给该 增加 `autowire-candidate="false"`。(`autowire-candidate`属性和`autowire`属性相互独立，互不相干) 另外，我们可以设置`default-autowire-candidates`属性，可以在该属性中指定可以用于自动装配候选 Bean 的匹配模式，比如 `default-autowire-candidates="*serv,*dao"`，这表示所有名字以serv或者dao结尾的 Bean 被列为候选，其他则忽略，相当于其他 Bean 都指定为 `autowire-candidate="false"`，此时可以显式为 指定 `autowire-candidate="true"`。在 上指定的设置要覆盖 上指定的设置。
* 如果在多个类型相同的 Bean 中有首选的 Bean，那么可以将该 的`primary`属性设置为 "true" ，这样自动装配时便优先使用该 Bean 进行装配。此时不能将`autowire-candidate` 设为 false

**使用`@Autowired`注解进行装配，只能是根据类型进行匹配**。@Autowired 注解可以用于 Setter 方法、构造函数、字段，甚至普通方法，前提是方法必须有至少一个参数。@Autowired 可以用于数组和使用泛型的集合类型。然后 Spring 会将容器中所有类型符合的 Bean 注入进来。@Autowired 标注作用于 Map 类型时，如果 Map 的 key 为 String 类型，则 Spring 会将容器中所有类型符合 Map 的 value 对应的类型的 Bean 增加进来，用 Bean 的 id 或 name 作为 Map 的 key。

当标注了`@Autowired`后，自动注入不能满足，则会抛出异常。我们可以给`@Autowired`标注增加一个`required=false`属性，以改变这个行为。另外，每一个类中只能有一个构造函数的`@Autowired.required()`属性为`true`。否则就出问题了。如果用`@Autowired`同时标注了多个构造函数，那么，Spring 将采用贪心算法匹配构造函数 ( 构造函数最长 )。

`@Qualifier`: 当容器中存在多个Bean的类型与需要注入的相同时，注入将不能执行，我们可以给`@Autowired`增加一个候选值，做法是在`@Autowired`后面增加一个`@Qualifier`标注，提供一个String类型的值作为候选的Bean的名字。
```java
@Autowired(required=false)
@Qualifier("ppp")
public void setPerson(person p){}
```
可以作用于方法的参数 ( 对于方法只有一个参数的情况，我们可以将 @Qualifer 标注放置在方法声明上面，但是推荐放置在参数前面 )
```java
@Autowired(required=false)     
public void sayHello(@Qualifier("ppp")Person p,String name){}
```
**如果没有明确指定Bean的qualifier名字，那么默认名字就是 Bean 的名字**
如果`@Autowired`注入的是**BeanFactory**、**ApplicationContext**、**ResourceLoader** 等系统类型，那么则不需要`@Qualifier`，此时即使提供了`@Qualifier`注解，也将会被忽略;而对于自定义类型的自动装配，如果使用了`@Qualifier`注解并且没有名字与之匹配的 Bean，则自动装配匹配失败。

----------

`@Resource`: **如果希望根据name执行自动装配，那么应该使用JSR-250提供的`@Resource`注解，而不应该使用`@Autowired`与`@Qualifier`的组合。**`@Resource`使用`byName`的方式执行自动封装。`@Resource`标注可以作用于带一个参数的Setter方法、字段，以及带一个参数的普通方法上。`@Resource`注解有一个name属性，用于指定 Bean 在配置文件中对应的名字。如果没有指定 name 属性，那么默认值就是字段或者属性的名字。*@Resource 和 @Qualifier 的配合虽然仍然成立，但是 @Qualifier 对于 @Resource 而言，几乎与 name 属性等效*。

如果 `@Resource` 没有指定 name 属性，那么使用`byName`匹配失败后，会退而使用`byType`继续匹配，如果再失败，则抛出异常。在没有为`@Resource`注解显式指定`name`属性的前提下，如果将其标注在**BeanFactory 类型**、**ApplicationContext类型**、**ResourceLoader类型**、**ApplicationEventPublisher类型**、**MessageSource类型**上，那么 Spring 会自动注入这些实现类的实例，不需要额外的操作。此时 name 属性不需要指定 ( 或者指定为"")，否则注入失败;如果使用了 `@Qualifier`，则该注解将被忽略。而对于用户自定义类型的注入，`@Qualifier` 和 name 等价，并且不被忽略

----------

#### JAVA配置Bean声明
`@Configuration`:  用于指定配置信息的类上加上`@Configuration`注解，以明确指出该类是 Bean 配置的信息源。Spring 对标注 Configuration 的类有如下要求

* 配置类不能是 final 的
* 配置类不能是本地化的，亦即不能将配置类定义在其他类的方法内部
* 配置类必须有一个无参构造函数

`@Bean`: `AnnotationConfigApplicationContext`将配置类中标注了`@Bean`的方法的返回值识别为**Spring Bean**，并注册到容器中，受IoC容器管理。`@Bean`的作用等价于XML 配置中的标签。
```java
@Configuration     
public class BookStoreDaoConfig{     
    @Bean     
    public UserDao userDao(){ return new UserDaoImpl();}     
    @Bean     
    public BookDao bookDao(){return new BookDaoImpl();}     
}
```
与以上配置等价的 XML 配置如下
```xml
<bean id="userDao" class="bookstore.dao.UserDaoImpl"/>      
<bean id="bookDao" class="bookstore.dao.BookDaoImpl"/>
```

`@Bean`具有以下四个属性：

* `name`: 指定一个或者多个 Bean 的名字。这等价于 XML 配置中 的 name 属性。
* `initMethod`: 容器在初始化完Bean之后，会调用该属性指定的方法。这等价于XML配置中的`init-method`属性。
* `destroyMethod`: 该属性与`initMethod`功能相似，在容器销毁 Bean 之前，会调用该属性指定的方法。这等价于XML配置中的`destroy-method`属性。
* `autowire`: 指定 Bean 属性的自动装配策略，取值是`Autowire`类型的三个静态属性。`Autowire.BY_NAME，Autowire.BY_TYPE，Autowire.NO`。与 XML 配置中的`autowire`属性的取值相比，这里少了` constructor`，这是因为`constructor`在这里已经没有意义了。

`AnnotationConfigApplicationContext`提供了三个构造函数用于初始化容器。

* `AnnotationConfigApplicationContext()`：该构造函数初始化一个空容器，容器不包含任何 Bean 信息，需要在稍后通过调用其`register()`方法注册配置类，并调用`refresh()`方法刷新容器。
* `AnnotationConfigApplicationContext(Class... annotatedClasses)`：这是最常用的构造函数，通过将涉及到的配置类传递给该构造函数，以实现将相应配置类中的 Bean 自动注册到容器中。
* `AnnotationConfigApplicationContext(String... basePackages)`：该构造函数会自动扫描以给定的包及其子包下的所有类，并自动识别所有的 Spring Bean，将其注册到容器中。它不但识别标注`@Configuration`的配置类并正确解析，而且同样能识别使用`@Repository`、`@Service`、`@Controller`、`@Component`标注的类。

除了使用上面第三种类型的构造函数让容器自动扫描 Bean 的配置信息以外，`AnnotationConfigApplicationContext` 还提供了`scan()`方法，其功能与上面也类似，该方法主要用在容器初始化之后动态增加 Bean 至容器中。调用了该方法以后，通常需要立即手动调用`refresh()`刷新容器，以让变更立即生效。

`@Import`:引入其他`@Configuration`类
```java
@Configuration   
@Import({BookStoreServiceConfig.class,BookStoreDaoConfig.class})   
public class BookStoreConfig{ … }  
```

**混合使用 XML 与注解进行 Bean 的配置**
以XML为主的配置
```java
@Configuration   
public class MyConfig{   
@Bean   
    public UserDao userDao(){   
        return new UserDaoImpl();   
    }   
}
```
在XML配置bean
```xml
<beans … >   
    ……   
    <context:annotation-config />   
    <bean class=”demo.config.MyConfig”/>   
</beans>
```
由于启用了针对注解的 Bean 后处理器，因此在 ApplicationContext 解析到 MyConfig 类时，会发现该类标注了`@Configuration`注解，随后便会处理该类中标注`@Bean`的方法，将这些方法的返回值注册为容器总的 Bean。

对于以上的方式，如果存在多个标注了`@Configuration`的类，则需要在 XML 文件中逐一列出。另一种方式是使用前面提到的自动扫描功能，配置如下：
```xml
<context:component-scan base-package=”bookstore.config” /> 
```
如果在`Configuration`中可以使用`@ComponentScan`进行扫描
对于以注解为中心的配置方式，只需使用`@ImportResource`注解引入存在的 XML 即可
```java
@Configuration   
@ImportResource(“classpath:/bookstore/config/spring-beans.xml”)   
public class MyConfig{   
……   
} 
```
容器的初始化过程和纯粹的以配置为中心的方式一致
```java
AnnotationConfigApplicationContext ctx =   
              new AnnotationConfigApplicationContext(MyConfig.class);
```

----------

### Spring 使用注解方式进行事物管理

> [Spring 使用注解方式进行事物管理](http://blog.csdn.net/zhaofsh/article/details/6285869)

`@Transactional`: 当标于类前时, 标示类中所有方法都进行事物处理

事物传播行为介绍:
`@Transactional(propagation=Propagation.REQUIRED)`
如果有事务, 那么加入事务, 没有的话新建一个(默认情况下)
`@Transactional(propagation=Propagation.NOT_SUPPORTED)`
容器不为这个方法开启事务
`@Transactional(propagation=Propagation.REQUIRES_NEW)`
不管是否存在事务,都创建一个新的事务,原来的挂起,新的执行完毕,继续执行老的事务
`@Transactional(propagation=Propagation.MANDATORY)`
必须在一个已有的事务中执行,否则抛出异常
`@Transactional(propagation=Propagation.NEVER)`
必须在一个没有的事务中执行,否则抛出异常(与`Propagation.MANDATORY`相反)
`@Transactional(propagation=Propagation.SUPPORTS)`
如果其他bean调用这个方法,在其他bean中声明事务,那就用事务.如果其他bean没有声明事务,那就不用事务.

事物超时设置:
`@Transactional(timeout=30)`//默认是30秒

事务隔离级别:
`@Transactional(isolation = Isolation.READ_UNCOMMITTED)`
读取未提交数据(会出现脏读, 不可重复读) 基本不使用
`@Transactional(isolation = Isolation.READ_COMMITTED)`
读取已提交数据(会出现不可重复读和幻读)
`@Transactional(isolation = Isolation.REPEATABLE_READ)`
可重复读(会出现幻读)
`@Transactional(isolation = Isolation.SERIALIZABLE)`
串行化

MYSQL: 默认为REPEATABLE_READ级别
SQLSERVER: 默认为READ_COMMITTED

脏读 : 一个事务读取到另一事务未提交的更新数据
不可重复读 : 在同一事务中, 多次读取同一数据返回的结果有所不同, 换句话说,
后续读取可以读到另一事务已提交的更新数据. 相反, "可重复读"在同一事务中多次
读取数据时, 能够保证所读数据一样, 也就是后续读取不能读到另一事务已提交的更新数据
幻读 : 一个事务读到另一个事务已提交的insert数据

----------

### Spring MVC部分注解

> [Spring4.0系列3-@RestController](http://wiselyman.iteye.com/blog/2002446)
> [Spring @SessionAttributes @ModelAttribute](http://my.oschina.net/rouchongzi/blog/161871)
> [6 详解@SessionAttributes](http://blog.sina.com.cn/s/blog_6c9e93cc0101a3h9.html)
> [spring学习之@ModelAttribute运用详解](http://blog.csdn.net/li_xiao_ming/article/details/8349115)
> [Spring MVC之@RequestMapping 详解](http://blog.csdn.net/kobejayandy/article/details/12690041)
> [@RequestParam @RequestBody @PathVariable 等参数绑定注解详解](http://blog.csdn.net/walkerjong/article/details/7946109)
> [@RequestBody, @ResponseBody 注解详解](http://blog.csdn.net/walkerjong/article/details/7520896)
> [SpringMVC @RequestBody接收Json对象字符串](http://www.cnblogs.com/quanyongan/archive/2013/04/16/3024741.html)
> [spring3.1.1 使用@ResponseBody 返回中文时出现乱码](http://www.iteye.com/topic/1129096)
> [Spring 注解学习手札（八）补遗——@ExceptionHandler](http://snowolf.iteye.com/blog/1636050)
> [Spring MVC 的请求参数获取的几种方法](http://babyduncan.iteye.com/blog/1124453)
> [Spring @PropertySource example](https://www.mkyong.com/spring/spring-propertysources-example/)

#### @RestController
`@RestController`: 继承于`@Controller`，开发REST服务的时候不需要使用`@Controller`而专门的`@RestController`.当实现一个RESTful web services的时候，response将一直通过response body发送。为了简化开发，Spring 4.0提供了一个专门版本的controller
```java
@Target(value=TYPE)  
@Retention(value=RUNTIME)  
@Documented  
@Controller  
@ResponseBody  
public @interface RestController
```
使用了`@RestController`注解之后，方法就可以不需要再指定`@ResponseBody`了

----------

#### @ModelAttribute
`@ModelAttribute`: 该注解有两个用法，一个是用于方法上，一个是用于参数上；
用于方法上时：  通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model；
用于参数上时： 用来通过名称对应，把相应名称的值绑定到注解的参数bean上；要绑定的值来源于：
A） `@SessionAttributes` 启用的attribute 对象上；
B） `@ModelAttribute` 用于方法上时指定的model对象；
C） 上述两种情况都没有时，new一个需要绑定的bean对象，然后把request中按名称对应的方式把值绑定到bean中

`@ModelAttribute`注释void返回值的方法
```java
public class HelloWorldController {  
  
    @ModelAttribute  
    public void populateModel(@RequestParam String abc, Model model) {  
       model.addAttribute("attributeName", abc);  
    }  

    @RequestMapping(value = "/helloWorld")  
    public String helloWorld() {  
       return "helloWorld";  
    }  
}
```
这个例子，在获得请求/helloWorld 后，populateModel方法在helloWorld方法之前先被调用，它把请求参数（/helloWorld?abc=text）加入到一个名为attributeName的model属性中，在它执行后helloWorld被调用，返回视图名helloWorld和model已由@ModelAttribute方法生产好了。

`@ModelAttribute`注释返回具体类的方法
```java
@ModelAttribute  
public Account addAccount(@RequestParam String number) {  
   return accountManager.findAccount(number);  
}
```
这种情况，model属性的名称没有指定，它由返回类型隐含表示，如这个方法返回Account类型，那么这个model属性的名称是account。

`@ModelAttribute(value="")`注释返回具体类的方法
```java
@Controller  
public class HelloWorldController {  

    @ModelAttribute("attributeName")  
    public String addAccount(@RequestParam String abc) {  
       return abc;  
    }  

    @RequestMapping(value = "/helloWorld")  
    public String helloWorld() {  
       return "helloWorld";  
    }  
}
```
这个例子中使用`@ModelAttribute`注释的value属性，来指定model属性的名称。model属性对象就是方法的返回值。它无须要特定的参数

`@ModelAttribute`和`@RequestMapping`同时注释一个方法
```java
@Controller  
public class HelloWorldController {  
    @RequestMapping(value = "/helloWorld.do")  
    @ModelAttribute("attributeName")  
    public String helloWorld() {  
       return "hi";  
    }  
}
```
这时这个方法的返回值并不是表示一个视图名称，而是model属性的值，视图名称由`RequestToViewNameTranslator`根据请求"/helloWorld.do"转换为逻辑视图helloWorld。
Model属性名称有`@ModelAttribute(value=””)`指定，相当于在request中封装了key=attributeName，value=hi

`@ModelAttribute`注释一个方法的参数
```java
@Controller  
public class HelloWorldController {  

    @ModelAttribute("user")  
    public User addAccount() {  
       return new User("jz","123");  
    }  

    @RequestMapping(value = "/helloWorld")  
    public String helloWorld(@ModelAttribute("user") User user) {  
       user.setUserName("jizhou");  
       return "helloWorld";  
    }  
}
```
在这个例子里，`@ModelAttribute("user") User user`注释方法参数，参数user的值来源于`addAccount()`方法中的model属性。
此时如果方法体没有标注`@SessionAttributes("user")`，那么scope为request，如果标注了，那么scope为session
    
----------

#### @SessionAttributes
`@SessionAttributes`:该注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。该注解有value、types两个属性，可以通过名字和类型指定要使用的attribute 对象.

通过Model绑定
```java
@Controller
@RequestMapping(value = "login")
@SessionAttributes("mysession")
//定义把Model中的mysession属性的值绑定到Session中
public class LoginController {
    @RequestMapping(method = RequestMethod.POST)
    public String login(@ModelAttribute User user, ModelMap model) {
       String viewName = "";
       boolean check = true;
       if (check) {
           model.addAttribute("mysession", "123");
           viewName = "redirect:/home";
       } else {
           viewName = "redirect:/";
       }
       return viewName;
    }
}
```
这样我们不但可以在请求所对应的JSP视图页面中通过`request.getAttribute()`和`session.getAttribute()`获取mysession，还可以在下一个请求所对应的JSP视图页面中通过`session.getAttribute()`或`ModelMap#get()`访问到这个属性。
这里我们仅将一个ModelMap的属性放入Session中，其实`@SessionAttributes`允许指定多个属性。你可以通过字符串数组的方式指定多个属性，如 `@SessionAttributes({“attr1”,”attr2”})`。此外，`@SessionAttributes`还可以通过属性类型指定要session化的ModelMap属性，如`@SessionAttributes(types=User.class)`，当然也可以指定多个类，如 `@SessionAttributes(types = {User.class,Dept.class})`，还可以联合使用属性名和属性类型指定：`@SessionAttributes(types = {User.class,Dept.class},value={“attr1”,”attr2”})`。

通过`@ModelAttribute`绑定
```java
@Controller
@RequestMapping(value = "login")
//此处定义需要绑定到session中的model名称
@SessionAttributes("user")
public class LoginController {
    @RequestMapping(method = RequestMethod.POST)
    //@ModelAttribute将绑定到session中
    public String login(@ModelAttribute("user") User user, ModelMap model){
       String viewName = "";
       boolean check = true;
       if (check) {
           viewName = "redirect:/home";
       } else {
           viewName = "redirect:/";
       }
       return viewName;
    }
}
```
`@SessionAttributes`需要清除时，使用`SessionStatus.setComplete()`;来清除。注意，它只清除`SessionAttributes`的session，不会清除HttpSession的数据。故如用户身份验证对象的session一般不同它来实现，还是用`session.setAttribute`等传统的方式实现。

----------

#### @RequestMapping
`@RequestMapping`: RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。主要有如下属性:

* `value`:  指定请求的实际地址，指定的地址可以是URI Template 模式,可以指定为普通的具体值,含有某变量的一类值及含正则表达式的一类值
* `method`: 指定请求的method类型， GET、POST、PUT、DELETE等
* `consumes`: 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
* `produces`: 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；
* `params`: 指定request中必须包含某些参数值是，才让该方法处理
* `headers`: 指定request中必须包含某些指定的header值，才能让该方法处理请求

```java
@Controller
@RequestMapping("/hello")
public class HelloController{
  //默认
  @RequestMapping("/one")
  public void oneMethod(){}
  
  //指定请求方法为POST类型
  @RequestMapping(value="/two",method=RequestMetho.POST)
  public void twoMethod(){}
  
  //含有某变量
  @RequestMapping(value="/{three}", method = RequestMethod.GET)
  public void threeMethod(@PathVariable String three){}
  
  //含正则表达式
  @RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\d\.\d\.\d}.{extension:\.[a-z]}")
  public void fourMethod(@PathVariable String version, @PathVariable String extension) {}
  
  //consumes指定仅处理request Content-Type为“application/json”类型的请求
  //produces指定返回的内容类型为text/plain,同时指定字符集，否则碰中文会乱码
  @RequestMapping(value="/five",method=RequestMethod.POST, consumes="application/json",produces="text/plain;charset=utf-8")
  @ReponseBody
  public void fiveMethod(@RequestBody User user){}
  
  //params指定仅处理请求中包含了名为“myParam”，值为“myValue”的请求
  //headers指定仅处理request的header中包含了指定“Refer”请求头和对应值为“http://www.baidu.com/”的请求
  @RequestMapping(value="/six",method=RequestMethod.GET, params="myParam=myValue", headers="Referer=http://www.baidu.com/")
  public void sixMethod(){}
}
```

----------

`@PathVariable`:当使用`@RequestMapping` URI template 样式映射时， 即 `someUrl/{paramId}`, 这时的paramId可通过`@Pathvariable`注解绑定它传过来的值到方法的参数上
```java
@RequestMapping(value="/{three}", method = RequestMethod.GET)
public void threeMethod(@PathVariable String three){}
```
上面代码把URI template 中变量three，绑定到方法的参数上。若方法参数名称和需要绑定的uri template中变量名称不一致，需要在`@PathVariable("name")`指定uri template中的名称

----------

`@RequestHeader`: 可以把Request请求header部分的值绑定到方法的参数上
Request 的header部分
```http
Host                    localhost:8080  
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9  
Accept-Language         fr,en-gb;q=0.7,en;q=0.3  
Accept-Encoding         gzip,deflate  
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7  
Keep-Alive              300
```
把request header部分的 Accept-Encoding的值，绑定到参数encoding上了， Keep-Alive header的值绑定到参数keepAlive上
```java
@RequestMapping("/seven")  
public void sevenMethod(@RequestHeader("Accept-Encoding") String encoding,  
                              @RequestHeader("Keep-Alive") long keepAlive)  { }
```

----------

`@CookieValue`: 可以把Request header中关于cookie的值绑定到方法的参数上
```java
@RequestMapping("/eight")  
public void eightMethod(@CookieValue("JSESSIONID") String cookie)  { }
```

----------

#### @RequestParam
`@RequestParam`
A） 常用来处理简单类型的绑定，通过`Request.getParameter()` 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用`request.getParameter()`方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值；

B）用来处理Content-Type: 为`application/x-www-form-urlencoded`编码的内容，提交方式GET、POST；

C) 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定；
```java
@RequestMapping("/nine", method=RequestMethod.POST)
public void nineMethod(@RequestMapping("name")String name, @RequestMapping(required=false)int age){}
```

----------

#### @RequestBody
1. 该注解用于读取Request请求的body部分数据，使用系统默认配置的`HttpMessageConverter`进行解析，然后把相应的数据绑定到要返回的对象上；
2. 再把`HttpMessageConverter`返回的对象数据绑定到 controller中方法的参数上。

使用时机：

1. GET、POST方式提时， 根据request header Content-Type的值来判断:

* `application/x-www-form-urlencoded`， 可选（即非必须，因为这种情况的数据`@RequestParam`, `@ModelAttribute`也可以处理，当然`@RequestBody`也能处理）；
* `multipart/form-data`, 不能处理（即使用`@RequestBody`不能处理这种格式的数据）；
* 其他格式， 必须（其他格式包括`application/json`, `application/xml`等。这些格式的数据，必须使用`@RequestBody`来处理）；

2. PUT方式提交时， 根据request header Content-Type的值来判断:

  * `application/x-www-form-urlencoded`， 必须；
  * `multipart/form-data`, 不能处理；
  * 其他格式， 必须；

说明：request的body部分的数据编码格式由header部分的Content-Type指定；
```js
$(function(){  
    var saveDataAry=[];  
    var data1={"userName":"u1","address":"by"};  
    var data2={"userName":"u2","address":"yx"};  
    saveDataAry.push(data1);  
    saveDataAry.push(data2);         
    $.ajax({ 
        type:"POST", 
        url:"user/saveUser", 
        dataType:"json",      
        contentType:"application/json",               
        data:JSON.stringify(saveData), 
        success:function(data){ 
                                    
        } 
        }); 
});
```
```java
@RequestMapping(value = "saveUser", method = {RequestMethod.POST }}) 
@ResponseBody  
public void saveUser(@RequestBody List<User> users) { 
     userService.save(users); 
}
```

----------

#### @ResponseBody
该注解用于将Controller的方法返回的对象，通过适当的`HttpMessageConverter`转换为指定格式后，写入到Response对象的body数据区。

使用时机：返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；
