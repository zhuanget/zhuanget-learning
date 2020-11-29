# Redis的集群模式

## 一、概念

一个Redis集群又多个节点组成，各个节点之间通过握手建立连接，构成集群。Redis集群通过将数据分片，交由不同节点负责，同时也提供复制和故障转移功能。集群模式在横向上对Redis进行了扩展，通过将数据分片，避免大量的数据存储在一台服务器上的情况，同时，当数据量不断增大时，也有利于系统的扩展。进一步提高系统的高可用性和可扩展性。

## 二、集群数据结构

通过定义节点数据结构clusterNode进行保存，该结构会保存一个节点的当前状态，比如节点的创建时间、节点的名字、节点当前的配置纪元、节点的IP地址和端口号等等。每个节点都会使用一个clusterNode结构来记录自己的状态，并为集群中的所有其他的节点都创建一个相应的clusterNode结构，以此来记录其他节点的状态。

大概结构如下：

```java
class ClusterNode {
    Time createTime;
    String nodeName;
    int flags;
    //用于实现故障转移
    int configEpoch;
    String ip;
    int port;
    List<ClusterLink> links;
}
```

ClusterLink结构保存了连接节点所需的有关信息，比如套接字描述符，输入缓冲区和输出缓冲区：

大概结构如下：

```java
Class ClusterLink {
 	Time createTime;
    //TCP套接字描述符
    int fd;
    //输出缓冲区，保存着等待发送给其他节点的信息
    Buffer sndbuf;
    //输入缓冲区，保存着从其他节点接收到的消息
    Buffer rcvbuf;
    //与这个连接相关联的节点，如果没有的话就为null
    ClusterNode node;
}
```

## 三、槽指派

集群通过分片的方式来保存数据库的键值对，集群会将数据库分为16384个槽，每个分片会规划几个槽，0~16384，然后将这些分片交由集群内的节点管理，通过命令```CLUSTER ADDSLOTS```进行分派。当所有的槽都有相应的节点处理时，视为该集群在线，如果有一个以上的槽没有被任何一个节点处理，视为不在线。

ClusterNode结构的slots属性和numslot属性记录了节点负责处理哪些槽，换句话说，```ADDSLOTS```实际上就是对这两个变量进行赋值，及set方法。

##### ```CLUSTER ADDSLOTS```命令的实现

要对槽进行分派，首先需要检查输入槽是否已经分派过了，如果已分派过，就会报错并返回；如果都为分派，才会进行后续分派动作，伪代码如下：

```java
for (int i = 0; i < inputSlot.length; i++) {
    if (isAssigned(inputSlot[i])) {
        throw new RuntimeException("input slots have one or more assigned!");
    }
}

for (int i = 0; i < inputSlot.length; i++) {
    clusterState.slots[i] = clusterState.myself;
    setSlotBit(clusterState.myself.slots, i);
}
```

clusterState.slots是一个数组，保存了所有槽的指派信息，程序要检查槽i是否已经被指派，又或者取得负责处理槽i的节点，只需要访问clusterState.slots[i]的值即可，时间复杂度为O(1)。

## 四、在集群中执行命令

当客户端向集群发送与数据库键相关的命令后，接收命令的节点会先计算该键所对应的槽位于哪一个节点上，

1）如果位于该接收命令的节点上，则将在该节点上直接执行命令；

2）如果不在该节点上，则会通过查询槽所在的节点，向客户端返回一个MOVED错误，指引客户端指向槽所在的节点，并再次发送之前想要执行的命令。

## 五、重新分片

Redis集群的重新分片操作可以将任意数量已经指派给某个节点（源节点）的槽改为指派给另一个节点（目标节点），并且相关槽所属的键值对也会从源节点被移动到目标节点。所以，也就是，先进行复制完成数据转移，然后再进行槽的重新指派。