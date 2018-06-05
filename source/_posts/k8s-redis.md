---
title: Kubernetes初体验：部署无状态服务Redis
date: 2018-04-13 17:18:02
tags: ["kubernetes"]
categories: ["kubernetes"]
---

最近看微服务相关的文章得知下一代微服务的王牌项目貌似是[istio](https://istio.io/)，而它现阶段又是构建于[Kubernetes](https://kubernetes.io/)上的，那就想着来简单体验一下Kubernetes。

<!-- more -->

**Kubernetes**又称**k8s**，根据官网的介绍，"**Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.**"。这里主要使用官网提供的minikube及kubectl工具在**Ubuntu16.04**上部署一个Kubernetes集群，然后实现官网给出的例子[Reliable, Scalable Redis on Kubernetes](https://github.com/kubernetes/kubernetes/tree/master/examples/storage/redis)

### 安装Kubernetes工具

> [使用minikube在本机搭建kubernetes集群](https://qii404.me/2018/01/06/minukube.html)

首先先安装相关工具`kubectl`，执行[官网](https://kubernetes.io/docs/tasks/tools/install-kubectl/)提供的命令
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl
```
当然还有**MacOS**及**Windows**下的安装方法，可以去[官网](https://kubernetes.io/docs/tasks/tools/install-kubectl/)查看。

接下来是安装`minikube`，执行以下命令:
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.25.2/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
需要**MacOS/Windows**版的可以找一下[latest release](https://github.com/kubernetes/minikube/releases)

`minikube`在**Linux**平台上支持`--vm-driver=none`选项，这个选项可以运行Kubernetes组件在主机的Docker上而不是VM中。

#### 启动
接下来就是启动`minikube`了：
```bash
sudo minikube start --vm-driver=none
```
如果是运行在VM中就不需要后边的选项了。最初运行的时候他会去自动下载所需镜像，等到下载完毕了出现下边的输出则说明启动成功了：
```bash
Starting local Kubernetes v1.9.4 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
===================
WARNING: IT IS RECOMMENDED NOT TO RUN THE NONE DRIVER ON PERSONAL WORKSTATIONS
	The 'none' driver will run an insecure kubernetes apiserver as root that may leave the host vulnerable to CSRF attacks

When using the none driver, the kubectl config and credentials generated will be root owned and will appear in the root home directory.
You will need to move the files to the appropriate location and then set the correct permissions.  An example of this is below:

	sudo mv /root/.kube $HOME/.kube # this will write over any previous configuration
	sudo chown -R $USER $HOME/.kube
	sudo chgrp -R $USER $HOME/.kube
	
	sudo mv /root/.minikube $HOME/.minikube # this will write over any previous configuration
	sudo chown -R $USER $HOME/.minikube
	sudo chgrp -R $USER $HOME/.minikube 

This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
Loading cached images from config file.
```
它给出的命令可以运行一下，影响的结果就是以后如果要执行`kubectl`的相关命令就无需在前边加`sudo`了。

> 如果**minikube**是运行在**VM**中的话可以在开始的时候添加一个选项`--registry-mirror="https://registry.docker-cn.com"`用来设置**Docker**的镜像加速地址,这里使用的是[Docker中国镜像加速](https://www.docker-cn.com/registry-mirror)。

#### 发布一个应用

接下来通过[run命令](http://docs.kubernetes.org.cn/468.html)来生成一个[Deployment](http://docs.kubernetes.org.cn/317.html)
```
kubectl run nginx --image="index.tenxcloud.com/docker_library/nginx" --port=80
```
运行成功将会生成一个`Deployment`，并且自动生成了对应的[Pod](http://docs.kubernetes.org.cn/312.html)。接下来运行命令可以查看到`Pod`的状态：
```bash
kubectl get pods

NAME                    READY     STATUS              RESTARTS   AGE
nginx-b5f754bc5-gnkls   0/1       ContainerCreating   0          1m
```
如果过了一会儿`Pod`的状态还是`ContainerCreating`，那八成是出问题了。由于没有创建成功，自然无法用`kubectl logs [POD_NAME]`命令查看，这时只能查看`minikube`日志:
```bash
sudo minikube logs
```
日志很长，不过其中如果可以看到`failed pulling image "gcr.io/google_containers/pause-amd64:3.0"`这类的信息那就是需要从官网拉取这依赖镜像失败所致。*由于`Kubernetes`的公司是一家在我这连主页都打不开的公司*，自然想要拉取镜像是得用点非常手段了。在这里可以通过[阿里云开发者平台](https://dev.aliyun.com/search.html)来获取对应的镜像，然后打标签为`gcr.io`:
```bash
sudo docker pull registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0 && sudo docker tag registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0
```

> 如果是运行在**VirtualBox**中，可以通过`minikube ssh`命令进入到**minikue**内部运行上边的命令

接下来删除原有的`Deployment`再重新执行上边的命令等一会儿查看一下`Pod`状态:
```bash
kubectl delete deployment nginx
kubectl run nginx --image="index.tenxcloud.com/docker_library/nginx" --port=80
kubectl get pods

NAME                    READY     STATUS    RESTARTS   AGE
nginx-b5f754bc5-gnkls   1/1       Running   0          16m
```
这下它已经成功运行起来了。

接下来需要通过[expose命令](http://docs.kubernetes.org.cn/475.html)将创建好的`Deployment`暴露一个新的[服务](http://docs.kubernetes.org.cn/703.html)
```bash
kubectl expose deployment nginx --name=nginx --type=NodePort
```
然后通过命令查看服务：
```bash
kubectl get services

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        40m
nginx        NodePort    10.105.80.209   <none>        80:31156/TCP   5s
```
由于`TYPE`为`NodePort`，系统会随机分配一个端口给这项服务，这里分配的是31156，在终端可以使用`curl`工具来访问:
```bash
curl $(sudo minikube ip):31156
```
如果能显示出nginx的首页源码则证明服务成功运行。

----

### 部署redis

这里主要是使用了官网提供的例子[Reliable, Scalable Redis on Kubernetes](https://github.com/kubernetes/kubernetes/tree/master/examples/storage/redis)。官网给出的例子用的是[Replication Controller](http://docs.kubernetes.org.cn/437.html),而现在使用的则是[Deployment](http://docs.kubernetes.org.cn/317.html)。

> 如果是运行在**VirtualBox**中，默认给**minikube**分配了一个共享文件夹为`C:\Users`，在**minikube**内部的文件路径为`/c/Users`。如果是在主机上下载了这些文件，可以放到共享文件夹中。[官网参考](https://kubernetes.io/docs/getting-started-guides/minikube/)

#### 创建镜像

首先需要构建镜像，利用官网给出的`image`资源，先修改了`Dockerfile`文件:
```
FROM registry.cn-hangzhou.aliyuncs.com/acs/alpine

RUN apk update
RUN apk upgrade
RUN apk add --no-cache redis sed bash

COPY redis-master.conf /redis-master/redis.conf
COPY redis-slave.conf /redis-slave/redis.conf
COPY run.sh /run.sh

RUN chmod +x /run.sh

CMD [ "/run.sh" ]
ENTRYPOINT [ "bash", "-c" ]
```
`Dockerfile`文件主要用来创建**Docker**镜像的，部分标签含义如下：

 *  `FROM`标签表示创建的镜像，这里用的是`alpine`，一个容量很小的`Linux`发行版。
 *  `RUN`标签用于执行的命令。这里是在`alpine`里安装`redis`。
 *  `CMD`标签用来指定容器启动时执行的命令，这里是需要运行`run.sh`脚本。
 *  `ENTRYPOINT`标签用来配置容器启动后执行的命令。
 
这里跟官方例子不同的地方最主要的是加了这句:
```
RUN chmod +x /run.sh
```
可能是由于我拉取的这个版本的`alpine`用户权限发生了变化，如果不加这句的话创建出来的容器都活不过三秒。`Pod`的状态也一直是`CrashLoopBackOff`。后来使用`kubectl logs POD_NAME`命令想入容器看一下发现说什么`/run.sh permission denied`之类的，大概就明白是权限问题了。加了这句就好了。

然后还修改了`run.sh`脚本的部分：
```
function launchsentinel() {
  while true; do
    ...
    if [[ -n ${master} ]]; then
      master="${master//\"}"
    else
      master=${REDIS_MASTER_SERVICE_HOST}
    fi
    ···
}
```
这里主要是在`redis`哨兵发现主机的判断部分有所修改，我通过将`redis-master`的服务暴露出来，然后获取到服务地址，通过哨兵的`sentinel monitor mymaster ${master} 6379 2`配置监听主机。

接下来`cd`到`Dockerfile`文件目录下运行`docker build`命令创建镜像:
```bash
sudo docker build -t alpine .
```
等创建成功之后可以用`docker images`查看一下镜像:
```
REPOSITORY  TAG     IMAGE ID     CREATED      SIZE
alpine      latest  0b73a4ce0ac4 19 hours ago 11.76 MB
```
对比**Ubuntu**之类的镜像可以说是十分迷你了。

#### 生成Kubernetes资源

接下来是配置生成`redis-master`的文件:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
spec:
  replicas: 1
  selector:
    matchLabels:
      redis-master: "true"
  template:
    metadata:
      labels:
        redis-master: "true"
    spec:
      containers:
      - name: redis-master
        image: alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 6379
        env:
        - name: MASTER
          value: "true"
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
      volumes:
      - name: data
        emptyDir: {}
```
这里生成了一个名为`redis-master`的`Deployment`，使用的镜像则是上边生成的`alpine`，暴露的端口则是**redis**默认的端口6379,这里传入的环境变量为`MASTER=true`，在`run.sh`中就会运行`launchmaster`方法,将**redis**角色设置为主服务器。运行以下命令创建`Deployment`：
```bash
kubectl create -f redis-master-deployment.yaml
```
然后查看一下：
```bash
kubectl get deployments
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
redis-master   1         1         1            1           5s
```
接下来将它发布成一个新的服务:
```bash
kubectl expose deployment redis-master --name=redis-master --port=6379 --target-port=6379
```

**Redis-master**创建成功之后，就可以来创建**redis**哨兵服务了。通过命令创建:
```bash
 kubectl run redis-sentinel --image=alpine --image-pull-policy=IfNotPresent --port=26379 --expose=true --replicas=3 --env="SENTINEL=true"
```
通过[run命令](http://docs.kubernetes.org.cn/468.html)生成一个名为`redis-sentinel`的`Deployment`，部分参数含义:

* `image`：指定镜像（**alpine**）
* `image-pull-policy`：镜像拉取策略，**IfNotPresent**表示优先从本地获取
* `env`：设置容器中的环境变量，`SENTINEL=true`则是让`alpine`的`run.sh`进入`launchsentinel`方法将**redis**的角色变为`sentinel`
* `expose`：将生成的`Deployment`自动生成一个服务
* `port`: 指定端口,同样用于生成的服务端口。
* `replicas`： 生成多少个副本

接下来可以查看一下已经生成的服务:
```bash
kubectl get services
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
redis-master     ClusterIP   10.98.123.202   <none>        6379/TCP       2h
redis-sentinel   ClusterIP   10.109.167.66   <none>        26379/TCP      2h
```
然后查看一下生成的`pod`:
```bash
kubectl get pods -o wide

NAME                              READY     STATUS    RESTARTS   AGE       IP
nginx-b5f754bc5-gnkls             1/1       Running   0          3h        172.17.0.4
redis-master-8485d87c4c-kpbfd     1/1       Running   0          2h        172.17.0.5
redis-sentinel-5655765c58-n455l   1/1       Running   0          2h        172.17.0.8
redis-sentinel-5655765c58-scrgw   1/1       Running   0          2h        172.17.0.7
redis-sentinel-5655765c58-zdqxx   1/1       Running   0          2h        172.17.0.6
```

**Kubernetes**的服务发现支持两种方式：环境变量与DNS，这里使用的是环境变量的方式。
随机查看一个**redis-sentinel**容器的环境变量:
```bash
kubectl exec -it redis-sentinel-5655765c58-n455l env|grep REDIS

REDIS_MASTER_PORT_6379_TCP_ADDR=10.98.123.202
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_SENTINEL_SERVICE_HOST=10.109.167.66
REDIS_MASTER_SERVICE_PORT=6379
REDIS_SENTINEL_PORT=tcp://10.109.167.66:26379
REDIS_SENTINEL_PORT_26379_TCP=tcp://10.109.167.66:26379
REDIS_MASTER_SERVICE_HOST=10.98.123.202
REDIS_MASTER_PORT_6379_TCP=tcp://10.98.123.202:6379
REDIS_SENTINEL_SERVICE_PORT=26379
REDIS_SENTINEL_PORT_26379_TCP_PROTO=tcp
REDIS_SENTINEL_PORT_26379_TCP_ADDR=10.109.167.66
REDIS_SENTINEL_PORT_26379_TCP_PORT=26379
REDIS_MASTER_PORT=tcp://10.98.123.202:6379
```
在这里我们可以看到生成的**redis-master**及**redis-sentinel**服务的**IP**及端口都已经写入到了容器中的环境变量中，分别是`REDIS_MASTER_SERVICE_HOST`及`REDIS_SENTINEL_SERVICE_HOST`。所以在`run.sh`中需要做对应的修改就明白怎么一回事了：
```bash
function launchsentinel() {
  while true; do
    master=$(redis-cli -h ${REDIS_SENTINEL_SERVICE_HOST} -p ${REDIS_SENTINEL_SERVICE_PORT} --csv SENTINEL get-master-addr-by-name mymaster | tr ',' ' ' | cut -d' ' -f1)
    if [[ -n ${master} ]]; then
      master="${master//\"}"
    else
      master=${REDIS_MASTER_SERVICE_HOST}
    fi

    redis-cli -h ${master} INFO
    if [[ "$?" == "0" ]]; then
      break
    fi
    echo "Connecting to master failed.  Waiting..."
    sleep 10
  done

  sentinel_conf=sentinel.conf

  echo "sentinel monitor mymaster ${master} 6379 2" > ${sentinel_conf}
  ···
}
```

这里首先是通过连接`REDIS_SENTINEL_SERVICE_HOST`的**redis-sentinel**服务来获取对应的**IP**，由于一开始服务并没有运行，所以这里会连接超时。失败了之后就指定`REDIS_MASTER_SERVICE_HOST`为**主Redis**服务器，接着通过`sentinel monitor mymaster ${master} 6379 2`配置**redis sentinel**去监听主服务器，判断这个主服务器失效至少需要2个**sentinel**同意。

> 这里如果是使用DNS的方式来访问服务可以通过`<service name>.<namespace>.svc.cluster.local`类似的地址来访问。

接下来通过`logs`命令来查看**sentinel**有没有成功运行起来，这里需要等待一会儿:
```bash
kubectl logs redis-sentinel-5655765c58-n455l

···
···# Sentinel ID is 2e8d3d57dbfe690e242a16397f5bcbcf6e3cfae1
···# +monitor master mymaster 10.98.123.202 6379 quorum 2
···
```
如果没有错误日志输出则说明**sentinel**服务已成功运行，接下来就创建**redis slave**服务吧:
```bash
kubectl run redis-slave --image=alpine --image-pull-policy=IfNotPresent --port=6379 --replicas=2
```
如果显示创建成功，可以查看一下`Pod`状态：
```bash
kubectl get pods -o wide

NAME                              READY     STATUS    RESTARTS   AGE       IP
redis-master-8485d87c4c-kpbfd     1/1       Running   0          2h        172.17.0.5
redis-sentinel-5655765c58-n455l   1/1       Running   0          2h        172.17.0.8
redis-sentinel-5655765c58-scrgw   1/1       Running   0          2h        172.17.0.7
redis-sentinel-5655765c58-zdqxx   1/1       Running   0          2h        172.17.0.6
redis-slave-d888d4974-m97wl       1/1       Running   0          3s        172.17.0.10
redis-slave-d888d4974-nvh9h       1/1       Running   0          3s        172.17.0.9
```

看样子整个**redis sentinel**服务已经建立起来了，接下来就来测试一下能不能成功了。在`Ubuntu 16.04`中倒是自带了`Python`环境，就使用它来测试一下。
首先需要运行命令`pip install redis`下载依赖，如果提示没有`pip`就跟着提示使用`apt`下载一个吧。然后随便写个测试:
```
>>> import redis
>>> from redis.sentinel import Sentinel
>>> sentinel = Sentinel([("10.109.167.66", 26379)], socket_timeout=0.1)
>>> print sentinel.discover_master('mymaster')
('10.98.123.202', 6379)
>>> print sentinel.discover_slaves('mymaster')
[('172.17.0.10', 6379), ('172.17.0.9', 6379)]
>>> master = sentinel.master_for('mymaster', socket_timeout=0.1)
>>> slave = sentinel.slave_for('mymaster', socket_timeout=0.1)
>>> master.set('hello', 'world')
True
>>> slave.get('hello')
'world'
>>> 
```
这里通过**redis-sentinel服务**提供的**IP/端口**连接，然后可以看到**redis**的主从服务器配置。这样看整个**Redis Sentinel主从服务**算是部署成功了吧。

#### 删除资源

最后如果需要删除上述创建的一切资源可以使用`delete`命令删除:
```
kubectl delete services,deployments,pods --all
```

----

### Kubernetes控制面板

如果这家公司的另一个产品**Tensorflow**提供了一个`tensorboard`，**Kubernetes**也有一个可视化界面。通过以下命令运行:
```
sudo minikube dashboard
```
首次运行，我估计十有八九都不会成功，应该都是一直显示`Waiting, endpoint for service is not ready yet...`这句日志，查看一下日志当然又是跟`pause`镜像一样的问题
```
sudo minikube logs
```
由于**dashboard**依赖的镜像比较多，可以使用以下命令列举出来:
```
sudo grep -r 'image:' /etc/kubernetes/

/etc/kubernetes/addons/storage-provisioner.yaml:    image: gcr.io/k8s-minikube/storage-provisioner:v1.8.1
/etc/kubernetes/addons/kube-dns-controller.yaml:        image: k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.5
/etc/kubernetes/addons/kube-dns-controller.yaml:        image: k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.5
/etc/kubernetes/addons/kube-dns-controller.yaml:        image: k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.5
/etc/kubernetes/addons/dashboard-dp.yaml:        image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.1
/etc/kubernetes/manifests/addon-manager.yaml:    image: gcr.io/google-containers/kube-addon-manager:v6.5
```
接下来还是得使用打标签大法将这些镜像自己拉取下来：
```
sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-addon-manager:v6.5 && sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-addon-manager:v6.5 gcr.io/google-containers/kube-addon-manager:v6.5

sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner:v1.8.1 && sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner:v1.8.1 gcr.io/k8s-minikube/storage-provisioner:v1.8.1

sudo docker pull registry.cn-hangzhou.aliyuncs.com/google-containers/kubernetes-dashboard-amd64:v1.7.1 && sudo docker tag registry.cn-hangzhou.aliyuncs.com/google-containers/kubernetes-dashboard-amd64:v1.7.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.1

sudo docker pull registry.cn-shenzhen.aliyuncs.com/gcrio/k8s-dns-kube-dns-amd64:latest && sudo docker tag registry.cn-shenzhen.aliyuncs.com/gcrio/k8s-dns-kube-dns-amd64:latest k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.5

sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.5 && sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.5 k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.5

sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-sidecar-amd64:1.14.5 && sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-sidecar-amd64:1.14.5 k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.5
```

成功拉取下对应的镜像并且打好标签之后重新运行一下**minikube**:
```
sudo minikube stop
sudo minikube start --vm-driver=none
```
成功运行起来之后再次运行
```
sudo minikube dashboard
```
如果运气好能够成功弹出浏览器，那就可以去看一下控制面板的内容了。
而我偏偏属于运气不好的，控制台还是一直在输出`Waiting, endpoint for service is not ready yet...`。只好去查看一下这些**Pod**的状态:
```
kubectl get pods --namespace=kube-system
```
发现那些什么`dashboard`、`storage-provisioner`的状态都是`CrashLoopBackOff`(已经见到它好几次了)，通过日志才发现估计是`secret`的问题，删除让系统自动生成就好了[github issue](https://github.com/kubernetes/minikube/issues/288):
```
kubectl delete secrets --namespace=kube-system --all
kubectl delete pods --namespace=kube-system --all
```
然后通过命令重启一下`minikube`，这个时候首先看一下**dashboard**的那几个**pod**的状态:
```
kubectl get pods --namespace=kube-system

NAME                                    READY     STATUS    RESTARTS   AGE
kube-addon-manager-ezio-virtualbox      1/1       Running   1          5h
kube-dns-54cccfbdf8-gzrdg               3/3       Running   0          5h
kubernetes-dashboard-77d8b98585-lgl54   1/1       Running   0          5h
storage-provisioner                     1/1       Running   0          5h
```

看到他们都已经成功运行了，再运行一下`sudo minikube dashboard`命令，终于算是成功打开浏览器并且访问到**Kubernetes**的控制面板了。