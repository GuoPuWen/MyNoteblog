[Kubernetes--Kubernetes简介以及Kubernetes安装](https://blog.csdn.net/weixin_44706647/article/details/118692499)

前面介绍了Kubernetes的安装方式，本篇文章主要介绍Kubernetes的资源管理方式，首先需要理解什么是资源？资源管理的方式一共有三种：命令式对象管理、命令式对象配置、声明式对象配置，三种方式的使用场景等等

# 什么是资源管理？

在K8s中，所有的内容都被抽象为资源，用户需要操作资源来操作K8s，资源可以分为计算资源、存储资源、网络资源，对于容器化技术来说对这些资源的定义、管理、分配尤其重要，在K8s中，任何可以被申请、分配、最终被使用的对象，都可以是K8s中的资源，例如CPU、内存等等

K8s的本质是一个集群系统，用户可以在集群中部署各种服务，就是在K8s的集群中运行一个一个的容器，并将指定的程序跑在容器中，例如部署一个Nginx容器。

但是K8s的最小单位是pod而不是容器，所以只能将容器放入到pod中，而且K8s也不直接管理pod，而是通过pod控制器来管理pod的，pod可以部署容器之后，那么怎么提供服务呢？需要通过service资源实现服务提供，如果pod需要存储数据持久化，K8s海提供了各种存储系统用于持久化

![image-20210713105726871](http://cdn.noteblogs.cn/image-20210713105726871.png)

# 资源管理方式

K8s对资源的管理一共有三种方式：

- 命令式对象管理：直接使用命令去操作K8s的资源

```
kubectl run nginx-pod --image=nginx:1.17.1 --port=80
```

- 命令式对象配置：通过命令配置和配置文件去操作资源

```shell
kubectl create/patch -f nginx-pod.yaml
```

- 声明式对象配置：通过apply命令和配置文件去操作资源

```shell
kubectl apply -f nginx-pod.yaml
```

| 类型           | 操作对象 | 适用环境 | 优点           | 缺点                             |
| -------------- | -------- | -------- | -------------- | -------------------------------- |
| 命令式对象管理 | 对象     | 测试     | 简单           | 只能操作活动对象，无法审计、跟踪 |
| 命令式对象配置 | 文件     | 开发     | 可以审计、跟踪 | 项目大时，配置文件多，操作麻烦   |
| 声明式对象配置 | 目录     | 开发     | 支持目录操作   | 意外情况下难以调试               |

# Kubectl命令详解

在介绍三种的资源管理方式之前，首先需要知道Kubectl命令的使用。Kubectl式使用K8s集群的命令行工具，通过它对集群本身进行管理，Kubectl的基本使用方式是：

```shell
Kubectl [command] [type] [name] [flags]
```

- command：指定要对资源执行的操作，例如create、get、delete等等
- type：指定资源的类型，例如pod、service等等
- name：指定资源的名称，注意大小写
- flages：指定额外的可选参数

例如：

```shell
[root@master ~]# kubectl get pod				#查看pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6867cdf567-lt9wz   1/1     Running   0          15h
[root@master ~]# kubectl get pod nginx-6867cdf567-lt9wz			#查看指定的pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6867cdf567-lt9wz   1/1     Running   0          15h
[root@master ~]# kubectl get pod nginx-6867cdf567-lt9wz -o yaml		# 查看pod的详细信息用yaml文件的方式
```

## 资源类型

在K8s中有很多的资源类型，可以使用`kubectl api-resources`进行查看，经常使用的资源有

<table>
	<tr>
	    <th>资源分类</th>
	    <th>资源名称</th>
		<th>缩写</th>
		<th>资源作用</th>
	</tr>
	<tr>
	    <td rowspan="2">集群级别资源</td>
        <td>nodes</td>
	    <td>no</td>
		<td>集群组成部分</td>
	</tr>
	<tr>
		<td>namespaces</td>
	    <td>ns</td>
		<td>隔离Pod</td>
	</tr>
	<tr>
		<td>pod资源</td>
	    <td>pods</td>
	    <td>po</td>
		<td>装载容器</td>
	</tr>
	<tr>
		<td rowspan="8">pod资源控制器</td>
	    <td>replicationcontrollers</td>
	    <td>rc</td>
		<td>控制pod资源</td>
	</tr>
	<tr>
	    <td>replicasets</td>
	    <td>rs</td>
		<td>控制pod资源</td>
	</tr>
	<tr>
	    <td>deployments</td>
	    <td>deploy</td>
		<td>控制pod资源</td>
	</tr>
	<tr>
	    <td>daemonsets</td>
	    <td>ds</td>
		<td>控制pod资源</td>
	</tr>
	<tr>
	    <td>jobs</td>
	    <td></td>
		<td>控制pod资源</td>
	</tr>	
	<tr>
	    <td>cronjobs</td>
	    <td>cj</td>
		<td>控制pod资源</td>
	</tr>	
	<tr>
	    <td>horizontalpodautoscalers</td>
	    <td>hpa</td>
		<td>控制pod资源</td>
	</tr>	
	<tr>
	    <td>statefulsets</td>
	    <td>sts</td>
		<td>控制pod资源</td>
	</tr>
	<tr>
		<td rowspan="2">服务发现资源</td>
	    <td>services</td>
	    <td>svc</td>
		<td>统一pod对外接口</td>
	</tr>
    <tr>
	    <td>ingress</td>
	    <td>ing</td>
		<td>统一pod对外接口</td>
	</tr>
	<tr>
		<td rowspan="3">存储资源</td>
	    <td>volumeattachments</td>
	    <td></td>
		<td>存储</td>
	</tr>
	<tr>
	    <td>persistentvolumes</td>
	    <td>pv</td>
		<td>存储</td>
	</tr>
	<tr>
	    <td>persistentvolumeclaims</td>
	    <td>pvc</td>
		<td>存储</td>
	</tr>
	<tr>
		<td rowspan="2">配置资源</td>
	    <td>configmaps</td>
	    <td>cm</td>
		<td>配置</td>
	</tr>
	<tr>
	    <td>secrets</td>
	    <td></td>
		<td>配置</td>
	</tr>
</table>

## 操作

使用`kubernetes --help`可以查看帮助文档

<table>
	<tr>
	    <th>命令分类</th>
	    <th>命令</th>
		<th>翻译</th>
		<th>命令作用</th>
	</tr>
	<tr>
	    <td rowspan="6">基本命令</td>
	    <td>create</td>
	    <td>创建</td>
		<td>创建一个资源</td>
	</tr>
	<tr>
		<td>edit</td>
	    <td>编辑</td>
		<td>编辑一个资源</td>
	</tr>
	<tr>
		<td>get</td>
	    <td>获取</td>
	    <td>获取一个资源</td>
	</tr>
   <tr>
		<td>patch</td>
	    <td>更新</td>
	    <td>更新一个资源</td>
	</tr>
	<tr>
	    <td>delete</td>
	    <td>删除</td>
		<td>删除一个资源</td>
	</tr>
	<tr>
	    <td>explain</td>
	    <td>解释</td>
		<td>展示资源文档</td>
	</tr>
	<tr>
	    <td rowspan="10">运行和调试</td>
	    <td>run</td>
	    <td>运行</td>
		<td>在集群中运行一个指定的镜像</td>
	</tr>
	<tr>
	    <td>expose</td>
	    <td>暴露</td>
		<td>暴露资源为Service</td>
	</tr>
	<tr>
	    <td>describe</td>
	    <td>描述</td>
		<td>显示资源内部信息</td>
	</tr>
	<tr>
	    <td>logs</td>
	    <td>日志</td>
		<td>输出容器在 pod 中的日志</td>
	</tr>	
	<tr>
	    <td>attach</td>
	    <td>缠绕</td>
		<td>进入运行中的容器</td>
	</tr>	
	<tr>
	    <td>exec</td>
	    <td>执行</td>
		<td>执行容器中的一个命令</td>
	</tr>	
	<tr>
	    <td>cp</td>
	    <td>复制</td>
		<td>在Pod内外复制文件</td>
	</tr>
		<tr>
		<td>rollout</td>
	    <td>首次展示</td>
		<td>管理资源的发布</td>
	</tr>
	<tr>
		<td>scale</td>
	    <td>规模</td>
		<td>扩(缩)容Pod的数量</td>
	</tr>
	<tr>
		<td>autoscale</td>
	    <td>自动调整</td>
		<td>自动调整Pod的数量</td>
	</tr>
	<tr>
		<td rowspan="2">高级命令</td>
	    <td>apply</td>
	    <td>rc</td>
		<td>通过文件对资源进行配置</td>
	</tr>
	<tr>
	    <td>label</td>
	    <td>标签</td>
		<td>更新资源上的标签</td>
	</tr>
	<tr>
		<td rowspan="2">其他命令</td>
	    <td>cluster-info</td>
	    <td>集群信息</td>
		<td>显示集群信息</td>
	</tr>
	<tr>
	    <td>version</td>
	    <td>版本</td>
		<td>显示当前Server和Client的版本</td>
	</tr>
</table>

## 简单操作

```shell
[root@master ~]# kubectl create ns dev				# 
namespace/dev created
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   25h
dev               Active   8s
kube-node-lease   Active   25h
kube-public       Active   25h
kube-system       Active   25h
[root@master ~]# kubectl run pod --image=nginx -n dev
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/pod created
[root@master ~]# kubectl get pod -n dev
NAME                   READY   STATUS              RESTARTS   AGE
pod-864f9875b9-mf9px   0/1     ContainerCreating   0          19s
[root@master ~]# kubectl describe  pod pod-864f9875b9-mf9px  -n dev
Name:         pod-864f9875b9-mf9px
Namespace:    dev
Priority:     0
Node:         node2/10.0.0.6
Start Time:   Tue, 13 Jul 2021 15:17:29 +0800
Labels:       pod-template-hash=864f9875b9
              run=pod
Annotations:  <none>
Status:       Running
IP:           10.244.2.3
IPs:
  IP:           10.244.2.3
Controlled By:  ReplicaSet/pod-864f9875b9
Containers:
  pod:
    Container ID:   docker://4307997e9e433225abb06cf5d069535d76a62372dedd9e3d9874e671adabb142
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:353c20f74d9b6aee359f30e8e4f69c3d7eaea2f610681c4a95849a2fd7c497f9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 13 Jul 2021 15:18:02 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ngk5d (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-ngk5d:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-ngk5d
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  60s   default-scheduler  Successfully assigned dev/pod-864f9875b9-mf9px to node2
  Normal  Pulling    60s   kubelet, node2     Pulling image "nginx"
  Normal  Pulled     27s   kubelet, node2     Successfully pulled image "nginx"
  Normal  Created    27s   kubelet, node2     Created container pod
  Normal  Started    27s   kubelet, node2     Started container pod
```

# 命令式对象管理

在使用kubectl命令的使用的时候，其实使用的就是命令式对象管理，就是只是使用命令对资源进行管理，这种方式一般适用于测试环境

# 命令式对象配置

命令式对象配置就是使用命令和配置文件的方式一起操作资源，例如同样是创建一个名为dev的namespace名称空间下的nginx资源

> 1. 创建一个nginxpod.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
  namespace: dev
spec:
  containers:
  - name: nginx-containers
    image: nginx:1.17.1
```

> 2. 执行create命令，创建资源

```shell
[root@master ~]# kubectl create -f nginxpod.yaml 
namespace/dev created
pod/nginxpod created
```

> 3. 执行get命令，查看创建的pod和ns

```shell
[root@master ~]# kubectl get -f nginxpod.yaml 
NAME            STATUS   AGE
namespace/dev   Active   54s

NAME           READY   STATUS    RESTARTS   AGE
pod/nginxpod   1/1     Running   0          53s
```

> 4. 执行删除命令

```shell
[root@master ~]# kubectl delete -f nginxpod.yaml 
namespace "dev" deleted
pod "nginxpod" deleted
```

# 声明式对象配置

声明式对象配置同样是写一个配置文件，但是只使用apply操作，也就是说无法执行删除操作使用apply操作资源会进行以下两种操作：

- 如果资源不存在，则直接创建相当于`kubectl create`
- 如果资源已经存在，那么执行更新相当于`kubectl patch`

```shell
[root@master ~]# kubectl apply -f nginxpod.yaml 
namespace/dev created
pod/nginxpod created
[root@master ~]# kubectl apply -f nginxpod.yaml 
namespace/dev unchanged
pod/nginxpod unchanged
[root@master ~]# kubectl get  -f nginxpod.yaml 
NAME            STATUS   AGE
namespace/dev   Active   18s

NAME           READY   STATUS    RESTARTS   AGE
pod/nginxpod   1/1     Running   0          17s
```



对于三种操作资源的方式，可以结合起来使用，因为声明式对象配置是不能进行删除操作的，可以如下结合：

- 创建/更新资源      使用声明式对象配置 kubectl apply -f  XXX.yaml

-  删除资源              使用命令式对象配置 kubectl delete -f  XXX.yaml

- 查询资源              使用命令式对象管理 kubectl get(describe) 资源名称

