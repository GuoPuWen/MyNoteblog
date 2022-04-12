# 简介

在前面的一些实验中，已经体会过当一个Pod由于某种原因挂掉之后又重新启动，例如启动busybox由于没有后台进程Pod，Pod会一直重启。也就是说K8s可以检测到Pod是否存活，能检测到Pod是否可以对外进行提供服务，这就涉及到Pod的生命周期问题

Pod的生命周期从整体上看，一共有以下几个过程：

1. Pod的创建过程
2. 运行初始化容器（init container）的过程
3. 运行主容器（main container）：
   - 容器启动后钩子（post start），容器终止前钩子（pre stop）
   - 容器的存活性探测（liveness probe），就绪性探测（readiness probe）
4. Pod的终止过程

![image-20210715100005679](http://cdn.noteblogs.cn/image-20210715100005679.png)

一个Pod一共有五个状态，Pod 的 `status` 字段是一个 PodStatus对象，其中包含一个 `phase` 字段。

Pod 的阶段（Phase）是 Pod 在其生命周期中所处位置的简单宏观概述，一共有以下五种取值：

- Pending（挂起）：apiserver已经已经创建好了Pod对象，但是它没有被调度或者还在下载网络镜像
- Running（运行中）：Pod已经被调度至某个节点，并且所有的容器都被kubelet创建完成
- Succeeded（成功）：Pod中的所有容器都已经成功终止并且不会被重启
- Failed（失败）：Pod中的所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非0值的退出状态
- Unknown（未知）：无法获取某个Pod的状态，一般是apiserver与主机通信失败

#  创建和终止

**Pod的创建过程**：

1. 用户通过kubectl或其他api客户端提交需要创建的pod信息给apiServer

2. apiServer开始生成pod对象的信息，并将信息存入etcd，然后返回确认信息至客户端

3. apiServer开始反映etcd中的pod对象的变化，其它组件使用watch机制来跟踪检查apiServer上的变动

4. scheduler发现有新的pod对象要创建，开始为Pod分配主机并将结果信息更新至apiServer

5. node节点上的kubelet发现有pod调度过来，尝试调用docker启动容器，并将结果回送至apiServer

6. apiServer将接收到的pod状态信息存入etcd中

<img src="http://cdn.noteblogs.cn/image-20200406184656917.png" alt="image-20200406184656917" style="zoom:100%;" />

**pod的终止过程**

1. 用户向apiServer发送删除pod对象的命令
2. apiServcer中的pod对象信息会随着时间的推移而更新，在宽限期内（默认30s），pod被视为dead
3. 将pod标记为terminating状态
4. kubelet在监控到pod对象转为terminating状态的同时启动pod关闭过程
5. 端点控制器监控到pod对象的关闭行为时将其从所有匹配到此端点的service资源的端点列表中移除
6. 如果当前pod对象定义了preStop钩子处理器，则在其标记为terminating后即会以同步的方式启动执行
7. pod对象中的容器进程收到停止信号
8. 宽限期结束后，若pod中还存在仍在运行的进程，那么pod对象会收到立即终止的信号
9. kubelet请求apiServer将此pod资源的宽限期设置为0从而完成删除操作，此时pod对于用户已不可见

# 初始化容器

初始化容器是在pod的主容器启动之前要运行的容器，主要是做一些主容器的前置工作，它具有两大特征：

1. 初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么kubernetes需要重启它直到成功完成
2. 初始化容器必须按照定义的顺序执行，当且仅当前一个成功之后，后面的一个才能运行

初始化容器有很多的应用场景，下面列出的是最常见的几个：

- 提供主容器镜像中不具备的工具程序或自定义代码
- 初始化容器要先于应用容器串行启动并运行完成，因此可用于延后应用容器的启动直至其依赖的条件得到满足

例如，一个Nginx容器的启动可能要检查MySQL、Redis是否启动成功，而初始化容器的存在便可以做到这种前置工作

下面做一个案例来演示初始化容器的作用

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod1
  labels:
    app: init
spec:
  containers:
  - name: init-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

上面创建一个主容器，但是在启动主容器之前需要启动两个启动容器，分别检测myservice和mydb的启动情况，在启动容器的时候，处于noready的状态，因为没有启动对应的两个服务

```shell
[root@master ~]# kubectl get pods -o wide
NAME                     READY   STATUS     RESTARTS   AGE     IP            NODE    NOMINATED NODE   READINESS GATES
init-pod1                0/1     Init:0/2   0          2m41s   10.244.2.16   node2   <none>           <none>
nginx-6867cdf567-lt9wz   1/1     Running    0          4d17h   10.244.1.3    node1   <none>           <none>
```

Service.yaml

```yaml
kind: Service
apiVersion: v1
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 6376
---
kind: Service
apiVersion: v1
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 6377
```

当启动两个服务之后，等待几分钟可以发现主容器处于启动状态，这就可以体会启动容器的作用了

```shell
[root@master ~]# kubectl get pods init-pod1 -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
init-pod1   1/1     Running   0          10m   10.244.2.16   node2   <none>           <none>
```

# 钩子函数

钩子函数可以感知自身生命周期中的事件，并在相应的时刻运行用户指定的程序代码，K8s提供了两个钩子函数：

- post start：在容器创建之后执行，如果失败会重启容器
- pre stop：在容器终止之前执行，执行完成之后容器将成功到达终止状态

钩子函数处理器支持使用三种方式定义动作：

- Exec命令：在容器内执行一次命令

```shell
……
  lifecycle:
    postStart: 
      exec:
        command:
        - cat
        - /tmp/healthy
……
```

- TCPSocker：在当前容器内尝试访问指定的socker

```shell
……      
  lifecycle:
    postStart:
      tcpSocket:
        port: 8080
……
```

- HTTPGet：在当前容器中向某url发起http请求

```shell
……
  lifecycle:
    postStart:
      httpGet:
        path: / #URI地址
        port: 80 #端口号
        host: 192.168.109.100 #主机地址
        scheme: HTTP #支持的协议，http或者https
……
```

来体验一下钩子函数的作用，创建pod-hook-exec.yaml文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hook-exec
  namespace: dev
spec:
  containers:
  - name: main-container
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
    lifecycle:
      postStart: 
        exec: # 在容器启动的时候执行一个命令，修改掉nginx的默认首页内容
          command: ["/bin/sh", "-c", "echo postStart... > /usr/share/nginx/html/index.html"]
      preStop:
        exec: # 在容器停止之前停止nginx服务
          command: ["/usr/sbin/nginx","-s","quit"]
```

上面容器的启动过程中，使用了一个postStart钩子函数，用于在容器启动的时候执行一个命令，修改掉Nginx的默认首页内容

```shell
[root@master ~]# kubectl create -f pod-hook-exec.yaml 
pod/pod-hook-exec created
[root@master ~]# kubectl get pods -o wide -n dev
NAME            READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
pod-hook-exec   1/1     Running   0          85s   10.244.2.17   node2   <none>           
[root@master ~]# curl 10.244.2.17
postStart...
```

# 容器探测

在Pod的基本配置章节，配置了一个busybox，但是由于里面没有配置一个后台进程，所以Pod将这个容器不断的重启，而Pod是怎么判定一个容器是否存活呢？Pod是怎么知道一个进程是否已经准备好对外提供服务了呢？

Pod的liveness probes存活性探针和readiness probes就绪性探针可以解释这个问题：

- liveness probes：存活性探针，用于检测应用实例当前是否处于正常运行状态，如果不是，k8s会重启容器
- readiness probes：就绪性探针，用于检测应用实例当前是否可以接收请求，如果不能，k8s不会转发流量

上面两种探针目前均支持三种探测方式：

- Exec命令：在容器内执行一次命令，如果命令执行的退出码为0，则认为程序正常，否则不正常

  ~~~yaml
  ……
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
  ……
  ~~~

- TCPSocket：将会尝试访问一个用户容器的端口，如果能够建立这条连接，则认为程序正常，否则不正常

  ~~~yaml
  ……      
    livenessProbe:
      tcpSocket:
        port: 8080
  ……
  ~~~

- HTTPGet：调用容器内Web应用的URL，如果返回的状态码在200和399之间，则认为程序正常，否则不正常

  ~~~yaml
  ……
    livenessProbe:
      httpGet:
        path: / #URI地址
        port: 80 #端口号
        host: 127.0.0.1 #主机地址
        scheme: HTTP #支持的协议，http或者https
  ……
  ~~~

下面以liveness probes为例，做几个演示：

**exec**

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    image: [root@master ~]# kubectl create -f exec-liveness.yaml 
pod/liveness-exec createdbusybox
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

这个配置文件给Pod配置了一个容器，periodSeconds规定Kubectl要每隔5秒执行一次livenessProbe，`initialDelaySeconds` 告诉kubelet在第一次执行probe之前要等待5秒钟，探测检测命令是在容器中执行cat /tmp/healthy 命令，如果命令执行成功则返回0，kubectl则会认为该容器是活着的并且很健康，如果返回非0值，kubectl就会将这个容器杀掉并且重启。同时在容器启动的时候执行命令

```shell
/bin/sh -c "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600"
```

在容器生命的最初30秒会创建一个文件，那么在这30秒内cat  /tmp/healthy会返回一个成功的状态码，30秒后cat  /tmp/healthy将返回一个失败的返回码

在30秒内，可以看到容器是成功创建的

```shell
[root@master ~]# kubectl create -f exec-liveness.yaml 
pod/liveness-exec created
[root@master ~]# kubectl describe pod liveness-exec
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  45s   default-scheduler  Successfully assigned default/liveness-exec to node1
  Normal  Pulling    45s   kubelet, node1     Pulling image "busybox"
  Normal  Pulled     27s   kubelet, node1     Successfully pulled image "busybox"
  Normal  Created    27s   kubelet, node1     Created container liveness
  Normal  Started    26s   kubelet, node1     Started container liveness
```

等经过30秒后，容器内部执行删除/tmp/healthy操作，所以容器的探测检测失败，容器被不断重启

```shell
[root@master ~]# kubectl get pods liveness-exec
NAME            READY   STATUS    RESTARTS   AGE
liveness-exec   1/1     Running   0          96s
[root@master ~]# 
[root@master ~]# kubectl get pods liveness-exec
NAME            READY   STATUS    RESTARTS   AGE
liveness-exec   1/1     Running   3          5m3s
[root@master ~]# kubectl get pods liveness-exec
NAME            READY   STATUS    RESTARTS   AGE
liveness-exec   1/1     Running   3          5m5s
```

# 重启策略

一但容器探测出现问题，K8s将需要对容器所在的Pod进行重启，Pod一共有三种重启：

- Always：容器失效时，自动重启该容器，默认
- OnFailure：容器终止运行且退出码不为0时重启
- Never：不论状态为何，都不重启该容器

重启策略适用于pod对象中的所有容器，首次需要重启的容器，将在其需要时立即进行重启，随后再次需要重启的操作将由kubelet延迟一段时间后进行，且反复的重启操作的延迟时长以此为10s、20s、40s、80s、160s和300s，300s是最大延迟时长

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-restartpolicy
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      httpGet:
        scheme: HTTP
        port: 80
        path: /hello
  restartPolicy: Never # 设置重启策略为Never
```

可以发现，Pod的重启次数始终为0

```shell
[root@master ~]# kubectl  get pods pod-restartpolicy -n dev
NAME                READY   STATUS      RESTARTS   AGE
pod-restartpolicy   0/1     Completed   0          35s
```

