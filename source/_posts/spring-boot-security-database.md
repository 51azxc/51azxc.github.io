---
title: spring boot security with database
date: 2017-09-06 10:48:47
tags: ["spring boot", "security"]
categories: ["java", "spring"]
---

> [Securing a Web Application](https://spring.io/guides/gs/securing-web/)
> [springboot+mybatis＋SpringSecurity 实现用户角色数据库管理](http://blog.csdn.net/u012373815/article/details/54632176)
> [Spring Boot + Spring MVC + Spring Security + MySQL](https://medium.com/@gustavo.ponce.ch/spring-boot-spring-mvc-spring-security-mysql-a5d8545d837d)

在`Spring Boot`中加入`Spring Security`功能的话，官方给出了一个很好的例子。例子中给出的验证用户是放在内存中的，不过我想试试常规一点的方法，将用户存储到数据库中。

<!-- more -->

### 前期配置

首先到[Spring Initializr](http://start.spring.io/)中配置一个基本的项目模板，依赖中选择`Security`、`JPA`、`H2`、`Thymeleaf`等依赖，如果需要体验一下热部署的功能，也可以选择`DevTools`。下载好模板项目导入到IDE工作空间中，引入依赖。

接下来需要配置一下`application.properties`的一些属性:
```
spring.jpa.hibernate.ddl-auto=update
# 设置thymeleaf不缓存，可以即时刷新Html页面
spring.thymeleaf.cache=false
# 设置H2存储数据的方式为内存存储，数据库名为test
spring.datasource.url=jdbc:h2:mem:test;DB_CLOSE_DELAY=-1
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=root
spring.datasource.password=root
# 配置访问数据库的链接
spring.h2.console.path=/h2-console
# 开启数据库访问连接
spring.h2.console.enabled=true
# 初始化数据库数据SQL文件
spring.datasource.data=classpath:data.sql
```
然后在`src/main/resources`目录下创建`data.sql`，这里主要给角色表创建了一些初始数据。
```sql
insert into `role`(`id`, `role_name`) values(1, 'ROLE_ADMIN'),(2, 'ROLE_USER');
```

### 定义用户角色

首先创建角色模型：
```java
@Entity
public class Role {
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @Column(name="role_name")
    private String roleName;
	
    // getters and setters
}
```
然后是用户模型：
```java
@Entity
public class User {
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String username;
    private String password;
    @ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    @JoinTable(joinColumns = @JoinColumn(name = "user_id"), 
               inverseJoinColumns = @JoinColumn(name = "role_id"))
    private Set<Role> roles;

    // getters and setters
}
```
接下来则是定义对应的数据访问层：
```java
@Repository
public interface RoleRepository extends JpaRepository<Role, Long> {
    Role findByRoleName(String roleName);
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
}
```
创建服务层需要实现`security`的内置接口`UserDetailsService`，通过重写`loadUserByUsername`方法查找对应的用户: 
```java
@Transactional
@Service
public class UserService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;
    @Autowired
    private RoleRepository roleRepository;
    @Autowired
    private BCryptPasswordEncoder bcryptEncoder;

    @Override
    public UserDetails loadUserByUsername(String arg0)
            throws UsernameNotFoundException {
        User user = userRepository.findByUsername(arg0);
        if (user == null) {
            throw new UsernameNotFoundException("user not found");
        }
        Set<GrantedAuthority> auths = new HashSet<>();
        // 添加用户权限
        for (Role role : user.getRoles()) {
            GrantedAuthority auth = new SimpleGrantedAuthority(
                    role.getRoleName());
            auths.add(auth);
        }
        return new org.springframework.security.core.userdetails.User(
                user.getUsername(), user.getPassword(), auths);
    }

    public void saveUser(User user, String roleName) {
        user.setPassword(bcryptEncoder.encode(user.getPassword()));
        Role role = roleRepository.findByRoleName(roleName);
        Set<Role> roles = new HashSet<Role>(Arrays.asList(role));
        user.setRoles(roles);
        userRepository.save(user);
    }

}
```

### 配置WebSecuriy

先定义一个`WebMVC`的配置文件，用来映射`login.html`等页面到控制器中，这样就可以省去写对应的控制器：
```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello").setViewName("hello");
        registry.addViewController("/login").setViewName("login");
    }

}
```
然后则是`WebSecurity`的配置文件:
```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    BCryptPasswordEncoder bcryptEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    UserDetailsService userService() {
        return new UserService();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth)
            throws Exception {
        auth.userDetailsService(userService())
            .passwordEncoder(bcryptEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 为H2数据库控制台大开方便之门
        http.csrf().disable()
            .headers().frameOptions().disable()
            .and()
            .authorizeRequests().antMatchers("/h2-console/**").permitAll();

        http.authorizeRequests().antMatchers("/", "/home").permitAll()
            .and()
            .authorizeRequests().anyRequest().authenticated()
            .and()
            .formLogin().loginPage("/login")
                .defaultSuccessUrl("/hello").permitAll()
            .and()
            .logout().permitAll();
    }
	
/*
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth)
            throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user")
            .password("password")
            .roles("USER");
    }
*/
}
```
在这里自定义一个`WebSecurity`配置类通过继承`WebSecurityConfigurerAdapter`来重写一些方法。
首先通过重写`configure(AuthenticationManagerBuilder auth)`方法来配置自定义的认证方式。通过配置`UserDetailsService`，系统会通过`loadUserByUsername`方法来查找对应的用户，并且指定了密码的编码模式为`BCryptPasswordEncoder`,保障用户安全。
然后通过重写`configure(HttpSecurity http)`方法来定义一些权限访问模式。主要是：
* 通过`authorizeRequests`方法指定哪些URL需要被保护,`antMatchers`方法则表示需要匹配的URL。
* `permitAll`方法表示任何人都可以访问，而`authenticated`方法则表示需要认证才能访问。
* 通过`formLogin`方法指定用户需要登陆的操作。`loginPage`指向登陆页面，`defaultSuccessUrl`指定成功登陆后跳转的页面。
* `configureGlobal`方法则是配置一个存于内存中的授权用户，当然这里暂时用不到它。

### HTML页面

使用`Thmeleaf`编写一些简单的页面，在`src/main/resources/templeate`创建登陆页面：
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Login</title>
    </head>
    <body>
        <div th:if="${param.error}">
            Invalid username and password.
        </div>
        <div th:if="${param.logout}">
            You have been logged out.
        </div>
        <form th:action="@{/login}" method="post">
            <div><label> User Name : <input type="text" name="username"/> </label></div>
            <div><label> Password: <input type="password" name="password"/> </label></div>
            <div><input type="submit" value="Sign In"/></div>
        </form>
    </body>
</html>
```
然后是成功登陆后的跳转页面：
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Hello</title>
    </head>
    <body>
        <h1 th:inline="text">Hello [[${#httpServletRequest.remoteUser}]]!</h1>
        <form th:action="@{/logout}" method="post">
            <input type="submit" value="Sign Out"/>
        </form>
    </body>
</html>
```
如果没有授权的用户直接访问`/hello`的话，就会被跳转到`/login`页面。当然，需要先存入一个用户才行。在应用启动的时候加入一个用户：
```java
public static Application {

	public static void main(String[] args) {
		ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);
		UserService userService = (UserService)(context.getBean(UserService.class));
		User user = new User();
		user.setUsername("root");
		user.setPassword("root");
		userService.saveUser(user, "ROLE_ADMIN");
	}
}
```
这里我们通过调用`UserSerivce`中的方法加入一个用户，接下来就是运行一下项目查看成果了。