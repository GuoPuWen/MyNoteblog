# Pod介绍

K8s中最小的运行单位是Pod，容器必须放在Pod里面才可以运行，而K8s中的Pod中的容器分为两类：

![image-20210714144237877](http://cdn.noteblogs.cn/image-20210714144237877.png)

- 用户自己定义的容器
- Pause容器，这个是每一个Pod都会有的一个容器

设置Pause容器有以下两个好处：

- 在一组容器作为一个单元运行的情况下，我们难以对Pod整体的运行情况进行判定，而引入一个与业务无关的Pause容器作为Pod的根容器，用它的状态代表整体容器的运行状态
- Pod里面的多个容器共享Pause容器的IP'，恭喜Pause容器挂载的Volume，这样使得Pod之间的容器可以互相通信，也很好的解决了文件共享问题

> K8s为每一个Pod都分配类一个IP，成为Pod IP，一个Pod里的多个容器共享Pod IP地址，K8s要求底层网络支持集群内任意两个Pod之间进行通信，一般采用虚拟网络技术进行实现，所以一个Pod里面的容器与另外一个主机上的Pod容器能够直接通信

# Kubectl explain

配置一个资源，可以使用三种方式命令式对象管理、命令式对象配置、声明式对象配置，而后面两种配置方式是比较常用的，这就需要编写一个ymal配置文件，配置文件里面可配置的选项很多，常用的有以下：

```yaml
apiVersion: v1     #必选，版本号，例如v1
kind: Pod       　 #必选，资源类型，例如 Pod
metadata:       　 #必选，元数据
  name: string     #必选，Pod名称
  namespace: string  #Pod所属的命名空间,默认为"default"
  labels:       　　  #自定义标签列表
    - name: string      　          
spec:  #必选，Pod中容器的详细定义
  containers:  #必选，Pod中容器列表
  - name: string   #必选，容器名称
    image: string  #必选，容器的镜像名称
    imagePullPolicy: [ Always|Never|IfNotPresent ]  #获取镜像的策略 
    command: [string]   #容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]      #容器的启动命令参数列表
    workingDir: string  #容器的工作目录
    volumeMounts:       #挂载到容器内部的存储卷配置
    - name: string      #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean #是否为只读模式
    ports: #需要暴露的端口库号列表
    - name: string        #端口的名称
      containerPort: int  #容器需要监听的端口号
      hostPort: int       #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string    #端口协议，支持TCP和UDP，默认TCP
    env:   #容器运行前需设置的环境变量列表
    - name: string  #环境变量名称
      value: string #环境变量的值
    resources: #资源限制和请求的设置
      limits:  #资源限制的设置
        cpu: string     #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string  #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests: #资源请求的设置
        cpu: string    #Cpu请求，容器启动的初始可用数量
        memory: string #内存请求,容器启动的初始可用数量
    lifecycle: #生命周期钩子
		postStart: #容器启动后立即执行此钩子,如果执行失败,会根据重启策略进行重启
		preStop: #容器终止前执行此钩子,无论结果如何,容器都会终止
    livenessProbe:  #对Pod内各容器健康检查的设置，当探测无响应几次后将自动重启该容器
      exec:       　 #对Pod容器内检查方式设置为exec方式
        command: [string]  #exec方式需要制定的命令或脚本
      httpGet:       #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:     #对Pod内个容器健康检查方式设置为tcpSocket方式
         port: number
       initialDelaySeconds: 0       #容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0    　　    #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
       periodSeconds: 0     　　    #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged: false
  restartPolicy: [Always | Never | OnFailure]  #Pod的重启策略
  nodeName: <string> #设置NodeName表示将该Pod调度到指定到名称的node节点上
  nodeSelector: obeject #设置NodeSelector表示将该Pod调度到包含这个label的node上
  imagePullSecrets: #Pull镜像时使用的secret名称，以key：secretkey格式指定
  - name: string
  hostNetwork: false   #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
  volumes:   #在该pod上定义共享存储卷列表
  - name: string    #共享存储卷名称 （volumes类型有很多种）
    emptyDir: {}       #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
    hostPath: string   #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
      path: string      　　        #Pod所在宿主机的目录，将被用于同期中mount的目录
    secret:       　　　#类型为secret的存储卷，挂载集群与定义的secret对象到容器内部
      scretname: string  
      items:     
      - key: string
        path: string
    configMap:         #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
      name: string
      items:
      - key: string
        path: string
```

上面的配置选项，可以通过分层级来记忆同时搭配`Kubectl explain`，例如

```shell
[root@master ~]# kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status       <Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```

上面已经列出来了pod中的一级配置，主要包含5个部分同时也给出了网址可以查看详细信息：

- apiVersion   \<string>     版本，由kubernetes内部定义，版本号必须可以用 kubectl api-versions 查询到
- kind \<string>                类型，由kubernetes内部定义，版本号必须可以用 kubectl api-resources 查询到

- metadata   \<Object>     元数据，主要是资源标识和说明，常用的有name、namespace、labels等

- spec \<Object>               描述，这是配置中最重要的一部分，里面是对各种资源配置的详细描述                

- status  \<Object>            状态信息，里面的内容不需要定义，由kubernetes自动生成

可以继续使用`Kubectl explain pod.spec`查看spec下的配置选项，常用的有以下几个选项：

- containers   <[]Object>       容器列表，用于定义容器的详细信息 
- nodeName \<String>           根据nodeName的值将pod调度到指定的Node节点上
- nodeSelector   <map[]>      根据NodeSelector中定义的信息选择将该Pod调度到包含这些label的Node 上
- hostNetwork  \<boolean>    是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
- volumes      <[]Object>       存储卷，用于定义Pod上面挂在的存储信息 
- restartPolicy	\<string>       重启策略，表示Pod在遇到故障的时候的处理策略

# Pod配置

上面通过`kubectl explain`查看到了Pod配置的一些选项，其中最重要的是`pod.spec.contains`属性，这是配置Pod中最为关键的一项配置

```shell
[root@master ~]# kubectl explain pod.spec.containers
KIND:     Pod
VERSION:  v1
RESOURCE: containers <[]Object>   # 数组，代表可以有多个容器
FIELDS:
   name  <string>     # 容器名称
   image <string>     # 容器需要的镜像地址
   imagePullPolicy  <string> # 镜像拉取策略 
   command  <[]string> # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
   args     <[]string> # 容器的启动命令需要的参数列表
   env      <[]Object> # 容器环境变量的配置
   ports    <[]Object>     # 容器需要暴露的端口号列表
   resources <Object>      # 资源限制和资源请求的设置
```

### 基本配置

```yaml
apiVersion: v1
kind: Pod
metadata:
        name: pod-base
        namespace: dev
        labels:
                user: root
spec:
        containers:
        - name: nginx
          image: nginx:1.17.1
        - name: busybox
          image: busybox:1.30 
```

上面定义了一个比较简单Pod的配置，里面有两个容器：

- nginx：用1.17.1版本的nginx镜像创建，（nginx是一个轻量级web容器）
- busybox：用1.30版本的busybox镜像创建，（busybox是一个小巧的linux命令集合）

```shell
[root@master ~]# kubectl create -f  pod-base.yaml 
pod/pod-base created
[root@master ~]# kubectl get pod -n dev
NAME       READY   STATUS             RESTARTS   AGE
pod-base   1/2     CrashLoopBackOff   1          5s
[root@master ~]# kubectl get pod -n dev
NAME       READY   STATUS             RESTARTS   AGE
pod-base   1/2     CrashLoopBackOff   1          9s
```

查看启动状态的时候，可以发现只启动了一个容器，这个问题在docker中也出现过，因为启动的容器是busybox里没有一个后台进程，所以pod控制器会一直尝试重启这个容器

### 镜像拉取策略

imagePullPolicy，用于设置镜像拉取策略，kubernetes支持配置三种拉取策略：

- Always：总是从远程仓库拉取镜像（一直远程下载）
- IfNotPresent：本地有则使用本地镜像，本地没有则从远程仓库拉取镜像（本地有就本地  本地没远程下载）
- Never：只使用本地镜像，从不去远程仓库拉取，本地没有就报错 （一直使用本地）

> 默认值说明：
>
> ​    如果镜像tag为具体版本号， 默认策略是：IfNotPresent
>
> ​	如果镜像tag为：latest（最终版本） ，默认策略是always

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-imagepullpolicy
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    imagePullPolicy: Always # 用于设置镜像拉取策略
  - name: busybox
    image: busybox:1.30
```

### 启动命令

command配置可以让容器在启动的时候，执行这个命令例如可以在busybox启动的时候运行一个死循环的进程，便可以busybox一直在后台启动

```shell
apiVersion: v1
kind: Pod
metadata:
  name: pod-command
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","touch /tmp/hello.txt;while true;do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;"]
```

```shell
[root@master ~]# kubectl create -f pod-command.yaml 
pod/pod-command created
[root@master ~]# 
[root@master ~]# 
[root@master ~]# kubectl get pod -n dev
NAME                  READY   STATUS             RESTARTS   AGE
pod-base              1/2     CrashLoopBackOff   10         26m
pod-command           2/2     Running            0          7s
pod-imagepullpolicy   1/2     CrashLoopBackOff   8          17m
```

可以使用`kubectl exec	`进入到容器内部查看

```shell
[root@master ~]# kubectl exec pod-command -n dev -it -c busybox /bin/sh
```

退出可以使用ctrl + p + q

这里可以简单的回顾一下Dockerfile中的ENTRYPOINT与CMD命令之间的区别：

- 当使用docker run的时候如果带有其他参数则不会覆盖ENTRYPOINT指令
- 当使用docker run的时候如果带有其他参数，将会覆盖CMD指令

同时在配置项里面还出现了一个args选项，用于传递参数，args选项和command选项可以实现覆盖Dockerfile中的ENTRYPOINT功能

>  1 如果command和args均没有写，那么用Dockerfile的配置。
>  2 如果command写了，但args没有写，那么Dockerfile默认的配置会被忽略，执行输入的command
>  3 如果command没写，但args写了，那么Dockerfile中配置的ENTRYPOINT的命令会被执行，使用当前args的参数
>  4 如果command和args都写了，那么Dockerfile的配置被忽略，执行command并追加上args参数

### 环境变量

使用env配置项可以设置容器的环境变量

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-env
  namespace: dev
spec:
  containers:
  - name: busybox
    image: busybox:1.30
    command: ["/bin/sh","-c","while true;do /bin/echo $(date +%T);sleep 60; done;"]
    env: # 设置环境变量列表
    - name: "username"
      value: "admin"
    - name: "password"
      value: "123456"
```

```shell
[root@master ~]# kubectl create -f pod-env.yaml 
pod/pod-env created
[root@master ~]# kubectl exec pod-env -it -n dev -c busybox /bin/sh
/ # 
/ # echo $username
admin
```

### 端口设置

使用containers中的ports选项可以配置端口

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-ports
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports: # 设置容器暴露的端口列表
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```

```shell
[root@master ~]# vim pod-ports.yaml
[root@master ~]#  kubectl create -f pod-ports.yaml
pod/pod-ports created
[root@master ~]# kubectl get pods 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6867cdf567-lt9wz   1/1     Running   0          44h
[root@master ~]# kubectl get pods -n dev
NAME                  READY   STATUS             RESTARTS   AGE
pod-base              1/2     CrashLoopBackOff   14         48m
pod-command           2/2     Running            0          21m
pod-env               1/1     Running            0          6m56s
pod-imagepullpolicy   1/2     CrashLoopBackOff   12         39m
pod-ports             1/1     Running            0          89s
[root@master ~]# kubectl get pods pod-ports  -n dev
NAME        READY   STATUS    RESTARTS   AGE
pod-ports   1/1     Running   0          111s
[root@master ~]# kubectl get pods pod-ports  -n dev -o wide
NAME        READY   STATUS    RESTARTS   AGE    IP           NODE    NOMINATED NODE   READINESS GATES
pod-ports   1/1     Running   0          115s   10.244.1.8   node1   <none>           <none>
[root@master ~]# curl 10.244.1.8:80
```

### 资源配额

资源配额很好理解，类似于Linux下的资源限额，对某个容器做资源限额，可以实现资源在各个pod之间的隔离，防止某个容器使用的资源过多，导致该Pod下的其他资源也无法使用，K8s提供了对内存和cpu资源进行配额的机制，这种机制通过resources选项实现，一般有两个子项：

- limits：用于限制运行时容器的最大占用资源，当容器占用资源超过limits时会被终止，并进行重启

- requests ：用于设置容器需要的最小资源，如果环境资源不够，容器将无法启动

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-resources
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    resources: # 资源配额
      limits:  # 限制资源（上限）
        cpu: "2" # CPU限制，单位是core数
        memory: "10Gi" # 内存限制
      requests: # 请求资源（下限）
        cpu: "1"  # CPU限制，单位是core数
        memory: "10Mi"  # 内存限制
```

```shell
[root@master ~]# kubectl create -f pod-resources.yaml 
pod/pod-resources created
```

gpwGPW2001