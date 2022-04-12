一般情况下，一个Pod在那个Pod节点上运行，是由Scheduler组件采用相应的计算算法计算出来的，本篇文章首先介绍Pod默认的调度机制，然后介绍Pod提供给我们的自定义调度规则，Kubenetes提供了四大类调度方式：

- 自动调度：运行在哪个节点上完全由Scheduler经过一系列的算法计算得出
- 定向调度：NodeName、NodeSelector
- 亲和性调度：NodeAffinity、PodAffinity、PodAntiAffinity
- 污点（容忍）调度：Taints、Toleration

# 自动调度：Kube-scheduler

调度器通过K8s的检测（Watch）机制来发现集群中新创建且尚未被调度到Node节点上的Pod，调度器会将发现的每一个未调度的Pod调度到一个合适的Node上来运行

Kube-scheduler是K8s集群的默认调度器，并且是集群控制的一部分。对于默认的Pod调度，Kube-scheduler会选择一个最优的Node去运行这个Pod，在这个Pod被调度在Node上之前，会根据特定的资源调度需求，需要对集群中的Node节点进行一次过滤

当过滤完之后得到可调度Pod的Node列表，然后根据一系列函数对这些可调度节点进行打分，选出其中得分最高的Node来运行Pod，之后调度器将这个决定通知给Kube-apiserver，这个过程叫做绑定

**kube-scheduler调度流程：**

Kube-scheduler给一个Pod做选择调度包含两个步骤：

- 过滤
- 打分

在过滤阶段，会将所有满足Pod调度需求的Node选出来，例如，PodFitsResources 过滤函数会检查候选Node的可用资源能否满足Pod的资源请求，在过滤之后得出一个Node列表，里面包含了所有可调度的节点，在过滤之后几乎如果这个列表是空的，代表这个不可调度

在打分阶段，调度器会为Pod从所有可调度节点中选取一个最合适的Node，根据当前启用的打分规则，调度器会给每一个调度节点进行打分，最后kube-scheduler回将Pod调度到最高得分的Node上，如果存在多个得分最高的Node，kube-scheduler 会从中随机选取一个进行调度

# Pod调度



