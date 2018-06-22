---
title: 初探istio
date: 2018-05-04 16:04:37
tags: ["istio","kubernetes"]
categories: ["kubernetes"]
---

[istio](https://istio.io/)项目是Service Mesh概念的最新实现，旨在所有主流集群管理平台上提供Service Mesh层，初期以实现Kubernetes上的服务治理层为目标。它由控制平面和数据平面组成：控制平面由Go语言实现，包括pilot、mixer、auth三个组件；数据平面功能由Envoy在pod中以Sidecar的部署形式提供。

<!-- more -->

istio由Google与IBM联合创作，看到它的爹都是这么强大的存在，还是赶紧来认识一下它。

这次的实验环境是在Kubernetes平台上，因为就目前来说istio对其支持最好。想想看istio的logo是一艘帆船，而kubernets的logo是舵轮，而docker的logo则是一只鲸鱼，它们之间的关系还真是微妙啊。

### 安装

目前使用的`istio`版本为`0.7.1`,对环境的要求为`Kubernetes >= 1.7.3,minikube >= 0.25`,我之前装的Kubernetes及minikube刚好符合这个要求，那就先启动minikube吧:
```bash
 sudo minikube start \
	--extra-config=controller-manager.ClusterSigningCertFile="/var/lib/localkube/certs/ca.crt" \
	--extra-config=controller-manager.ClusterSigningKeyFile="/var/lib/localkube/certs/ca.key" \
	--extra-config=apiserver.Admission.PluginNames=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota \
	--kubernetes-version=v1.9.0 \
	--vm-driver=none \
	--registry-mirror="https://registry.docker-cn.com"
```
这里对比官网的命令就多加了最后两个选项。`--vm-driver`表示使用本地**Docker**环境，`--registry-mirror`指定镜像加速地址。不然等下下载istio的核心服务镜像就可能不成功了。

然后根据官网的命令下载：
```bash
curl -L https://git.io/getLatestIstio | sh -
```
下载好后通过`cd`命令进入到`istio-0.7.1`文件目录下，将`istioctl`客户端加入到环境变量中:
```bash
export PATH=$PWD/bin:$PATH
```
在安装istio的核心组件之前，需要修改一下`istio.yaml`配置文件的`istion-ingress`服务部分:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: istio-ingress
  namespace: istio-system
  labels:
    istio: ingress
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 32000
    name: http
  - port: 443
    name: https
  selector:
    istio: ingress
```
因为我是在**minikube**环境中运行的，按照官网的说法(minikube不支持外部负载均衡)，需要将`istio-ingress`服务的类型设置为**NodePort**。如果不希望每次运行都随机分配一个端口号，可以通过`nodePort`属性指定。
接下来运行命令安装核心组件
```bash
kubectl apply -f install/kubernetes/istio.yaml
```
如果想要体验一下`istio`的自动注入功能，那就需要安装[sidecar injector webhook](https://istio.io/docs/setup/kubernetes/sidecar-injection.html#automatic-sidecar-injection),(要求`Kubernetes >= 1.9.0`)。这个功能模块我安装的过程中出了一个问题，就是在执行`webhook-create-signed-cert.sh`脚本的时候报了一个错：
```bash
error: error validating "STDIN": error validating data: [apiVersion not set, kind not set]; if you choose to ignore these errors, turn validation off with --validate=false
```
然后在[github issue](https://github.com/istio/issues/issues/261)上边发现也有人讨论类似的问题，解决办法就是将这个脚本的最后一行注释下边的语句替换成:
```shell
# create the secret with CA cert and server cert/key
echo "apiVersion: v1" > ${tmpdir}/create-secret.yaml
echo "kind: Secret" >> ${tmpdir}/create-secret.yaml
kubectl create secret generic ${secret} \
        --from-file=key.pem=${tmpdir}/server-key.pem \
        --from-file=cert.pem=${tmpdir}/server-cert.pem \
        --dry-run -o yaml >> ${tmpdir}/create-secret.yaml
kubectl -n ${namespace} apply -f ${tmpdir}/create-secret.yaml
```
这样这个模块才安装成功。感觉这部分可能是非正式版所以错误还是有的，后边就没有用自动注入的方式而采取的是手动注入的方式。

安装好之后查看一下相关服务及Pod是否已经成功运行了:
```bash
$ kubectl get svc -n istio-system
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                             AGE
istio-ingress   NodePort    10.104.248.2     <none>        80:32000/TCP,443:31392/TCP                                          2h
istio-mixer     ClusterIP   10.106.203.121   <none>        9091/TCP,15004/TCP,9093/TCP,9094/TCP,9102/TCP,9125/UDP,42422/TCP    2h
istio-pilot     ClusterIP   10.105.143.198   <none>        15003/TCP,15005/TCP,15007/TCP,15010/TCP,8080/TCP,9093/TCP,443/TCP   2h

$ kubectl get po -n istio-system
NAME                             READY     STATUS    RESTARTS   AGE
istio-ca-86f55cc46f-qqggd        1/1       Running   0          2h
istio-ingress-5bb556fcbf-g2mn5   1/1       Running   0          2h
istio-mixer-86f5df6997-d8gvp     3/3       Running   0          2h
istio-pilot-67d6ddbdf6-6b74h     2/2       Running   0          2h
```
> 如果安装了自动注入的模块，还有一个`istio-sidecar-injector-*`的Pod

----

### 部署应用

#### 编写测试用例

在成功安装好了istio的核心组件之后，就可以开始部署应用了。官网有个十分完整的案例[Bookinfo](https://istio.io/docs/guides/bookinfo.html),不过这里我就想着一起复习一下[spring boot 2 webflux](https://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html)就自己写了一个挺简单的例子。

主要有两个服务:`service1`及`service2`。客户端调用`service2`的服务，然后`service2`来调用`service1`的服务，最终将结果返回给客户端。

首先先创建`service1`的`spring boot`项目:
首先是一个简单的用户类:
```java
public class User {
	private Integer id;
	private String username;
	private String version;
	
	// getter and setter ...
}
```
接下来编写一个`Repository`类，实现最基本的*CRUD*：
```java
@Repository
public class UserRepository {
	private Map<Integer, User> map = new ConcurrentHashMap<>();
	private final static AtomicInteger ids = new AtomicInteger(0);
	
	public Mono<User> getUserById(Integer id) {
		return Mono.justOrEmpty(map.get(id)).switchIfEmpty(Mono.empty());
	}
	public Flux<User> getUsers() {
		return Flux.fromIterable(map.values());
	}
	public Mono<Integer> createUser(final Mono<User> user) {
		return user.doOnNext(u -> {
			Integer id = ids.incrementAndGet();
			u.setId(id);
			map.put(id, u);
		}).flatMap(u -> Mono.just(u.getId()));
	}
	public Mono<User> updateUser(final Mono<User> user) {
		return user.doOnNext(u -> map.put(u.getId(), u));
	}
	public Mono<User> removeUser(final Integer id) {
		return Mono.justOrEmpty(map.remove(id)).switchIfEmpty(Mono.empty());
	}
}
```
`Mono`表示`0..1`,`Flux`表示`0..n`。`Reactive Streams`已是`Rx`的标准了。这大概是`Spring boot`会采用它的原因之一吧。
接下来是`Service`：
```java
@Service
public class UserHandler {

	@Autowired
	private UserRepository repo;
	@Value("${SERVICE_VERSION:v1}")
	private String version;
	
	public Mono<ServerResponse> listUsers(ServerRequest request) {
		Flux<User> users = repo.getUsers();
		return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(users, User.class);
	}
	
	public Mono<ServerResponse> getUserById(ServerRequest request) {
		Integer userId = Integer.valueOf(request.pathVariable("id"));
		Mono<ServerResponse> notFound = ServerResponse.notFound().build();
		Mono<User> user = repo.getUserById(userId);
		return user.flatMap(u -> ServerResponse.ok()
					.contentType(MediaType.APPLICATION_JSON)
					.body(BodyInserters.fromObject(u)))
				.switchIfEmpty(notFound);
	}
	
	public Mono<ServerResponse> addUser(ServerRequest request) {
		Mono<User> user = request.bodyToMono(User.class).doOnNext(u->u.setVersion(version));
		return repo.createUser(user)
				.flatMap(id -> ServerResponse.ok()
						.contentType(MediaType.APPLICATION_JSON)
						.body(BodyInserters.fromObject(id)))
				.switchIfEmpty(ServerResponse.badRequest().build());
	}
	
	public Mono<ServerResponse> updateUser(ServerRequest request) {
		Integer userId = Integer.valueOf(request.pathVariable("id"));
		Mono<User> user = request.bodyToMono(User.class).doOnNext(u->{
			u.setId(userId);
			u.setVersion(version);
		});
		return repo.updateUser(user)
				.flatMap(u -> ServerResponse.ok()
						.contentType(MediaType.APPLICATION_JSON)
						.body(BodyInserters.fromObject(u)))
				.switchIfEmpty(ServerResponse.badRequest().build());
	}
	
	public Mono<ServerResponse> deleteUser(ServerRequest request) {
		Integer userId = Integer.valueOf(request.pathVariable("id"));
		return repo.removeUser(userId).flatMap(user -> ServerResponse.ok()
						.contentType(MediaType.APPLICATION_JSON)
						.body(BodyInserters.fromObject(user)))
				.switchIfEmpty(ServerResponse.badRequest().build());
	}
}
```
这里的`version`通过寻找系统变量`SERVICE_VERSION`来注入，默认值是`v1`。这个变量在后边创建的`Docker`镜像中加入。
早先接触`RxJava`的时候就觉得这种链式写法十分爽，调试起来就十分酸爽。
然后是`Routes`文件用来配置路由:
```java
import static org.springframework.web.reactive.function.server.RequestPredicates.*;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;

@Configuration
public class Routes {

	@Autowired
	private UserHandler userHandler;
	
	@Bean
	public RouterFunction<ServerResponse> routerFunction() {
		return route(GET("/api/users").and(accept(APPLICATION_JSON)), userHandler::listUsers)
		  .and(route(GET("/api/users/{id}").and(accept(APPLICATION_JSON)), userHandler::getUserById))
		  .and(route(POST("/api/users").and(contentType(APPLICATION_JSON)), userHandler::addUser))
		  .and(route(PUT("/api/users/{id}").and(contentType(APPLICATION_JSON)), userHandler::updateUser))
		  .and(route(DELETE("/api/users/{id}").and(accept(APPLICATION_JSON)), userHandler::deleteUser));
	}
}
```
这种声明式路由最早在`RoR`中见过，可能比较好管理吧。
最后在`applicatioin.properties`文件中设置`server.port=8088`指定端口。

然后继续写一个`service2`项目来调用上边的服务`service1`，这里用到了`webflux`的客户端`WebClient`，最主要的类就是一个`Controller`:
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

	@Autowired
	private WebClient webClient;
	
	@GetMapping
	public Flux<User> list() {
		return webClient.get().uri("/api/users").accept(MediaType.APPLICATION_JSON).retrieve().bodyToFlux(User.class);
	}
	
	@GetMapping("{id}")
	public Mono<String> getUser(@PathVariable final Integer id) {
		return webClient.get().uri("/api/users/{id}",id).accept(MediaType.APPLICATION_JSON)
				.retrieve().onStatus(HttpStatus::is4xxClientError, res -> Mono.error(new Throwable(res.statusCode().getReasonPhrase())))
				.bodyToMono(User.class)
				.flatMap(user->Mono.justOrEmpty("Hello, " + user.getUsername() + " " + user.getVersion()))
				.onErrorReturn("error");
	}
	
	@PostMapping
	public Mono<Integer> addUser(@RequestBody final User user) {
		return webClient.post().uri("/api/users").contentType(MediaType.APPLICATION_JSON)
				.body(BodyInserters.fromObject(user)).retrieve()
				.bodyToMono(Integer.class);
	}
	
	@PutMapping("{id}")
	public Mono<User> updateUser(@PathVariable final Integer id, @RequestBody final User user) {
		return webClient.put().uri("/api/users/{id}",id).contentType(MediaType.APPLICATION_JSON)
				.body(BodyInserters.fromObject(user)).retrieve()
				.bodyToMono(User.class);
	}
	
	@DeleteMapping("{id}")
	public Mono<String> deleteUser(@PathVariable final Integer id) {
		return webClient.delete().uri("/api/users/{id}",id).accept(MediaType.APPLICATION_JSON)
				.retrieve().bodyToMono(User.class)
				.doOnError(error->Mono.error(error))
				.flatMap(user->Mono.justOrEmpty("Bye, " + user.getUsername() + " " + user.getVersion()))
				.onErrorReturn("error");
	}
}
```
这里需要拷贝一下`service1`的`User.java`,然后指定一个配置类注入`WebClient`：
```java
@Configuration
public class AppConfig {
	
	@Value("${service1.url}")
	private String service1Url;
	
	@Bean WebClient webClient() {
		return WebClient.create(service1Url);
	}
}
```
最后需要在`application.properties`文件中配置端口及调用服务的`url`：
```
server.port=8880
service1.url=http://service1:8088
```
接下来就是通过`Maven`将各自的项目打包，然后放到不同的文件夹下，准备生成`Docker`镜像了。


#### 创建Docker镜像

将打包好的`jar`放入不同的文件夹下，然后分别生成对应的`Dockerfile`文件:
```
FROM java:8-jdk-alpine

COPY service1-0.0.1-SNAPSHOT.jar service1.jar

ARG service_version
ENV SERVICE_VERSION ${service_version:-v1}

ENTRYPOINT [ "java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "service1.jar" ]
```
这里主要是配置了一个`SERVICE_VERSION`环境变量，用来标识版本号,然后运行`build`命令生成镜像:
```bash
sudo docker build -t service1:v1 --build-arg service_version=v1 .
sudo docker build -t service1:v2 --build-arg service_version=v2 .
```

接下来是生成service2的镜像,跟上边的`Dockerfile`文件差不多，只不过需要拷贝的jar包不一样:
```
FROM java:8-jdk-alpine

COPY service2-0.0.1-SNAPSHOT.jar service2.jar

ENTRYPOINT [ "java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "service2.jar" ]
```
然后生成镜像:
```bash
sudo docker build -t service2:v1 .
```

#### 生成Kubernetes服务

接着生成相关的`Service`及`Deployment`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service1
  labels:
    app: service1    
spec:
  ports:
  - name: http
    port: 8088
  selector:
    app: service1
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: service1-v1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: service1
        version: v1
    spec:
      containers:
      - name: service1
        image: service1:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8088
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: service1-v2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: service1
        version: v2
    spec:
      containers:
      - name: service1
        image: service1:v2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8088
---
apiVersion: v1
kind: Service
metadata:
  name: service2
  labels:
    app: service2    
spec:
  ports:
  - name: http
    port: 8880
  selector:
    app: service2
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: service2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: service2
        version: v1
    spec:
      containers:
      - name: service2
        image: service2:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8880
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: service2-ingress
  annotations:
    kubernetes.io/ingress.class: "istio"
spec:
  rules:
  - http:
      paths:
      - path: /api/users.*
        backend:
          serviceName: service2
          servicePort: 8880
---
```
这里使用`Ingress`向集群外暴露服务。这里匹配了如果是`/api/users`前缀的路径流量都会流转到`service2`服务中。

接下来就是通过手动注入`istio`的方式部署应用了：
```bash
kubectl create -f <(istioctl kube-inject -f service1.yaml)
```
然后通过命令查看一下是否部署成功：
```bash
$ kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
service1-v1-6bcd8675b9-qgrdh   2/2       Running   1          3h
service1-v2-759bc48449-zwk4p   2/2       Running   1          3h
service2-6df4cd875c-jvzj6      2/2       Running   0          3h
```
这里每个`Pod`的数量都是2,说明istio已经成功将组件服务注入到对应的`Pod`中了。

由于服务是运行在minikube中的，`Ingress`并没有外部`IP`访问，所以需要通过以下命令来获取访问地址:
```
export TEST_URL=$(kubectl get po -l istio=ingress -n istio-system -o 'jsonpath={.items[0].status.hostIP}'):$(kubectl get svc istio-ingress -n istio-system -o 'jsonpath={.spec.ports[0].nodePort}')
```
然后通过`curl`命令增加一条数据：
```bash
curl -H 'Content-Type: application/json' -X POST -d '{"username":"a"}' http://$TEST_URL/api/users
```
由于`service1`服务有`v1`及`v2`版本，以上命令需要运行两次，这样两个版本都可以添加到相同的数据。

接下来编写一个脚本用来批量访问服务:
```shell
for i in {1..10}
do
  curl http://$TEST_URL/api/users/1;
  echo "";
done
```
> 如果运行脚本提示没有权限就先用chmod u+x 命令给予权限

运行脚本，查看输出可以看到流量十分均匀的分布至`v1`及`v2`上。

----

### 流量管理

#### 流量切分

编写一个路由规则文件，指定80%的流量访问`v1`，而20%的流量访问到`v2`上:
```yaml
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: service1-v1
spec:
  destination:
    name: service1
  precedence: 1
  route:
  - labels:
      version: v1
    weight: 80
  - labels:
      version: v2
    weight: 20
```
`weight`属性来指定流量的权重，需要注意的是最终权重相加一定要等于100。
`precedence`表示优先度。

通过`istioctl create`命令生成流量规则:
```bash
istio create -f service1-routerule-v1.yaml
```
成功生成之后，可以通过命令`istioctl get routerule -o yaml`方式来查看所有的流量规则。

之后等待一会儿运行一下脚本查看一下结果：
```bash
$ ./request.sh 
Hello, a v1
Hello, a v1
Hello, a v2
Hello, a v1
Hello, a v1
Hello, a v1
Hello, a v2
Hello, a v1
Hello, a v1
Hello, a v1
```
流量大部分都是访问到`v1`版本的`service1`中。规则成功生效。
试验成功之后可以删除这个规则:
```bash
istioctl delete -f service1-routerule-v1.yaml
```
当然也可以通过流量规则名来删除:
```bash
istioctl delete routerule service1-v1

```

#### 错误注入

这里通过`httpFault`属性来注入一些故障。比如设置一个延迟时间为2s,然后50%的流量会返回400错误:
```yaml
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: service1-delay-abort
spec:
  destination:
    name: service1
  route:
  - labels:
      version: v1
  httpFault:
    delay:
      fixedDelay: 2s
    abort:
      percent: 50
      httpStatus: 400
```
通过`istio create`生成规则，然后等待一段时间运行脚本查看结果：
```bash
$ ./request.sh
error
error
Hello, a v1
error
Hello, a v1
Hello, a v1
error
error
error
error
```
最后记得清除规则。

#### 请求超时

在这里将`service1`的服务调用延时2s，然后设置`service2`的请求超时时间为1s：
```yaml
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: service1-delay-timeout
spec:
  destination:
    name: service1
  route:
  - labels:
      version: v1
  httpFault:
    delay:
      percent: 100
      fixedDelay: 2s
---
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: service2-timeout
spec:
  destination:
    name: service2
  route:
  - labels:
      version: v1
  httpReqTimeout:
    simpleTimeout:
      timeout: 1s
```
然后通过`istioctl`创建规则，等待一段时间运行脚本：
```bash
$ ./request.sh
upstream request timeout
upstream request timeout
upstream request timeout
upstream request timeout
upstream request timeout
upstream request timeout
upstream request timeout
upstream request timeout
upstream request timeout
upstream request timeout

```
基本上都出现了超时错误。试验完毕删除规则。

#### 后记

istio还有其他各种强大的流量管理功能比如熔断等等, 不知道是不是运行在minikube中实在是不稳定，经常性的服务访问不到，其他的功能就没试了。目前官网的流量管理规则已经有`v1alpha3`版本了，已经有挺大的变化，`RouteRule`变成了`VirtualService`，`Ingress`还强烈推荐用`Gateway`替代，不过**Github**上边的`v1alpha3`的路由规则在我现在使用的这个版本(0.7.1)中是会出错的，而官网文档的这部分跟**Github**上边也不一致，最终还是用回了`v1alpha2`的版本。

----

### 总结

istio运行下来觉得毕竟还没有到正式版，`api`改动还挺大，而且运行过程中经常服务访问不到，流量规则有时候没有效果，可能是运行环境的问题，还是等到正式版出来再看看效果。

最后如果需要删除istio的组件的话进入到istio的文件夹下运行命令：
```
kubectl delete -f install/kubernetes/istio.yaml
```