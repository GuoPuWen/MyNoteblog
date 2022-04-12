本篇文章重在实现，跳表这种数据结构第一次接触是在Redis里面，当时只是学习了跳表的理论知识，光靠理论是难以支撑的，这点在字节面试过程中被问到跳表便可体会到

跳表是一种非常高校的数据结构，是由美国科学家William Pugh发明的，他在一篇论文里面非常详细的介绍了跳表数据结构和插入删除等操作

# 基本思想

首先，跳表是对有序链表的改进，对于普通链表来说无论是普通链表还是有序链表，对于一个节点的查找操作都需要从头部开始逐个比较，有序链表有序的性质在这里不可用。有什么办法能用到有序链表的有序性质呢，使得有序链表的查找时间复杂度下降？

跳表就是这样一种数据结构，跳表在有序链表的基础上，引入了分层的概念，每一层都是一个有序的链表，除了最下面一层之外其他层都是索引，在这个索引的基础上进行查找将会非常快速，例如

![image-20210730093811204](http://cdn.noteblogs.cn/image-20210730093811204.png)

对上图上的结构，如果要查找数据5，首先从第一层开始，第一层只有一个元素，向下查找，第二次5<7接着向下查找，到了第三层4<5<7所以从节点4向下查找，对于第四层，查找到数据5

到这里，大家就明白了这其实就是二分的思想，让数据快速的接近要查找的数据，这就是跳表的基本思想，接下来就从具体实现来理解跳表是如何实现的，怎么建立层级的，插入删除操作是怎么做的

# 数据结构定义

首先，肯定需要一个节点的定义：

```java
/**
 * 跳表的实现Node节点
 * @param <T>
 */
class SkipNode<T>{
    int key;
    T value;
    SkipNode right, down;       //分别代表向右和向下的指针
    public SkipNode(int key, T value) {
        this.key = key;
        this.value = value;
    }

    /**
     * 初始化节点 用于初始化头节点
     * @param key
     */
    public SkipNode(int key){
        this.key = key;
        this.value = null;
    }
}
```

接着定义一个类

```java
public class SkipList<T> {
    SkipNode headNode;          //头节点
    int highLevel;              //层级高度
    Random random;              //随机的值 用于后续判断是否需要添加层级
    final int MAX_LEVEL = 12;   //最大的层级数

    /**
     * 构造函数 初始化跳表
     */
    public SkipList() {
        this.random = new Random();
        this.headNode = new SkipNode(Integer.MAX_VALUE);
        highLevel = 0;
    }
}
```

需要注意的是，为了让头节点与其他节点一样我们这里定义了一个虚拟的头节点，里面存在的元素是Integer.MAX_VALUE，这里还有一个random随机值，到后面插入的时候回使用到，highLevel表示当前跳表的层级高度，MAX_LEVEL表示最大的层级数

# 查找方法

对于查找方法的流程如下：

1）使用temp为头节点，将使用这个节点进行遍历

2）如果temp.key == key，那么查找成功，返回

3）如果temp.right == null，那么当前层级无这个key 必须向下 temp = temp.down

4）如果temp.right != null && temp.key < key，那么继续向右查找 temp = temp.right

5）如果temp.right != null && temp.key > key，那么当前层级无这个key，必须向下查找 temp = temp.down

```java
/**
     * 查找方法：四种情况
     * 1. 如果temp.key == key，那么查找成功，返回
     * 2. 如果temp.right == null，那么当前层级无这个key 必须向下 temp = temp.down
     * 3. 如果temp.right != null && temp.key < key，那么继续向右查找 temp = temp.right
     * 4. 如果temp.right != null && temp.key > key，那么当前层级无这个key，必须向下查找 temp = temp.down
     *
     * @param key
     * @return
     */
public SkipNode search(int key) {
  SkipNode temp = this.headNode;
  while (temp != null) {
    if (temp.key == key) {
      return temp;
    } else if (temp.right == null) {
      temp = temp.down;
    } else if (temp.key < key) {
      temp = temp.right;
    } else {
      temp = temp.down;
    }
  }
  return null;
}
```

# 删除方法

删除方法的流程为：

1）如果temp.right == null，那么temp = temp.down

2）如果temp.right != null && temp.right.key = key，那么删除temp.right节点，接着删除下一层级的节点 temp = temp.down

3）如果temp.right != null && temp.right.key < key，那么temp = temp.right

4）如果temp.right != null && temp.right.key > key，那么当前层级无这个key，temp = temp.down

```java
    /**
     * 删除：找到要删除节点的前一个节点，然后删除也有四种情况：
     * 1. 如果temp.right == null，那么temp = temp.down
     * 2. 如果temp.right != null && temp.right.key = key，那么删除temp.right节点，接着删除下一层级的节点 temp = temp.down
     * 3. 如果temp.right != null && temp.right.key < key，那么temp = temp.right
     * 4. 如果temp.right != null && temp.right.key > key，那么当前层级无这个key，temp = temp.down
     *
     * @param key
     */
    public void delete(int key) {
        SkipNode temp = this.headNode;
        while (temp != null) {
            if (temp.right == null) {
                temp = temp.down;
            } else if (temp.right.key == key) {
                temp.right = temp.right.right;      //删除节点
                temp = temp.down;                   //接着向下查找
            } else if (temp.right.key < key) {
                temp = temp.right;
            } else {
                temp = temp.down;
            }
        }
    }
```

# 插入方法

```java
    /**
     * 插入操作，需要考虑三个问题：
     * 1. 最底层肯定要将这个node插入，但是上面一层是否需要插入？采用随机值的方法判断是否要将这个节点往上一层插入
     * 2. 如果最高层还需要新建一个更高层，怎么操作？需要新建一个Node节点作为新的头节点，同时指向老的头节点
     * 3. 如何找到上层的待插入的节点？如果有双向指针这个简单，但是单向指针如何操作？借助栈，每次向下的时候就将当前节点存入栈里，下次要插入的时候再取出即可
     *
     * @param node
     */
    public void add(SkipNode<T> node) {
        int key = node.key;
        //判断是否存在该节点
        SkipNode findNode = this.search(node.key);
        if (findNode != null) {
            findNode.value = node.value;
            return;
        }
        Stack<SkipNode> stack = new Stack<>();
        SkipNode temp = this.headNode;
        while (temp != null) {
            if (temp.right == null || temp.right.key > key) {
                stack.add(temp);
                temp = temp.down;
            } else {
                temp = temp.right;
            }
        }
        int level = 1;      //代表当前层数
        SkipNode downNode = null;   //垂直方向的前驱节点
        while (!stack.isEmpty()) {
            temp = stack.pop();     //需要向后插入的点
            SkipNode nodeTemp = new SkipNode(node.key, node.value);
            nodeTemp.down = downNode;
            downNode = nodeTemp;    //处理水平方向
            //右侧为null 说明在末尾插入
            if (temp.right == null) {
                temp.right = nodeTemp;
            } else {
                nodeTemp = temp.right;
                temp.right = nodeTemp;
            }

            //考虑是否需要向上插入
            if (level > this.MAX_LEVEL) break;
            if (this.random.nextDouble() > 0.5) break;   //运气不好 不需要插入
            level++;
            if (level > this.highLevel) {
                highLevel = level;
                //创建一个新的头节点
                SkipNode headNew = new SkipNode(Integer.MAX_VALUE);
                headNew.down = this.headNode;
                this.headNode = headNew;
                stack.add(headNew);
            }
        }

    }
```

# 打印方法

```java
 public void print() {
        SkipNode teamNode = headNode;
        int index = 1;
        SkipNode last = teamNode;
        while (last.down != null) {
            last = last.down;
        }
        while (teamNode != null) {
            SkipNode enumNode = teamNode.right;
            SkipNode enumLast = last.right;
            System.out.printf("%-8s", "head->");
            while (enumLast != null && enumNode != null) {
                if (enumLast.key == enumNode.key) {
                    System.out.printf("%-5s", enumLast.key + "->");
                    enumLast = enumLast.right;
                    enumNode = enumNode.right;
                } else {
                    enumLast = enumLast.right;
                    System.out.printf("%-5s", "");
                }

            }
            teamNode = teamNode.down;
            index++;
            System.out.println();
        }

    }
```

# main

```java
    public static void main(String[] args) {
        SkipList<Integer> list = new SkipList<Integer>();
        for (int i = 1; i < 20; i++) {
            list.add(new SkipNode(i, 666));
        }
        System.out.println("删除前");
        list.print();
        list.delete(4);
        list.delete(8);
        System.out.println("删除后");
        list.print();
    }
```

# 完整代码

```java
package cn.noteblogs.skipList;

import com.sun.jmx.snmp.SnmpOid;

import java.awt.*;
import java.util.Random;
import java.util.Stack;

/**
 * 跳表的实现Node节点
 * @param <T>
 */
class SkipNode<T>{
    int key;
    T value;
    SkipNode right, down;       //分别代表向右和向下的指针
    public SkipNode(int key, T value) {
        this.key = key;
        this.value = value;
    }

    /**
     * 初始化节点 用于初始化头节点
     * @param key
     */
    public SkipNode(int key){
        this.key = key;
        this.value = null;
    }
}


/**
 * 跳表
 */
public class SkipList<T> {
    SkipNode headNode;          //头节点
    int highLevel;              //层级高度
    Random random;              //随机的值 用于后续判断是否需要添加层级
    final int MAX_LEVEL = 12;   //最大的层级数

    /**
     * 构造函数 初始化跳表
     */
    public SkipList() {
        this.random = new Random();
        this.headNode = new SkipNode(Integer.MAX_VALUE);
        highLevel = 0;
    }

    /**
     * 查找方法：四种情况
     * 1. 如果temp.key == key，那么查找成功，返回
     * 2. 如果temp.right == null，那么当前层级无这个key 必须向下 temp = temp.down
     * 3. 如果temp.right != null && temp.key < key，那么继续向右查找 temp = temp.right
     * 4. 如果temp.right != null && temp.key > key，那么当前层级无这个key，必须向下查找 temp = temp.down
     *
     * @param key
     * @return
     */
    public SkipNode search(int key) {
        SkipNode temp = this.headNode;
        while (temp != null) {
            if (temp.key == key) {
                return temp;
            } else if (temp.right == null) {
                temp = temp.down;
            } else if (temp.key < key) {
                temp = temp.right;
            } else {
                temp = temp.down;
            }
        }
        return null;
    }

    /**
     * 删除：找到要删除节点的前一个节点，然后删除也有四种情况：
     * 1. 如果temp.right == null，那么temp = temp.down
     * 2. 如果temp.right != null && temp.right.key = key，那么删除temp.right节点，接着删除下一层级的节点 temp = temp.down
     * 3. 如果temp.right != null && temp.right.key < key，那么temp = temp.right
     * 4. 如果temp.right != null && temp.right.key > key，那么当前层级无这个key，temp = temp.down
     *
     * @param key
     */
    public void delete(int key) {
        SkipNode temp = this.headNode;
        while (temp != null) {
            if (temp.right == null) {
                temp = temp.down;
            } else if (temp.right.key == key) {
                temp.right = temp.right.right;      //删除节点
                temp = temp.down;                   //接着向下查找
            } else if (temp.right.key < key) {
                temp = temp.right;
            } else {
                temp = temp.down;
            }
        }
    }

    /**
     * 插入操作，需要考虑三个问题：
     * 1. 最底层肯定要将这个node插入，但是上面一层是否需要插入？采用随机值的方法判断是否要将这个节点往上一层插入
     * 2. 如果最高层还需要新建一个更高层，怎么操作？需要新建一个Node节点作为新的头节点，同时指向老的头节点
     * 3. 如何找到上层的待插入的节点？如果有双向指针这个简单，但是单向指针如何操作？借助栈，每次向下的时候就将当前节点存入栈里，下次要插入的时候再取出即可
     *
     * @param node
     */
    public void add(SkipNode<T> node) {
        int key = node.key;
        //判断是否存在该节点
        SkipNode findNode = this.search(node.key);
        if (findNode != null) {
            findNode.value = node.value;
            return;
        }
        Stack<SkipNode> stack = new Stack<>();
        SkipNode temp = this.headNode;
        while (temp != null) {
            if (temp.right == null || temp.right.key > key) {
                stack.add(temp);
                temp = temp.down;
            } else {
                temp = temp.right;
            }
        }
        int level = 1;      //代表当前层数
        SkipNode downNode = null;   //垂直方向的前驱节点
        while (!stack.isEmpty()) {
            temp = stack.pop();     //需要向后插入的点
            SkipNode nodeTemp = new SkipNode(node.key, node.value);
            nodeTemp.down = downNode;
            downNode = nodeTemp;    //处理水平方向
            //右侧为null 说明在末尾插入
            if (temp.right == null) {
                temp.right = nodeTemp;
            } else {
                nodeTemp = temp.right;
                temp.right = nodeTemp;
            }

            //考虑是否需要向上插入
            if (level > this.MAX_LEVEL) break;
            if (this.random.nextDouble() > 0.5) break;   //运气不好 不需要插入
            level++;
            if (level > this.highLevel) {
                highLevel = level;
                //创建一个新的头节点
                SkipNode headNew = new SkipNode(Integer.MAX_VALUE);
                headNew.down = this.headNode;
                this.headNode = headNew;
                stack.add(headNew);
            }
        }

    }

    public void print() {
        SkipNode teamNode = headNode;
        int index = 1;
        SkipNode last = teamNode;
        while (last.down != null) {
            last = last.down;
        }
        while (teamNode != null) {
            SkipNode enumNode = teamNode.right;
            SkipNode enumLast = last.right;
            System.out.printf("%-8s", "head->");
            while (enumLast != null && enumNode != null) {
                if (enumLast.key == enumNode.key) {
                    System.out.printf("%-5s", enumLast.key + "->");
                    enumLast = enumLast.right;
                    enumNode = enumNode.right;
                } else {
                    enumLast = enumLast.right;
                    System.out.printf("%-5s", "");
                }

            }
            teamNode = teamNode.down;
            index++;
            System.out.println();
        }

    }

    public static void main(String[] args) {
        SkipList<Integer> list = new SkipList<Integer>();
        for (int i = 1; i < 20; i++) {
            list.add(new SkipNode(i, 666));
        }
        System.out.println("删除前");
        list.print();
        list.delete(4);
        list.delete(8);
        System.out.println("删除后");
        list.print();
    }
}

```

![image-20210730100416876](http://cdn.noteblogs.cn/image-20210730100416876.png)