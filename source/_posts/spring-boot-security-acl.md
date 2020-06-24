---
title: Spring Security ACL的基本使用
date: 2019-03-29 19:27:13
tags: ["Spring", "Spring Boot", "security"]
categories: ["Java", "Spring"]
---

访问控制列表（*Access Control List*,即 ACL）是用以对指定对象权限进行管理的一组列表。Spring Security ACL可以在单个域对象上定义特定的用户/角色权限。例如，一个拥有管理员角色的用户可以读取(READ)与删除(DELETE)所有的资源，而普通用户只能查看自己的资源。可以认为是不同的用户/角色对不同的指定对象有着不同的权限。接下来我就来试试Spring Security ACL是如何实现这一基本功能的。

<!--more-->

### 配置

#### 前期准备

首先需要准备依赖，这里使用的是Spring Boot。如果使用的是maven，`pom.xml`部分配置如下：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-acl</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-data</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
</dependency>
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.10.6</version>
    <type>jar</type>
</dependency>
```
接下来需要在`application.yml`配置文件中配置数据库连接信息：
```yml
spring:
  datasource:
    url: jdbc:h2:mem:test;DB_CLOSE_DELAY=-1
    driver-class-name: org.h2.Driver
    username: root
    password: root
    schema: "classpath:db_schema.sql"
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```
因为Spring Security ACL是基于数据库表来管理的，因此需要使用`spring.datasource.schema`这属性来指定需要执行的sql文件来生成需要的表结构。[Spring官网](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#dbschema-acl)已经提供了不同数据库的表生成脚本，可以根据自己的需要去获取对应的sql语句存储到`db_schema.sql`中。
接下来简单的说一下生成的各张表的用途：

 * `acl_sid`:用于存储系统可识别的安全身份。主要有两种，一种是`PrncipalSid`，是唯一的已认证用户，一种是`GrantedAuthoritySid`，是可以给多个用户的权限。
 * `acl_class`:存储域对象的全限定名。
 * `acl_object_identity`: 存储特定域对象的对象标识定义。
 * `acl_entry`: 存储每个SID与ObjectIdentity对应的ACL权限。
 
#### 配置类

接下来需要通过一个配置类来配置所需要的ACL服务:
```java
@Configuration
@EnableAutoConfiguration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class AclConfig extends GlobalMethodSecurityConfiguration {
    private final DataSource dataSource;
    public AclConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    ...
}
```
在`@EnableGlobalMethodSecurity`注解里，可以通过指定开启对应的注解来对方法进行权限控制：

* **prePostEnabled**: 开启之后可以使用一系列前缀为`@Pre`或者`@Post`的注解。它们用来检查对方法的调用前/后的权限。常用的有`@PreAuthorize`,`@PostAuthorize  `,`@PreFilter`,`@PostFilter`。支持SpEL表达式。如`@PreAuthorize("hasRole('ADMIN')")`
* **securedEnabled**：开启之后可以使用`@Secured`注解。如`@Secured("ROLE_ADMIN")`,`@Secured({"ROLE_USER", "ROLE_ADMIN"})`。
*  **jsr250Enabled**:开启之后支持`@RolesAllowed`注解。如`@RolesAllowed("ROLE_ADMIN")`。

首先需要定义的自然是`AclService`接口,它负责对ACL进行相关操作。这里使用的是一个基于JDBC的相关实现类`JdbcMutableAclService`:
```java
@Bean
public JdbcMutableAclService aclService() {
    return new JdbcMutableAclService(dataSource, lookupStrategy(), aclCache());
}
```
类如其名，`JdbcMutableAclService`是通过`JdbcTemplate`对数据库进行访问。它会通过id来检索`acl_sid`与`acl_class`表的新记录，如果使用的是其他的数据库，需要提供正确的主键查找方式给`JdbcMutableAclService`的`sidIdentityQuery`及`classIdentityQuery`。
`JdbcMutableAclService`构造方法需要传入三个参数，第一个是`DataSource`，提供给`JdbcTemplate`。`dataSource`已经在配置类中通过构造方法注入了，因此可以直接拿来使用了。`LookupStrategy`是一个基于SQL语句查找策略类`BasicLookupStrategy`：
```java
@Bean
public LookupStrategy lookupStrategy() {
    return new BasicLookupStrategy(dataSource, aclCache(), aclAuthorizationStrategy(), permissionGrantingStrategy());
}
```
它是用来给`AclService`提供一个查找策略来发现`Sid`与`ObjectIdentity`的关系。它需要一个`AclAuthorizationStrategy`接口来判断一个认证用户是否有足够的权限对`Acl`进行操作:
```java
@Bean
public AclAuthorizationStrategy aclAuthorizationStrategy() {
    return new AclAuthorizationStrategyImpl(new SimpleGrantedAuthority("ROLE_ADMIN"));
}
```
这里使用了`AclAuthorizationStrategyImpl`这个默认的实现类来指定哪些角色对指定对象是否具有必需的权限。它的构造方法支持输入的参数必须为1或者3个。这里边会定义三个特殊的权限：修改所有人权限，修改统计信息权限及更改其他ACL/ACE的详细信息。如果参数只有一个，那就会将这三个特殊权限全部给定这一个角色。
`BasicLookupStrategy`的构造方法还需要一个`PermissionGrantingStrategy`来判断一个`ACL`对象的权限是否能够授予一个或者多个`Sid`:
```java
@Bean
public PermissionGrantingStrategy permissionGrantingStrategy() {
    return new DefaultPermissionGrantingStrategy(new ConsoleAuditLogger());
}
```
这里就使用默认的权限授予策略了。

最后需要配置用来缓存`ACL`信息的缓存类:
```java
@Bean
public EhCacheBasedAclCache aclCache() {
    return new EhCacheBasedAclCache(aclEhCacheFactoryBean().getObject(), permissionGrantingStrategy(), aclAuthorizationStrategy());
}

@Bean
public EhCacheFactoryBean aclEhCacheFactoryBean() {
    EhCacheFactoryBean ehCacheFactoryBean = new EhCacheFactoryBean();
    ehCacheFactoryBean.setCacheManager(aclCacheManager().getObject());
    ehCacheFactoryBean.setCacheName("aclCache");
    return ehCacheFactoryBean;
}

@Bean
public EhCacheManagerFactoryBean aclCacheManager() {
    return new EhCacheManagerFactoryBean();
}
```
Spring Security ACL是使用`Ehcache`来做缓存的，这里就按部就班的提供对应的服务即可。

在主要的`AclService`已经配置好了之后，如果想在`PrePostEnabled`那些注解中来通过ACL判断权限，就需要开启基于表达式的访问控制权限:
```java
@Override
protected MethodSecurityExpressionHandler createExpressionHandler() {
    return defaultMethodSecurityExpressionHandler();
}

@Bean
public MethodSecurityExpressionHandler defaultMethodSecurityExpressionHandler() {
    DefaultMethodSecurityExpressionHandler expressionHandler = new DefaultMethodSecurityExpressionHandler();
    AclPermissionEvaluator permissionEvaluator = new AclPermissionEvaluator(aclService());
    expressionHandler.setPermissionEvaluator(permissionEvaluator);
    expressionHandler.setPermissionCacheOptimizer(new AclPermissionCacheOptimizer(aclService()));
    return expressionHandler;
}
```
之前这个配置类继承于`GlobalMethodSecurityConfiguration`，因此可以通过重写`createExpressionHandler`方法，将`AclPermissionEvaluator`加入到其中，这样就可以在SPEL中使用`hasPermission`等关键词了。配置了那么久，接下来应该试一试它们的功力了。

### 通过SpEL指定方法级的访问权限

首先随便定义一个需要被保护的资源:
```java
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false, length = 50)
    private String title;
    @Column
    private String content;
    @Column
    private String author;
    //getter/setter...
}
```
接下来自然就是对应的数据仓库类了:
```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {

    @PostAuthorize("hasRole('ADMIN') or hasPermission(returnObject.get(), 'READ')")
    Optional<Post> findById(Long aLong);

    @PostFilter("hasRole('ADMIN') or hasPermission(filterObject, 'READ')")
    List<Post> findAll();

    @PreAuthorize("hasRole('ADMIN') or hasPermission(#post, 'DELETE')")
    void delete(@Param("post") Post post);
}
```
首先在读取单个对象的时候，这里通过`@PostAuthorize`注解来判断，当调用方法之后，当前调用者是否拥有`ROLE_ADMIN`角色护着对这个返回的对象是否有读取的权限(`BasePermission.READ`)。因为这里返回的类型是`Optional`，所以需要写成`returnObject.get()`;如果返回的就是对象本身，直接用`returnObject`即可。如果是读取一堆对象，会在调用方法之后筛选出符合指定权限访问的那些对象。而对于删除操作，则会在当前用户调用方法之前判断其对当前对象有无对应的权限，没有的话则会抛出`AccessDeniedException`异常。

接下来在服务层需要创建ACL关联对应的用户与对象:
```java
@Service
public class PostService {
    private final PostRepository postRepository;
    private final MutableAclService aclService;

    @Autowired
    public PostService(PostRepository postRepository, 
                        MutableAclService aclService) {
        this.postRepository = postRepository;
        this.aclService = aclService;
    }

    @Transactional
    public void savePost(Post post) {
        Post p = postRepository.save(post);
        ObjectIdentity objectIdentity = new ObjectIdentityImpl(Post.class, p.getId());
        MutableAcl acl = aclService.createAcl(objectIdentity);
        PrincipalSid sid = new PrincipalSid(post.getAuthor());
        int index = acl.getEntries().size();
        acl.insertAce(index++, BasePermission.ADMINISTRATION, 
            new GrantedAuthoritySid("ROLE_ADMIN"), true);
        acl.insertAce(index++, BasePermission.DELETE, sid, true);
        acl.insertAce(index++, BasePermission.READ, sid, true);
        aclService.updateAcl(acl);
    }

    public Optional<Post> getPost(Long id) {
        return postRepository.findById(id);
    }
    
    public List<Post> getPosts() {
        return postRepository.findAll();
    }
    
    @Transactional
    public void deletePost(Long id) {
        getPost(id).ifPresent(post -> {
            postRepository.delete(post);
            ObjectIdentity objectIdentity = new ObjectIdentityImpl(Post.class, id);
            aclService.deleteAcl(objectIdentity, false);
        });
    }
}
```
在创建实体成功之后，根据返回的id创建一个`ObjectIdentity`,然后用这个`ObjectIdentity`通过`AclService`的`createAcl`方法创建一个ACL。接下来就需要指定哪些Sid对这个`ObjectIdentity`有哪些权限。这里首先指定了拥有管理员角色(`ROLE_ADMIN`)的`GrantedAuthoritySid`对它是有`ADMINISTRATION`权限。然后再指定当前创建它的用户与读(`READ`)与删除(`DELETE`)的权限。最后再通过`AclService`的`updateAcl`方法就可以更新ACL了。
`PrincipalSid`构造函数支持通过`Authentication`来获取认证用户信息，所以也可以使用`new PrincipalSid(SecurityContextHolder.getContext().getAuthentication())`来当前用户为Sid。只不过这样需要集成Spring Security。

> *BasePermission*有5个定义的权限，从小到大依次是`READ`,`WRITE`,`CREATE`, `DELETE`,`ADMINISTRATION`。如果不满足，也可以自定义BasePermission来增加权限。

删除实体之后自然也需要通过`AclService`的`deleteAcl`方法删除对应的ACL。

### 单元测试

最后我们需要写一些单元测试来看看他们是否生效。首先需要在测试开始之前就插入一些测试用的数据：
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class AclTests {
    @Autowired PostService postService;

    @Before
    public void setUp() {
        Post p1 = new Post("post 1", "admin's post", "admin");
        postService.savePost(p1);
        Post p2 = new Post("post 2", "user1's post", "user1");
        postService.savePost(p2);
        Post p3 = new Post("post 3", "user2's post", "user2");
        postService.savePost(p3);
    }
    ...
}
```
接下来可以看看拥有管理员权限的用户是否能够对那些资源进行操作:
```java
@WithMockUser(username = "admin", roles = {"ADMIN", "USER"})
@Test
public void testAdminAccess() {
    Pageable pageable = PageRequest.of(0, 10);
    Assert.assertThat(postService.getPosts().size(), CoreMatchers.is(3));
    postService.deletePost(1L);
    Assert.assertThat(postService.getPosts.size(), CoreMatchers.is(2));
    postService.deletePost(2L);
    Assert.assertThat(postService.getPosts().size(), CoreMatchers.is(1));
}
```
接下来，我们看看只有普通用户权限的用户是不是只能看到自己创建的资源:
```java
@WithMockUser("user1")
@Test
public void testUserGetPosts() {
    List<Post> posts = postService.getPosts();
    Assert.assertThat(posts.size(), CoreMatchers.is(1));
    Assert.assertThat(posts.get(0).getAuthor(), CoreMatchers.is("user1"));
    Assert.assertThat(postService.getPost(2L).get().getAuthor(), CoreMatchers.is("user1"));
}
```
然后再看看如果查看不属于自己的资源是否会抛出`AccessDeniedException`异常:
```java
@WithMockUser("user2")
@Test(expected = AccessDeniedException.class)
public void testUserCannotGetPost() {
    Assert.assertThat(postService.getPost(2L).get().getAuthor(), CoreMatchers.is("user1"));
    Assert.assertThat(postService.getPost(3L).get().getAuthor(), CoreMatchers.is("user2"));
}
```
最后看看能否删除属于自己的资源，以及强行删除他人资源会不会出错：
```java
@WithMockUser("user1")
@Test(expected = AccessDeniedException.class)
public void testUserCannotDeletePost() {
    postService.deletePost(3L);
}

@WithMockUser("user2")
@Test
public void testUserDeletePost() {
    Assert.assertThat(postService.getPost(3L).get().getAuthor(), CoreMatchers.is("user2"));
    postService.deletePost(3L);
}
```

### 总结
ACL是一个比基于用户/角色权限管理还要更加细分细粒度的一种权限模型。Spring Security是通过一些数据库表来管理相关ACL的，`AclService`则是完成所有操作的核心。这部分并没有集成到spring-boot-starter-security中，可能是有点小众吧。
如果想看完整的代码可以参考[Github](https://github.com/51azxc/JavaBaseExample/tree/master/spring-boot-security-acl)。

> 参考
> [Spring Security（19）——对Acl的支持](https://elim.iteye.com/blog/2269021)
> [Spring Security中的ACL](https://www.bbsmax.com/A/amd0m016zg/)