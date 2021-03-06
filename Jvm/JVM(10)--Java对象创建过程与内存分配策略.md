# 一、Java对象创建过程

### 第一步：类加载检查

当虚拟机遇到一条new指令的时候，会去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个类是否被加载、连接、初始化过，如果这些都完成直接返回该对象即可，如果没有则进行下一步操作

### 第二步：分配内存

在类加载检查通过，接下来虚拟机将会新生对象分配对象，对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同与把一块确定大小的内存同堆中划分出来，分配方式有两种，指针碰撞和空闲列表，使用这两种方法的规则判定是当前的内存空间是否完整，而堆空间是否完整取决于使用哪种的回收算法

标记-清除算法会产生内存碎片，造成了内存空间不完整，使用标记-整理或者复制算法都不会产生内存碎片，所以空间都是规整的

- 指针碰撞：适合于堆内存规整的情况，堆内存规整的情况下，一边是使用过的内存，另外一边是空间的内存，中间有一个指针作为边界指示器，分配内存只需要将指针往空闲方向移动
- 空闲列表：适用于内存不规整的情况，也就是说内存用过的空间和空闲的空间是交错的，这就需要维护一个列表，来记录一下那些内存可用，在给对象分配内存的时候，查找列表找到足够的的区域，然后更新这个列表

==在分配内存时如何保证线程安全？==

在分配内存的时候保证线程有两种办法：

- CAS+失败重试：对分配内存的动作进行原子操作，使用CAS机制保证更新操作的原子性
- TLAB：前面说堆是线程共享的区域，但是在堆中有一部分很特殊的空间，TLAB是堆上线程独占的区域，在分配内存时可以先在TLAB上进行分配，当对象的空间大于TLAB，或者TLAB的空间不足的时候在进行CAS+失败重试的方法在堆上共享的空间进行分配

### 第三步：初始化零值

内存分配完成之后，需要将分配到的内存空间都初始化为0值，但是不包括对象头，这一步操作在类加载机制的时候说到过，初始化零值保证了静态变量，类实例变量不显示指定值也可以使用

### 第四步：设置对象头

初始化零值完成以后，就需要对对象进行必要的设置，这就是对对象头进行设置，对象头一般存储两方面的数据

- 自身的运行信息，包括哈希码，GC分代标记，锁状态也被成为Mark Work
- 类型指针，对象指向它的类的元数据的指针
- 数组长度，这是可选的字段，只有当对象是数组是才有

### 第五步：执行init方法

上面的工作都完成之后，在JVM看来对象已经创建完成了，但是对于程序员来说这还没有结束，因为init方法还没有执行，所有的字段都还为0，所以这里需要执行init方法

# 二、对象的内存分配策略

对象的内存分配策略是建立在垃圾的是进行分代收集的，根据分代算法，堆区分为新生代和老年代，新生代具有Eden区、form区和To区，那么一个对象创建分配到哪里呢？有什么样的顺序？

首先是，对象优先在Eden区进行分配，当Eden区没有足够的空间进行分配时，虚拟机将会发起一次Minor GC，将Eden区域进行垃圾回收，如果进行了Minor GC后，Eden区还是放不下该对象，则该对象将会直接放入老年代也就是所谓的大对象，如果老年代还是放不下该对象，则直接报出OOM

在进行Minor GC的时候会清理Eden区的对象，将Eden区内存活的对象存到from区，同时也会判断from区是否有垃圾需要进行回收，有垃圾则进行回收，同时将剩下的对象再次转移到to区并且将这些对象的年龄计数加1，经过一阵时间后，有些对象的年龄到达15(这个阈值可以通过参数进行设定)，那么将这些对象移动到老年代。

# 三、对象访问定位的方式

上面说明了Java创建对象的过程以及如何进行内存分配，与此相对应的是如何进行访问的呢？引用是存在Java虚拟机栈的栈帧上的，Java虚拟机规范没有定义这个引用该使用何种方式进行定位、访问这些在堆上的对象，一般来说有两种方式，通过句柄和直接指针。

### 3.1 句柄

使用句柄的方式的话，Java堆中将会划分出一块内存来作为句柄池，reference中存储的是对象的句柄地址，那句柄中包含了对象实例数据和类型数据的具体地址值

![这里写图片描述](http://cdn.noteblogs.cn/20180119233225143)

### 3.2 直接指针

直接指针是指在局部变量表里存的直接是Java堆中对象的地址

![这里写图片描述](http://cdn.noteblogs.cn/20180119233413460)

这两种访问方式的不同，使用句柄的最大好处是reference里存的是稳定的句柄地址，在对象被移动的时候只会改变句柄中的实例数据指针，而引用reference并不需要修改。

使用直接指针的方式最大的好处就是访问速度快，不需要经过一次指针定位的时间