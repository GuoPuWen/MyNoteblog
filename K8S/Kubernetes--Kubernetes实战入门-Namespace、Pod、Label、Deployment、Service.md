了解了K8s中的资源管理方式之后，本篇文章将会介绍如何在K8s中部署一个Nginx服务，并且让外部可以进行访问，将会介绍Namespace、Pod、Deployment、Service这些资源

# Namespace

Namespace是一种在多个用户之间划分集群资源的方法，就是说可以实现多套环境的资源隔离。因为在默认情况下，K8s集群中的所有pod都是可以互相通信的，但是在实际生产环境下，可能不允许两个pod之间进行通信，那么就可以使用Namespace，将两个pod划分到两个Namespace下，可以形成逻辑上的组别的概念，做到资源的隔离。

Namespace还可以做到资源配额，通过K8s的授权机制，将不同的Namespace交给不同的用户进行管理，可以限定不同用户所占用的资源，例如CPU使用量，内存使用量这些，来实现租户可用资源的管理

![image-20210713171008898](http://cdn.noteblogs.cn/image-20210713171008898.png)

### 查看命名空间

```shell
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   26h
kube-node-lease   Active   26h
kube-public       Active   26h
kube-system       Active   26h

[root@master ~]# kubectl get ns default -o yaml		#指定输出格式
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-07-12T06:15:43Z"
  name: default
  resourceVersion: "151"
  selfLink: /api/v1/namespaces/default
  uid: 24805218-aa2c-41a2-9247-3535f3c075db
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
  
[root@master ~]# kubectl describe ns default # 查看更详细的信息
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.	
```

K8s集群一搭建会给我们创建4个初始的命名空间：

- `default` 没有其他命名空间的对象的默认命名空间
- `kube-system` Kubernetes 系统创建的对象的命名空间
- `kube-public`此命名空间是自动创建的，所有用户（包括未通过身份验证的用户）均可读取。这个命名空间主要是为集群使用保留的，以防某些资源在整个集群中公开可见和可读。这个命名空间的公共方面只是一个约定，而不是一个要求。
- `kube-node-lease` 与每个节点关联的租用对象的这个命名空间，随着集群的扩展，它提高了节点心跳的性能。

### 创建与删除

```shell
[root@master ~]# kubectl create ns dev
namespace/dev created
[root@master ~]# kubectl delete ns dev
namespace "dev" deleted
# 创建pod在置顶的命名空间
[root@master ~]# kubectl run nginx01 --image=nginx --namespace dev
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx01 created
#查看信息
[root@master ~]# kubectl get pods -n dev
NAME                       READY   STATUS    RESTARTS   AGE
nginx01-69d4c7c56f-2tmfc   1/1     Running   0          59s
```

### 配置方式

ns-dev.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```shell
[root@master ~]# kubectl create -f ns-dev.yaml 
namespace/dev created
[root@master ~]# kubectl get namespace
NAME              STATUS   AGE
default           Active   27h
dev               Active   10s
kube-node-lease   Active   27h
kube-public       Active   27h
kube-system       Active   27h
[root@master ~]# kubectl delete -f ns-dev.yaml 
```

# Pod

Pod是K8s中的最小管理单元，程序必须部署在容器中，而容器必须存在于pod中，一个pod可以存在一个或者多个容器，在Pod中也存在默认的容器，这个容器在命令行界面是不显示的

![image-20210713173159343](http://cdn.noteblogs.cn/image-20210713173159343.png)

在Namespace中存在四个默认的命名空间，可以试着查看一下这些命名空间存在的pod

```shell
[root@master ~]# kubectl get pods -n kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-6955765f44-4ljg2         1/1     Running   0          27h
coredns-6955765f44-rv6qv         1/1     Running   0          27h
etcd-master                      1/1     Running   0          27h
kube-apiserver-master            1/1     Running   0          27h
kube-controller-manager-master   1/1     Running   0          27h
kube-flannel-ds-4b8hz            1/1     Running   0          21h
kube-flannel-ds-9m8b9            1/1     Running   0          21h
kube-flannel-ds-prdlj            1/1     Running   0          21h
kube-proxy-gmxzj                 1/1     Running   0          27h
kube-proxy-j6wh8                 1/1     Running   0          27h
kube-proxy-xrbxv                 1/1     Running   0          27h
kube-scheduler-master            1/1     Running   0          27h
```

在第一篇介绍K8s的架构的时候，就已经知道了K8s中的一些组件，现在可以发现这些组件也是以Pod的方式运行的

### 创建与删除

需要注意：k8s没有提供单独运行pod的命名，都是通过pod控制来实现的，pod控制器就是对pod进行管理的组件，当使用`kubectl run	`创建的都是pod控制器。正是因为创建的是pod控制器，所以在删除指定的pod的时候，删除完了又创建，这些下面会说到

```shell
 [root@master ~]# kubectl run nginx01 --image=nginx --port=80 -n dev
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx01 created
```

上面在dev的命名空间内创建了一个nginx01的pod控制器，通过输出的日志可也可以发现创建的是deployment，接收控制器

```shell
[root@master ~]# kubectl get pod -n dev
NAME                     READY   STATUS    RESTARTS   AGE
nginx01-7744bfb8-ngvhv   1/1     Running   0          2m19s

#查看pod的详细信息
[root@master ~]# kubectl describe pod nginx01-7744bfb8-ngvhv -n dev
Name:         nginx01-7744bfb8-ngvhv
Namespace:    dev
Priority:     0
Node:         node2/10.0.0.6
Start Time:   Tue, 13 Jul 2021 17:37:51 +0800
Labels:       pod-template-hash=7744bfb8
              run=nginx01
Annotations:  <none>
Status:       Running
IP:           10.244.2.7
IPs:
  IP:           10.244.2.7
Controlled By:  ReplicaSet/nginx01-7744bfb8
Containers:
  nginx01:
    Container ID:   docker://ad1883d2354f6c2eb4b4f2d89c4ad568245c4a2df69d9e697270e24f6fbbe381
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:353c20f74d9b6aee359f30e8e4f69c3d7eaea2f610681c4a95849a2fd7c497f9
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 13 Jul 2021 17:38:14 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8mfq4 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-8mfq4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8mfq4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m53s  default-scheduler  Successfully assigned dev/nginx01-7744bfb8-ngvhv to node2
  Normal  Pulling    2m53s  kubelet, node2     Pulling image "nginx"
  Normal  Pulled     2m31s  kubelet, node2     Successfully pulled image "nginx"
  Normal  Created    2m31s  kubelet, node2     Created container nginx01
  Normal  Started    2m30s  kubelet, node2     Started container nginx01
  
# 删除pod

[root@master ~]# kubectl get pods -n dev
NAME                     READY   STATUS    RESTARTS   AGE
nginx01-7744bfb8-ngvhv   1/1     Running   0          4m49s
[root@master ~]# kubectl delete pod nginx01-7744bfb8-ngvhv -n dev
pod "nginx01-7744bfb8-ngvhv" deleted
[root@master ~]# kubectl get pods -n dev
NAME                     READY   STATUS              RESTARTS   AGE
nginx01-7744bfb8-wkvrd   0/1     ContainerCreating   0          6s
```

可以发现，在删除了指定的pod之后，很快又重新创建了该pod，因为pod是有pod控制器进行控制的，当控制器发现pod被删除之后，又进行重新创建了，要删除该pod可以删除该pod对应的pod控制器，这在下面中会介绍到

### 配置方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx02
  namespace: dev
spec:
  containers:
  - image: nginx:1.17.1
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```

    - 创建：kubectl  create  -f  pod-nginx.yaml
    - 删除：kubectl  delete  -f  pod-nginx.yaml

使用这种方式创建的是pod而不是pod控制器

```shell
[root@master ~]# kubectl create -f pod-nginx.yaml 
pod/nginx02 created
```

那么可以直接进行pod的删除

# Label

Label标签的作用是在资源上添加标识，用来对资源进行区分和选择，可能会有疑惑：不是已经有Namespace命名空间了吗，还需要标签吗？Namespace命名空间会让两组pod之间无法通信，但是如果有需要让pod之间正常通信但是又想加以区别，这个时候就需要用到标签

标签的特点是：

- 键值对，一个Label会以键值对的方式附加到各种对象上，例如Node、Pod、Service等等
- 一个资源对象可以定义任意数量的Label，同一个Label也可以添加到任意数量的资源对象上去
- Label可以在资源对象定义时被确定，也可以在对象创建之后动态的添加和删除

一些常用的标签有：

```shell
"release" : "stable", "release" : "canary"
"environment" : "dev", "environment" : "qa", "environment" : "production"
"tier" : "frontend", "tier" : "backend", "tier" : "cache"
"partition" : "customerA", "partition" : "customerB"
"track" : "daily", "track" : "weekly"
```

### 标签选择器

标签选择器用于选择标签，目前有两种标签选择器

- 基于等式的Label Selector

  name = slave: 选择所有包含Label中key="name"且value="slave"的对象

  env != production: 选择所有包括Label中的key="env"且value不等于"production"的对象

- 基于集合的Label Selector

  name in (master, slave): 选择所有包含Label中的key="name"且value="master"或"slave"的对象

  name not in (frontend): 选择所有包含Label中的key="name"且value不等于"frontend"的对象

### 使用

```shell
# 查看当前dev命名空间的pod
[root@master ~]# kubectl get pods -n dev
NAME                     READY   STATUS    RESTARTS   AGE
nginx01-7744bfb8-wkvrd   1/1     Running   0          16h
nginx02                  1/1     Running   0          16h
#给pod打上标签
[root@master ~]# kubectl label pod nginx01-7744bfb8-wkvrd version=01 -n dev
pod/nginx01-7744bfb8-wkvrd labeled
#查看标签
[root@master ~]# kubectl get pods -n dev --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx01-7744bfb8-wkvrd   1/1     Running   0          16h   pod-template-hash=7744bfb8,run=nginx01,version=01
nginx02                  1/1     Running   0          16h   <none>
#覆盖标签
[root@master ~]# kubectl label pod nginx01-7744bfb8-wkvrd version=02 -n dev --overwrite
pod/nginx01-7744bfb8-wkvrd labeled
#查看标签
[root@master ~]# kubectl get pods -n dev --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx01-7744bfb8-wkvrd   1/1     Running   0          16h   pod-template-hash=7744bfb8,run=nginx01,version=02
nginx02                  1/1     Running   0          16h   <none>
#筛选标签
[root@master ~]# kubectl get pod -n dev -l version=02 --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx01-7744bfb8-wkvrd   1/1     Running   0          16h   pod-template-hash=7744bfb8,run=nginx01,version=02
```

### 配置方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
  labels:
    version: "3.0" 
    env: "test"
spec:
  containers:
  - image: nginx:1.17.1
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```

kubectl  apply  -f  pod-nginx.yaml即可

# Deployment

在K8s中，pod是最小的控制单元，但是K8s一般很少去直接控制Pod而是通过Pod控制器来完成的，Pod控制器确保Pod资源符合预期的状态，当Pod资源出现故障的时候，会尝试进行重启或者重建Pod，在K8s中有很多类型的Pod控制器，这里介绍Deployment

其实前面也说过，直接使用`kubectl run nginx`这样默认创建的就是Deployment控制器，例如：

```shell
#创建Pod控制器 --replicas=3表示副本数为3
[root@master ~]# kubectl run nginx03 --image=nginx --port=80 --replicas=3 -n dev 
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx03 created

#查看dev环境下的pod
[root@master ~]# kubectl get pods -n dev
NAME                       READY   STATUS    RESTARTS   AGE
nginx01-7744bfb8-wkvrd     1/1     Running   0          16h
nginx02                    1/1     Running   0          16h
nginx03-84db5b6896-spkpx   1/1     Running   0          71s
nginx03-84db5b6896-tgk6l   1/1     Running   0          71s
nginx03-84db5b6896-tmkbb   1/1     Running   0          71s

# 查看deploy的信息  
[root@master ~]# kubectl get deploy -n dev
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
nginx01   1/1     1            1           17h
nginx03   3/3     3            3           12m
#更详细的信息
[root@master ~]# kubectl get deploy -n dev -o wide
NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
nginx01   1/1     1            1           17h   nginx01      nginx    run=nginx01
nginx03   3/3     3            3           13m   nginx03      nginx    run=nginx03
```

### 配置方式

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.17.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
```

```shell
kubectl  create  -f  deploy-nginx.yaml
```

# Service

Deployment可以用来创建一组Pod来提供高可用的服务，但是每一个Pod都会提分配一个单独的Ip，这样通过ip访问就会出现问题：

- 如果一组为前端的Pod需要调用后端的一组Pod，但是因为后端的每一个Pod都有自己的ip地址，那么如何实现找到对应的ip地址以及负载均衡的问题？

- Pod IP只是提供的集群内可见的虚拟IP，外部无法访问

![image-20210714105656655](http://cdn.noteblogs.cn/image-20210714105656655.png)

```shell
#查找当前的deployment
[root@master ~]# kubectl get deploy -n dev
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
nginx01   1/1     1            1           17h
nginx03   3/3     3            3           26m
#service的缩写是svc，通过上图可以看到service服务是需要通过deploy来管理的
[root@master ~]# kubectl expose deploy nginx01 --name=svc-nginx03 --type=ClusterIP --port=80 --target-port=80 -n dev 
service/svc-nginx03 exposed
[root@master ~]# kubectl get svc -n dev
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
svc-nginx03   ClusterIP   10.98.227.74   <none>        80/TCP    18s
[root@master ~]# curl 10.98.227.74
```

上面创建的还只是集群内部可以访问的，创建集群外部的也可以访问的service，只需要改变类型即可

```shell
[root@master ~]# kubectl expose deploy nginx03 --name=svc-nginx02 --type=NodePort --port=80 --target-port=80 -n devservice/svc-nginx02 exposed
[root@master ~]# kubectl get svc -n dev
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
svc-nginx02   NodePort    10.103.139.151   <none>        80:31036/TCP   14s
svc-nginx03   ClusterIP   10.98.227.74     <none>        80/TCP         7m2s
#删除serivce
[root@master ~]# kubectl delete svc svc-nginx03 -n dev
service "svc-nginx03" deleted
```

通过虚拟机ip:31036变可以外部访问

