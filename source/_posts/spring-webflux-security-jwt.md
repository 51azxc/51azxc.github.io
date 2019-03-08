---
title: Spring WebFlux Security与JWT整合
date: 2019-03-08 19:22:16
tags: ["Spring", "Spring Boot", "security"]
categories: ["Java", "Spring"]
---

现在的Web项目基本上都是前后端分离，后端专注于提供API接口即可，使用Spring Boot来开发RESTful API十分方便，如果需要保护这些API接口搭配Spring全家桶套餐中的Spring Security算是不错的选择。之前在Spring官网看Spring Security相关的示例是包含了网页访问相关的内容，并不是只提供接口访问的Web后端。接下来我就试试用JWT来保护这些后端提供的接口。

> `JSON Web Token(JWT)`是一个基于JSON的开放标准[（RFC 7519）](https://tools.ietf.org/html/rfc7519)，用于创建声明一些声明的访问令牌。[wiki](https://en.wikipedia.org/wiki/JSON_Web_Token)

### 基本配置

首先得去[Spring Initializr](https://start.spring.io/)获取需要的依赖。这里选择了`Reactive Web`,`Security`,`JPA`及`H2`。使用的Spring Boot版本是`2.1.1`。如果使用Maven,那对应的pom.xml部分配置如下：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
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
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.1</version>
    </dependency>
</dependencies>
```
主要使用了`Spring WebFlux`及`Spring Security`相关组件，数据库则是使用`H2 Database`。第三方的则是`jjwt`，用它来生成及解析`JSON web token`。

接下来是`application.yml`，用惯了`application.properties`，这次试试这个新的配置文件:
```yml
server:
  port: 8088

spring:
  datasource:
    url: jdbc:h2:mem:test;DB_CLOSE_DELAY=-1
    driver-class-name: org.h2.Driver
    username: root
    password: root
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```
这里主要指定了数据库的连接配置及端口。

### 数据模型

既然是基于经典的“用户-角色”权限管理模型，那数据层的实体类就必须要有一个用户及一个角色类。
```java
public enum RoleType {
    ROLE_ADMIN,
    ROLE_USER;

    public static RoleType fromString(String type) {
        if ("ROLE_ADMIN".equals(type)) {
            return ROLE_ADMIN;
        } else if ("ROLE_USER".equals(type)) {
            return ROLE_USER;
        } else {
            return null;
        }
    }
}
```
```java
@Entity
public class Role {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Enumerated(EnumType.STRING)
    @Column(length = 20, nullable = false)
    private RoleType roleType;

    public Role(RoleType roleType) {
        this.roleType = roleType;
    }
    // getter/setter...
}
```
角色实体类中的字段为`id`及`roleType`。`id`则是在数据库中的主键，而`roleType`则是用来指定一些定义好的角色名称。它通过`@Enumerated`来使用枚举类型字段，而`EnumType.STRING`表示插入到数据库的数据为枚举的名称而不是默认的数字类型。
接下来需要做的是一个用户实体类，用户实体类包含了必不可少的`id`,`username`,及`password`字段外，还需要有一个`roles`用来关联对应的角色实体类`Role`。这里为了后续查找更加方便就直接实现了`UserDetails`接口：
```java
@Entity
@JsonIgnoreProperties(
        value = {"password", "roles", "hibernateLazyInitializer", "handler"}
)
public class User implements UserDetails {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 50, unique = true)
    private String username;

    @Column(nullable = false)
    private String password;

    @ManyToMany(cascade = { CascadeType.PERSIST, CascadeType.MERGE, CascadeType.REFRESH }, fetch = FetchType.LAZY)
    @JoinTable(joinColumns = @JoinColumn(name = "user_id"), inverseJoinColumns = @JoinColumn(name = "role_id"))
    private Set<Role> roles = new HashSet<>();

    public User() {}

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles.stream()
                .map(role -> new SimpleGrantedAuthority(role.getRoleType().name()))
                .collect(Collectors.toList());
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
    
    // getter/setter...
}
```
数据实体都有了，接下来自然是数据仓库层了。得益于`Spring boot JPA`强大的封装能力，我们可以用很少的代码就完成大部分的工作：
```java
@Repository
public interface RoleRepository extends CrudRepository<Role, Long> {
    Optional<Role> findByRoleType(RoleType roleType);
    @Query(value = "select * from role r join user_roles ur on r.id = ur.role_id " +
            "join user u on ur.user_id = u.id where u.username = ?", nativeQuery = true)
    Set<Role> findByUsers_username(String username);
}
```
只需要集成`CrudRepository`接口，就能拥有`CRUD`一系列初始的方法。如果需要自定义查询方法，只需要按照`findBy<Properties>`的格式就能实现一个基本的查找方法。如果想要实现复杂的查询，可以使用`@Query`注解来实现。

### 服务层

要对用户身份进行验证，自然需要通过某种方式来加载用户信息。首先需要编写一个实现`ReactiveUserDetailsService`接口的子类。它类似于WebFlux中的`UserDetailsService`,里边需要实现的方法是`findByUsername`,对应着`UserDetailsService`里的`loadUserByUsername`方法。这个方法可以定义从指定的数据仓库中获取用户信息。由于这里是将用户数据存入到数据库中，自然需要用到上边定义的`UserRepository`来查找:
```java
public class UserService implements ReactiveUserDetailsService {

    private final UserRepository userRepository;
    private final RoleRepository roleRepository;
    private final PasswordEncoder passwordEncoder;

    @Autowired
    public UserService(UserRepository userRepository,
                       RoleRepository roleRepository,
                       PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.roleRepository = roleRepository;
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public Mono<UserDetails> findByUsername(String username) {
        Set<Role> roles = roleRepository.findByUsers_username(username);
        Optional<User> user = userRepository.findByUsername(username);
        user.ifPresent(u -> u.setRoles(roles));
        return Mono.justOrEmpty(user);
    }

    @Transactional
    public User saveUser(User user, List<String> roleTypes) {
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        Set<Role> roles = roleTypes.stream().map(RoleType::fromString)
                .map(roleType -> roleRepository.findByRoleType(roleType).get())
                .collect(Collectors.toSet());
        user.setRoles(roles);
        return userRepository.save(user);
    }

    @Transactional
    public void saveRoles(List<String> roleTypes) {
        Set<Role> roles = roleTypes.stream()
                .map(RoleType::fromString)
                .map(Role::new)
                .collect(Collectors.toSet());
        roleRepository.saveAll(roles);
    }

    public boolean existsUser(String username) {
        return userRepository.existsByUsername(username);
    }

    public Mono<String> getUsernameById(Long id) {
        return Mono.justOrEmpty(userRepository.findUsernameById(id));
    }
}
```
这里边还有些其他的方法，都是根据对用户实体类的一些操作。最主要的是`findByUsername`方法，它通过`UserRepository`查找到对应的用户，并且返回一个`Mono<User>`对象。
接下来需要通过`jjwt`这个库来生成及解析jwt密钥：
```java
@Component
public class TokenProvider {

    private final static String SECRET_KEY = "HelloWorld";
    private final static int EXPIRATION_DAY = 7;
    private final static String PAYLOAD_ROLES = "roles";

    public String encode(Authentication auth) {
        String subject = String.valueOf(auth.getPrincipal());
        List<String> roles = auth.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority).collect(Collectors.toList());

        LocalDate localDate = LocalDate.now();
        Date now = Date.from(localDate.atStartOfDay(ZoneId.systemDefault()).toInstant());
        LocalDate expiredLocalDate = localDate.plusDays(EXPIRATION_DAY);
        Date expiredDate = Date.from(expiredLocalDate.atStartOfDay(ZoneId.systemDefault()).toInstant());

        return Jwts.builder()
                .setSubject(subject)
                .claim(PAYLOAD_ROLES, roles)
                .setIssuedAt(now)
                .setExpiration(expiredDate)
                .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
                .compact();
    }

    public Authentication decode(String token) {
        Claims claims;
        try {
            claims = Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token).getBody();
        } catch (ExpiredJwtException e) {
            throw new BadCredentialsException("Expired token");
        } catch (UnsupportedJwtException e) {
            throw new BadCredentialsException("Unsupported token");
        } catch (MalformedJwtException e) {
            throw new BadCredentialsException("Malformed token");
        } catch (SignatureException | IllegalArgumentException e) {
            throw new BadCredentialsException("Invalid token");
        }
        List<String> roles = (List<String>)claims.get(PAYLOAD_ROLES, List.class);
        List<GrantedAuthority> authorities = roles.stream()
                .map(SimpleGrantedAuthority::new).collect(Collectors.toList());
        return new UsernamePasswordAuthenticationToken(claims.getSubject(), token, authorities);
    }
}
```
首先先定义了一些需要用到的常量，例如密钥，过期天数及需要传递的一些重要信息。`encode`方法是将`Authentication`中的认证用户部分信息编译成一个jwt，而`decode`方法自然是从jwt中解析出对应的信息，并包装成一个`UsernamePasswordAuthenticationToken`以供后续调用。

> 对于生成的json web token，可以使用[jwt.io](https://jwt.io/)来解析一下看看内部存储结构是怎么样的。

现在获取用户信息及编码/解码jwt的接口都有了，接下来就应该通过`Spring Security`组件将它们整合在一起了。

### 安全配置

首先需要写一个`AuthenticationManager`用来认证用户。它实现了`ReactiveAuthenticationManager`接口，这个接口里需要实现的方法是`authenticate`方法，在这个方法里来认证一个用户是否有效:
```java
public class TokenAuthenticationManager implements ReactiveAuthenticationManager {

    private final PasswordEncoder passwordEncoder;
    private final UserService userService;

    @Autowired
    public TokenAuthenticationManager(UserService userService, PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
        this.userService = userService;
    }

    @Override
    public Mono<Authentication> authenticate(Authentication authentication) {
        if (authentication.isAuthenticated()) { return Mono.just(authentication); }
        return Mono.just(authentication)
                .switchIfEmpty(Mono.error(new BadCredentialsException("Bad Credentials")))
                .map(authenticationToken -> authenticationToken.getPrincipal().toString())
                .flatMap(userService::findByUsername)
                .switchIfEmpty(Mono.error(new UsernameNotFoundException("User not found")))
                .filter(u -> passwordEncoder.matches(authentication.getCredentials().toString(), u.getPassword()))
                .switchIfEmpty(Mono.error(new BadCredentialsException("Invalid username or password")))
                .cast(User.class)
                .map(u -> new UsernamePasswordAuthenticationToken(u.getId(), null, u.getAuthorities()));
    }
}
```
在return那条挺长的语句里，通过传入的`authentication`获取指定的用户凭证，然后通过`UserService`里的`findByUsername`找到指定用户，并且判断用户密码是不是一致。如果成功最终封装成一个`UsernamePasswordAuthenticationToken`。这里将用户ID作为用户的认证信息，用户的权限角色作为传递的权限组。

定义好了`AuthenticationManager`之后，自然需要指定在哪里调用它。接下来是需要是实现一个`ServerSecurityContextRepository`接口的类来调用它：
```java
public class TokenSecurityContextRepository implements ServerSecurityContextRepository {

    private final TokenProvider tokenProvider;
    private final TokenAuthenticationManager tokenAuthenticationManager;

    @Autowired
    public TokenSecurityContextRepository(TokenProvider tokenProvider, TokenAuthenticationManager tokenAuthenticationManager) {
        this.tokenProvider = tokenProvider;
        this.tokenAuthenticationManager = tokenAuthenticationManager;
    }

    @Override
    public Mono<Void> save(ServerWebExchange exchange, SecurityContext context) {
        return Mono.defer(() -> Mono.error(new UnsupportedOperationException("No save method")));
    }

    @Override
    public Mono<SecurityContext> load(ServerWebExchange exchange) {
        return Mono.justOrEmpty(exchange.getRequest().getHeaders().getFirst(HttpHeaders.AUTHORIZATION))
                .filter(s -> s.length() > 7 && s.startsWith("Bearer "))
                .map(s -> tokenProvider.decode(s.substring(7)))
                .onErrorResume(Mono::error)
                .flatMap(auth -> tokenAuthenticationManager.authenticate(auth))
                .switchIfEmpty(Mono.error(new BadCredentialsException("Invalid Credentials")))
                .map(SecurityContextImpl::new);
    }
}
```
这里需要实现的是一个`sateless`风格的后端服务，因此`save`方法就不需要使用到了。主要实现的是`load`方法，通过请求头部的`Authorization`属性获取对应的`Bearer Token`，然后利用上述的`TokenProvider`来解码，得到的`Authentication`通过`AuthencationManager`来验证用户是否成功验证。如果成功就返回一个`SecurityContextImpl`最终可以存入到`SecurityContextHolder`中。

最终需要配置的自然是`WebFlux Security`配置类了：
```java
@Configuration
@EnableReactiveMethodSecurity
@EnableWebFluxSecurity
public class WebSecurityConfig {

    private final UserRepository userRepository;
    private final RoleRepository roleRepository;
    private final TokenProvider tokenProvider;

    @Autowired
    public WebSecurityConfig(UserRepository userRepository,
                             RoleRepository roleRepository,
                             TokenProvider tokenProvider) {
        this.userRepository = userRepository;
        this.roleRepository = roleRepository;
        this.tokenProvider = tokenProvider;
    }

    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity httpSecurity) {
        return httpSecurity
                .csrf().disable()
                .logout().disable()
                .cors()
                .and()
                .exceptionHandling()
                .accessDeniedHandler((exchange, e) -> Mono.error(e))
                .and()
                .authenticationManager(tokenAuthenticationManager())
                .securityContextRepository(tokenSecurityContextRepository())
                .authorizeExchange()
                .pathMatchers("/login","/register","/favicon.ico").permitAll()
                .pathMatchers(HttpMethod.GET, "/").permitAll()
                .pathMatchers("/admin").hasRole("ADMIN")
                .pathMatchers("/user").hasRole("USER")
                .anyExchange().authenticated()
                .and()
                .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    public UserService userService() {
        return new UserService(userRepository, roleRepository, passwordEncoder());
    }

    @Bean
    public TokenAuthenticationManager tokenAuthenticationManager() {
        return new TokenAuthenticationManager(userService(), passwordEncoder());
    }

    @Bean
    public TokenSecurityContextRepository tokenSecurityContextRepository() {
        return new TokenSecurityContextRepository(tokenProvider, tokenAuthenticationManager());
    }
}
```
最开头的那些`@Enable...`注解看名字就知道是什么功能了。整个配置类中最关键的自然是`securityWebFilterChain`方法。根据那些调用链的方法名字其实都挺好理解里边的方法作用。首先`exceptionHandling`指定了异常发生的情况下需要怎么处理。这里就指定了一种，及`accessDeniedHandler`，在用户权限不够的情况下该怎么办。然后需要将我们自定义的`authenticationManager`及`securityContextRepository`组合在一起愉快的工作。接下来的`.authorizeExchange()`当然是用来配置请求级别的安全性的。`.pathMatchers().permitAll()`用来配置哪些路径是给所有用户开放访问的，包括匿名用户。`.pathMatchers().hasRole()`则指定了这些路径必须要有指定的角色用户才能够访问。`.anyExchange()`表示匹配以上任何请求，而`.authenticated()`则将匹配的请求仅限于经过身份验证的用户。

> `.hasRole()`方法是会将传入的参数加上"ROLE_"前缀的，如果自定义的角色名不是这个样式的可以使用`.hasAuthority()`方法

### 完成功能接口

现在基本的配置都已经完成了，就该来写一些接口调用那些服务了。首先还是得写一些简单的数据传输载体类:
```java
public class AuthRequest {
    @NotBlank @Size(min = 3, max = 20)
    private String username;
    @NotBlank
    private String password;
    
    //getter/setter...
}
```
这个是用来给用户注册/登陆的。然后需要定义一个登陆成功后返回的载体:
```java
public class AuthResponse {
    private String token;
    
    //getter/setter...
}
```
很简单，只是返回一个可用的token而已。
接下来应该是通过传统的`@Controller`来提供对外访问接口。不过这里用到的是`WebFlux`，索性就将函数式编程风格弄到底吧。直接用`@Service`来操作:
```java
@Service
public class UserHandler {

    private final UserService userService;
    private final TokenAuthenticationManager tokenAuthenticationManager;
    private final Validator validator;
    private final TokenProvider tokenProvider;

    @Autowired
    public UserHandler(UserService userService,
                       TokenAuthenticationManager tokenAuthenticationManager,
                       Validator validator, TokenProvider tokenProvider) {
        this.userService = userService;
        this.tokenAuthenticationManager = tokenAuthenticationManager;
        this.validator = validator;
        this.tokenProvider = tokenProvider;
    }
    
    ...
}
```
首先是给提供给用户注册的方法：
```java
public Mono<ServerResponse> signUp(ServerRequest request) {
    return request.bodyToMono(AuthRequest.class)
            .filter(authRequest -> validator.validate(authRequest).isEmpty())
            .switchIfEmpty(Mono.error(new BadCredentialsException("Invalid username or password")))
            .filter(authRequest -> !userService.existsUser(authRequest.getUsername()))
            .switchIfEmpty(Mono.error(new UserExistsException("Username Exists")))
            .map(authRequest -> new User(authRequest.getUsername(), authRequest.getPassword()))
            .doOnNext(user -> userService.saveUser(user, Arrays.asList("ROLE_USER")))
            .flatMap(user -> ServerResponse.ok()
                    .body(Mono.just("success"), String.class)
                    .switchIfEmpty(ServerResponse.badRequest()
                            .body(Mono.just("failed"), String.class)));
}
```
通过传入的数据判断一下用户名是否存在，如果不存在就通过相关Service方法存入用户信息到数据库，并且将相关信息返回给前端。
接下来是登陆的方法：
```java
public Mono<ServerResponse> signIn(ServerRequest request) {
    return request.bodyToMono(AuthRequest.class)
            .filter(authRequest -> validator.validate(authRequest).isEmpty())
            .switchIfEmpty(Mono.error(new BadCredentialsException("Invalid username or password")))
            .flatMap(authRequest -> {
                Authentication authentication= new UsernamePasswordAuthenticationToken(
                        authRequest.getUsername(), authRequest.getPassword());
                return tokenAuthenticationManager.authenticate(authentication);
            })
            .doOnError(e -> new BadCredentialsException("Invalid username or password"))
            .doOnNext(authentication -> ReactiveSecurityContextHolder.withAuthentication(authentication))
            .map(auth -> new AuthResponse(tokenProvider.encode(auth)))
            .flatMap(authResponse ->
                    ServerResponse.ok().contentType(MediaType.APPLICATION_JSON_UTF8)
                            .body(BodyInserters.fromObject(authResponse))
                            .switchIfEmpty(ServerResponse.badRequest().build()));

}
```
通过`AuthenticationManager`来验证传入的用户名/密码是否正确，成功之后就通过`ReactiveSecurityContextHolder.withAuthentication`将验证信息存入到`SecurityContext`中。最后通过`TokenProvider`生成对应的token返回给前端。
接下来就是一些对应权限的方法了：
```java
public Mono<ServerResponse> helloPage(ServerRequest request) {
    return ServerResponse.ok().body(Mono.just("Hello World!"), String.class);
}

public Mono<ServerResponse> adminPage(ServerRequest request) {
    return ReactiveSecurityContextHolder.getContext()
            .flatMap(this::getCurrentUsername)
            .flatMap(username -> ServerResponse.ok()
                    .body(BodyInserters.fromObject("Hello admin: " + username)));
}

public Mono<ServerResponse> userPage(ServerRequest request) {
    return ReactiveSecurityContextHolder.getContext()
            .flatMap(this::getCurrentUsername)
            .flatMap(username -> ServerResponse.ok()
                    .body(Mono.just("Hello user: " + username), String.class));
}

private Mono<String> getCurrentUsername(SecurityContext securityContext) {
    return Mono.justOrEmpty(securityContext.getAuthentication()).filter(Objects::nonNull)
            .map(authentication -> authentication.getPrincipal()).filter(Objects::nonNull)
            .map(o -> Long.valueOf(o.toString()))
            .flatMap(userService::getUsernameById)
            .switchIfEmpty(Mono.error(new BadCredentialsException("Current user not exists")));
}
```
分别对应着不同用户权限返回的不同信息。
上述注册功能中使用了一个自定义的异常类`UserExistsException`:
```java
public class UserExistsException extends RuntimeException  {
    public UserExistsException(String message) {
        super(message);
    }
    @Override
    public synchronized Throwable fillInStackTrace() {
        return this;
    }
}
```

最后需要在配置类中将这些方法与对应的访问路径关联起来：
```java
 @Bean
public RouterFunction<ServerResponse> routerFunction() {
    return RouterFunctions.route(GET("/"), userHandler::helloPage)
            .andRoute(POST("/login")
                    .and(contentType(MediaType.APPLICATION_JSON)
                    .and(accept(MediaType.APPLICATION_JSON))), userHandler::signIn)
            .andRoute(POST("/register")
                    .and(contentType(MediaType.APPLICATION_JSON)), userHandler::signUp)
            .andRoute(GET("/admin"), userHandler::adminPage)
            .andRoute(GET("/user"), userHandler::userPage);
}
```

### 测试功能接口

现在一切都准备好了，只待运行应用了。不过这里用的是内存数据库，所以一开始还是得在运行的时候加入一些初始数据的:
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);
        UserService userService = context.getBean(UserService.class);
        List<String> roles = Arrays.asList("ROLE_ADMIN", "ROLE_USER");
        userService.saveRoles(roles);
        userService.saveUser(new User("admin", "admin"), roles);
    }
}
```
在这里加入了`ROLE_ADMIN`及`ROLE_USER`两个角色，分别对应着管理员/普通用户的权限，然后生成了一个管理员用户。
如无意外，一般都能够正常运行。接下来就需要测试一下这些接口是否能够正常运作了。测试RESTful API接口的话，如果想要更方便的体验可以使用postman。这里使用curl来测试，毕竟是简单的体验。

首先先看看对所有用户都开放的主页：
```bash
$ curl http://localhost:8088
Hello World!
```
接下来试试一个受保护的接口:
```bash
$ curl http://localhost:8088/admin
{"status":"UNAUTHORIZED","localDateTime":"2019-03-01 16:15:28","message":"Invalid Credentials"}
```
可以看到匿名用户是没有权限访问这个接口的，接下来就试试初始的用户登陆后的访问结果：
```bash
$ curl -H "Content-Type: application/json" -X POST -d '{"username":"admin", "password":"admin"}' http://localhost:8088/login
{"token":"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwicm9sZXMiOlsiUk9MRV9VU0VSIiwiUk9MRV9BRE1JTiJdLCJpYXQiOjE1NTEzOTg0MDAsImV4cCI6MTU1MjAwMzIwMH0.1lqpT4lGsgQTfyl4i15jMemeHultjE27xLxws2YFsro"}
```
接下来将这个返回的token放到头部再访问上边的接口:
```bash
$ curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwicm9sZXMiOlsiUk9MRV9VU0VSIiwiUk9MRV9BRE1JTiJdLCJpY
XQiOjE1NTEzOTg0MDAsImV4cCI6MTU1MjAwMzIwMH0.1lqpT4lGsgQTfyl4i15jMemeHultjE27xLxws2YFsro" http://localhost:8088/admin
Hello admin: admin
```
可以看到它能够正常显示了。接下来试试其他的用户接口：
```bash
$ curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwicm9sZXMiOlsiUk9MRV9VU0VSIiwiUk9MRV9BRE1JTiJdLCJpY
XQiOjE1NTEzOTg0MDAsImV4cCI6MTU1MjAwMzIwMH0.1lqpT4lGsgQTfyl4i15jMemeHultjE27xLxws2YFsro" http://localhost:8088/user
Hello user: admin
```
因为一开始创建的管理员用户拥有`ROLE_ADMIN`及`ROLE_USER`两个角色,因此他能够正常访问这些接口。接下来试试注册一个普通用户，看看他的访问权限能否正常区分:
```bash
$ curl --header "Content-Type: application/json" --request POST --data '{"username":"user", "password":"user"}' http://
localhost:8088/register
success
```
注册成功之后再用这个用户登陆一下：
```bash
$ curl -H "Content-Type: application/json" -X POST -d '{"username":"user", "password":"user"}' http://localhost:8088/lo
gin
{"token":"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIyIiwicm9sZXMiOlsiUk9MRV9VU0VSIl0sImlhdCI6MTU1MTM5ODQwMCwiZXhwIjoxNTUyMDAzMjAwfQ.pW7WFJZzvHxKHW4tYU-AQTG1P0ky43nMsTfWFNCKQl8"}
```
然后利用这个token去访问一下对应的接口：
```bash
$ curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIyIiwicm9sZXMiOlsiUk9MRV9VU0VSIl0sImlhdCI6MTU1MTM5ODQwM
CwiZXhwIjoxNTUyMDAzMjAwfQ.pW7WFJZzvHxKHW4tYU-AQTG1P0ky43nMsTfWFNCKQl8" http://localhost:8088/user
Hello user: user
```
结果正常，然后试试看他能不能访问只有`ROLE_ADMIN`才有权限访问的接口:
```bash
$ curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIyIiwicm9sZXMiOlsiUk9MRV9VU0VSIl0sImlhdCI6MTU1MTM5ODQwM
CwiZXhwIjoxNTUyMDAzMjAwfQ.pW7WFJZzvHxKHW4tYU-AQTG1P0ky43nMsTfWFNCKQl8" http://localhost:8088/admin
{"status":"FORBIDDEN","localDateTime":"2019-03-01 16:32:25","message":"Access Denied"}
```
不出意外的出现了访问失败的提示了。

### 总结

至此，所有的流程都结束了。过程挺简单，`WebFlux`跟传统的`WebMVC`是有点区别，函数式编程风格还是挺爽的，不过大体上都一致。如果需要查看完整的代码，可以访问[这里](https://github.com/51azxc/JavaBaseExample/tree/master/spring-boot-webflux-security)。
