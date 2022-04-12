# 七、Zookeeper事件监听机制

### 7.1 watcher概念

zookeeper提供了数据的发布/订阅功能，多个订阅者可同时监听某一特定主题对象，当该主题对象的自身状态发生变化时(例如节点内容改变、节点下的子节点列表改变等)，会实时、主动通知所有订阅者

zookeeper采用了Watcher机制实现数据的发布/订阅功能。该机制在被订阅对象发生变化时会异步通知客户端，因此客户端不必在Watcher注册后轮询阻塞，从而减轻了客户端压力。

### 7.2 watcher架构

![image-20201218193211986](Zookeeper(2).assets/image-20201218193211986.png)

1. 客户端首先将Watcher注册到服务端，同时将Watcher对象保存到客户端的Watch管理器中
2. 当ZooKeeper服务端监听的数据状态发生变化时，服务端会主动通知客户端，接着客户端的Watch管理器会触发相关Watcher来回调相应处理逻辑，从而完成整体的数据发布/订阅流程。  

Watcher实现由三个部分组成：

- Zookeeper服务端
- Zookeeper客户端
- 客户端的ZKWatchManager对象  

### 7.3 watcher特性

|       特性       |                             说明                             |
| :--------------: | :----------------------------------------------------------: |
|      一次性      | watcher是一次性的，一旦被触发就会移除，再次使用时需要重新注册 |
| 客户端顺 序回 调 | watcher回调是顺序串行化执行的，只有回调后客户端才能看到最新的数据状态。一个watcher回调逻辑不应该太多，以免影响别的watcher执行 |
|      轻量级      | WatchEvent是最小的通信单元，结构上只包含通知状态、事件类型和节点路径，并不会告诉数据节点变化前后的具体内容； |
|      时效性      | watcher只有在当前session彻底失效时才会无效，若在session有效期内快速重连成功，则watcher依然存在，仍可接收到通知； |

### 7.4 watcher接口

- Watcher通知状态(KeeperState)  

|   枚举属性    |           说明           |
| :-----------: | :----------------------: |
| SyncConnected | 客户端与服务器正常连接时 |
| Disconnected  | 客户端与服务器断开连接时 |
|    Expired    |    会话session失效时     |
|  AuthFailed   |      身份认证失败时      |

- Watcher事件类型(EventType)  

EventType是数据节点(znode)发生变化时对应的通知类型。EventType变化时KeeperState永远处于SyncConnected通知状态下；当KeeperState发生变化时，EventType永远为None。其路径为org.apache.zookeeper.Watcher.Event.EventType，是一个枚举类，枚举属性如下：  

|      枚举属性       |                            说明                            |
| :-----------------: | :--------------------------------------------------------: |
|        None         |                             无                             |
|     NodeCreated     |               Watcher监听的数据节点被创建时                |
|     NodeDeleted     |               Watcher监听的数据节点被删除时                |
|   NodeDataChanged   | Watcher监听的数据节点内容发生变更时(无论内容数据 是否变化) |
| NodeChildrenChanged |        Watcher监听的数据节点的子节点列表发生变更时         |

等会儿通过案例来使用这些接口

# 八、一些案例

### 8.1 配置中心案例

工作中有这样的一个场景： 数据库用户名和密码信息放在一个配置文件中，应用读取该配置文件，配置文件信息放入缓存。若数据库的用户名和密码改变时候，还需要重新加载缓存，比较麻烦，通过ZooKeeper可以轻松完成，当数据库发生变化时自动完成缓存同步。  

```java
public class MyConfigCenter {
    private static String IP = "192.168.18.102:2181";
    private CountDownLatch count = new CountDownLatch(1);
    private ZooKeeper zooKeeper;
    private static MyConfig config = new MyConfig();

    public MyConfigCenter(){
        this.getValue();
    }

    class MyWatch implements Watcher {

        public void process(WatchedEvent watchedEvent) {
            if(watchedEvent.getType() == Event.EventType.None){
                if(watchedEvent.getState() == Event.KeeperState.SyncConnected){
                    System.out.println("连接成功");
                    count.countDown();
                }else if (watchedEvent.getState() == Event.KeeperState.Disconnected){
                    System.out.println("连接断开！");
                } else if (watchedEvent.getState() == Event.KeeperState.Expired){
                    System.out.println("连接超时！自动重新连接。。。");
                    try {
                        zooKeeper = new ZooKeeper(IP,5000,this);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                } else if (watchedEvent.getState() == Event.KeeperState.AuthFailed){
                    System.out.println("验证失败！");
                }
                //监听节点发生变化
            }else if (watchedEvent.getType() == Event.EventType.NodeDataChanged) {
                getValue();
            }
        }
    }
    public void getValue(){
        try {
            //判断zookeeper是否为空值，防止重复创建
            if(zooKeeper == null) {
                zooKeeper = new ZooKeeper(IP, 5000, new MyWatch());
				count.await();
            }
            config.setUsername(new String(zooKeeper.getData("/config/username", new MyWatch(), null)));
            config.setPassword(new String(zooKeeper.getData("/config/password", new MyWatch(), null)));
            config.setUrl(new String(zooKeeper.getData("/config/url", new MyWatch(), null)));
           
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        MyConfigCenter center = new MyConfigCenter();
        while (true){
            //休眠3秒
            try { TimeUnit.SECONDS.sleep(3);} catch (InterruptedException e) {e.printStackTrace();}
            System.out.println(config);
        }
    }
}
public class MyConfig {
    private String url;
    private String username;
    private String password;
}
```

测试：

![image-20201218203939752](Zookeeper(2).assets/image-20201218203939752.png)

当节点的配置信息发生变化时

![image-20201218204108195](Zookeeper(2).assets/image-20201218204108195.png)



### 8.2 生成分布式唯一ID

在过去的单库单表型系统中，通常可以使用数据库字段自带的auto_increment属性来自动为每条记录生成一个唯一的ID。但是分库分表后，就无法在依靠数据库的auto_increment属性来唯一标识一条记录了。此时我们就可以用zookeeper在分布式环境下生成全局唯一ID。  

```java
public class GloballyUniqueId {
    private String IP = "192.168.18.102:2181";
    private CountDownLatch count = new CountDownLatch(1);
    private ZooKeeper zooKeeper;
    private String defaultPath = "/uniqueId";
    public GloballyUniqueId(){
        if (zooKeeper == null) {
            try {
                zooKeeper  = new ZooKeeper(IP, 5000, new MyWatch());
                count.await();
            } catch (Exception e) {
                e.printStackTrace();
            }

        }
    }
    class MyWatch implements Watcher {

        public void process(WatchedEvent watchedEvent) {
            if (watchedEvent.getType() == Event.EventType.None) {
                if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                    System.out.println("连接成功");
                    count.countDown();
                } else if (watchedEvent.getState() == Event.KeeperState.Disconnected) {
                    System.out.println("连接断开！");
                } else if (watchedEvent.getState() == Event.KeeperState.Expired) {
                    System.out.println("连接超时！自动重新连接。。。");
                    try {
                        zooKeeper = new ZooKeeper(IP, 5000, this);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                } else if (watchedEvent.getState() == Event.KeeperState.AuthFailed) {
                    System.out.println("验证失败！");
                }
            }
        }


    }

    public String getId() {
        String path = "";
        try {

            //创建临时节点
            path = zooKeeper.create(defaultPath, new byte[0],
                    ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);

        } catch (Exception e) {
            e.printStackTrace();
        }
        //System.out.println(path);
       //   /uniqueId0000000015
        return path.substring(9);
    }

    public static void main(String[] args) {
        GloballyUniqueId Id = new GloballyUniqueId();
        while (true){
            try { TimeUnit.SECONDS.sleep(3);} catch (InterruptedException e) {e.printStackTrace();}
            System.out.println(Id.getId());

        }

    }
}
```

![image-20201218210112077](Zookeeper(2).assets/image-20201218210112077.png)

### 8.3 分布式锁

设计思路：

1. 每个客户端往/Locks下创建临时有序节点/Locks/Lock000000001
2. 客户端取得/Locks下子节点，并进行排序，判断排在最前面的是否为自己，如果自己的锁节点在第一位，代表获取锁成功
3. 如果自己的锁节点不在第一位，则监听自己前一位的锁节点。例如，自己锁节点Lock 000000001
4. 当前一位锁节点（Lock000000002）的逻辑
5. 监听客户端重新执行第2步逻辑，判断自己是否获得了锁  

多线程中的锁可以有sync锁，这里我采用的是自旋的方式

```java
public class MyLock {

    private String IP = "192.168.18.102:2181";
    private CountDownLatch count = new CountDownLatch(1);
    private ZooKeeper zooKeeper;
    private String lockPath = "/Locks4";
    private String lockName = "Lock_";
    private String path = "";
    private boolean flag = false;
    class MyWatch implements Watcher {
        public void process(WatchedEvent watchedEvent) {
            if (watchedEvent.getType() == Event.EventType.None) {
                if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                    System.out.println("连接成功");
                    count.countDown();
                }else if (watchedEvent.getState() == Event.KeeperState.Disconnected) {
                    System.out.println("连接断开！");
                } else if (watchedEvent.getState() == Event.KeeperState.Expired) {
                    System.out.println("连接超时！自动重新连接。。。");
                    try {
                        zooKeeper = new ZooKeeper(IP, 5000, this);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                } else if (watchedEvent.getState() == Event.KeeperState.AuthFailed) {
                    System.out.println("验证失败！");
                }
            }
        }
    }
    public MyLock(){
        if (zooKeeper == null) {
            try {
                zooKeeper  = new ZooKeeper(IP, 5000,new MyWatch());
                count.await();
            } catch (Exception e) {
                e.printStackTrace();
            }

        }
    }
    public void lock(){
        createNode();
        attempLock();
    }

    public void createNode(){
        try {
            Stat stat = zooKeeper.exists(lockPath, null);
            if(stat == null){
                zooKeeper.create(lockPath, new byte[0],
                        ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            }
            //创建临时有序子节点
            path = zooKeeper.create(lockPath + "/" + lockName, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.EPHEMERAL_SEQUENTIAL);

            System.out.println("节点创建成功" + path);
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    Watcher watcher = new Watcher() {
        public void process(WatchedEvent watchedEvent) {
            //监听到删除节点
            if (watchedEvent.getType() == Event.EventType.NodeDeleted) {
//                synchronized (this) {
//                    notifyAll();
//                }
                flag = true;
            }
        }
    };

    public void attempLock(){

        try {
            List<String> list = zooKeeper.getChildren(lockPath, false);
            Collections.sort(list);
            // /Locks/Lock_000000001
            //判断自己创建的这个节点是否是子节点中的第一个节点
            // 获取 Lock_000000001
            int i = list.indexOf(path.substring(lockPath.length() + 1));
            //说明是第一个进来的
            if(i == 0){
                System.out.println("获取锁成功！");
                return;
            }else{
                String prePath = list.get(i - 1);
                Stat stat = zooKeeper.exists(lockPath + "/" + prePath, watcher);
                if(stat == null){
                    attempLock();
                }else{
//                    synchronized (watcher){
//                        watcher.wait();
//                    }
                    while(flag){}
                    attempLock();
                }

            }
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void unlock(){
        try {
            zooKeeper.delete(path, -1);
            flag = false;
            System.out.println("锁已经释放" + path);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }
}
```

# 九、一致性协议zab

ZAB 协议是 Zookeeper 专门设计的一种支持崩溃恢复的原子广播协议。通过该协议，Zookeepe 基于
主从模式的系统架构来保持集群中各个副本之间数据的一致性。具体如下：

Zookeeper 使用一个单一的主进程来接收并处理客户端的所有事务请求，并采用原子广播协议将数据状
态的变更以事务 Proposal 的形式广播到所有的副本进程上去。如下图  

![image-20201219190749978](Zookeeper(2).assets/image-20201219190749978.png)

ZAB 协议包括两种基本的模式，分别是崩溃恢复和消息广播：

1. 崩溃恢复
   当整个服务框架在启动过程中，或者当 Leader 服务器出现异常时，ZAB 协议就会进入恢复模式，通过过半选举机制产生新的 Leader，之后其他机器将从新的 Leader 上同步状态，当有过半机器完成状态同步后，就退出恢复模式，进入消息广播模式。
2.  消息广播
   ZAB 协议的消息广播过程使用的是原子广播协议。在整个消息的广播过程中，Leader 服务器会每个事物请求生成对应的 Proposal，并为其分配一个全局唯一的递增的事务 ID(ZXID)，之后再对其进行广播。具体过程如下：

Leader 服务会为每一个 Follower 服务器分配一个单独的队列，然后将事务 Proposal 依次放入队列中，并根据 FIFO(先进先出) 的策略进行消息发送。Follower 服务在接收到 Proposal 后，会将其以事务日志的形式写入本地磁盘中，并在写入成功后反馈给 Leader 一个 Ack 响应。当 Leader 接收到超过半数 Follower 的 Ack 响应后，就会广播一个 Commit 消息给所有的 Follower 以通知其进行事务提交，之后 Leader 自身也会完成对事务的提交。而每一个 Follower 则在接收到 Commit 消息后，完成事务的提交。  

# 十、zookeeper的leader选举  

### 4.1 服务器状态

- looking：寻找leader状态。当服务器处于该状态时，它会认为当前集群中没有

- leader，因此需要进入leader选举状态。

- leading： 领导者状态。表明当前服务器角色是leader。

- following： 跟随者状态。表明当前服务器角色是follower。

- observing：观察者状态。表明当前服务器角色是observer。

  成为leader。  

### 4.2 服务器启动时期的leader选举  

集群初始化阶段，当有一台服务器server1启动时，其单独无法进行和完成leader选举，当第二台服务器server2启动时，此时两台机器可以相互通信，每台机器都试图找到leader，于是进入leader选举过程。选举过程如下：

1. 每个server发出一个投票。由于是初始情况，server1和server2都会将自己作为leader服务器来进行投票，每次投票会包含所推举的服务器的myid和zxid，使用(myid, zxid)来表示，此时server1的投票为(1, 0)，server2的投票为(2, 0)，然后各自将这个投票发给集群中其他机器。
2. 集群中的每台服务器接收来自集群中各个服务器的投票。
3. 处理投票。针对每一个投票，服务器都需要将别人的投票和自己的投票进行pk，pk
   规则如下：
   1. 优先检查zxid。zxid比较大的服务器优先作为leader。
   2. 如果zxid相同，那么就比较myid。myid较大的服务器作为leader服务器。
   3. 对于Server1而言，它的投票是(1, 0)，接收Server2的投票为(2, 0)，首先会比较两者的zxid，均为0，再比较myid，此时server2的myid最大，于是更新自己的投票为(2, 0)，然后重新投票，对于server2而言，其无须更新自己的投票，只是再次向集群中所有机器发出上一次投票信息即可。

4. 统计投票。每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到相同的投票信息，对于server1、server2而言，都统计出集群中已经有两台机器接受了(2, 0)的投票信息，此时便认为已经选出了leader
5. 改变服务器状态。一旦确定了leader，每个服务器就会更新自己的状态，如果是follower，那么就变更为following，如果是leader，就变更为leading。  

### 4.3 服务器运行时期的Leader选举  

在zookeeper运行期间，leader与非leader服务器各司其职，即便当有非leader服务器宕机或新加入，此时也不会影响leader，但是一旦leader服务器挂了，那么整个集群将暂停对外服务，进入新一轮leader选举，其过程和启动时期的Leader选举过程基本一致。


