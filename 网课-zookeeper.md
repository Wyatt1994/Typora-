主要用java写的，直接解压，cp重命名一个zoo.cfg文件启动即可。zkServer.sh start 

**核心：文件系统+监听通知机制（观察者模式）**

![1542282975246](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542282975246.png)



![1542283151133](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542283151133.png)

文件系统的每一个节点都可以存储数据

![1542284739291](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542284739291.png)

**get /zkPro:**查看节点信息和数据，体现每一个节点的管理就像一个**文件系统**

![1542285128238](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542285128238.png)

也可以继续创建子节点 **create /zkPro/child  wyatt888**，然后get  /zkPro/child 即可获得对应的数据

即create  目录路径（含创建的目录名）  节点名（该目录下的节点名称），  get  目录位置

![1542285716623](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542285716623.png)

每一个客户端对指定的节点进行监听，一旦有变化则反向通知给客户端

## zookeeper集群

也就是要有3个不同的zoo-x.cfg配置文件，需要修改dataDir（快照数据存放目录)，clientPort，以及整个集群的所有主机的ip和两个端口（一个端口用于和集群中leader的通信端口，一个是用于选举leader端口）

![1542289904497](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542289904497.png)

![1542289968400](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542289968400.png)

![1542290721375](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542290721375.png)

使用 zkServer.sh status zoo-x.cfg:查看集群节点的信息，是否为leader或follower

然后java客户端连接时，直接在加上对应的ip，用逗号隔开即可

![1542291573090](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542291573090.png)



#### **leader能提供读写服务，follower只能执行读服务。**

注：对于集群连接时，使用从节点进行修改数据，即写请求依然成功，是因为将写请求**转发给了leader**节点。然后leader节点会根据写请求生成一个**唯一、全局、顺序的zxid(z：zookeeper，x:表示事务，严格保证顺序，先进先出，锁机制)**，然后将写事务进行**广播**，所有follower收到后需要进行**确认**（Ack），只要收到**半数以上**的follower的Ack，然后leader会进行commint提交事务，同时广播给所有follower要求进行**commit提交**，即也进行写，**更新自己本地的数据**，相当于进行了**同步**，保证数据一致性，即每台zookeeper上的数据都一致。

#### 原子消息广播（ZAB）

包含**集群leader选举**和**集群消息广播**



##### 如何进行集群消息广播？

对于集群连接时，使用从节点进行修改数据，即写请求依然成功，是因为将写请求**转发给了leader**节点。然后leader节点会根据写请求生成一个**唯一、全局、顺序的zxid(z：zookeeper，x:表示事务，严格保证顺序，先进先出，锁机制)**，然后将写事务进行**广播**，所有follower收到后需要进行**确认**（Ack），只要收到**半数以上**的follower的Ack，然后leader会进行commint提交事务，同时广播给所有follower要求进行**commit提交**，即也进行写，**更新自己本地的数据**，相当于进行了**同步**，保证数据一致性，即每台zookeeper上的数据都一致。



##### 如何进行leader选举？

存活的机器会发起投票，消息格式为（myid,zxid），myid就是每一个机器的唯一id。zxid有可能由于突然宕机导致zxid全局不一致，但也会发起投票，每台存活的机器根据自己收到的最近的zxid+1作为最新的zxid。当汇总后，只会选择**最大的zxid选票，其他选票作废，则作废节点会更新自己的zxid，myid更改为通知其作废的那个节点的myid**。所有节点进行统计自己受到的选票，当选票**大于**，（不含等于）总机器数（包含宕机的）的一半



##### 为什么规定要求大于 可用节点数量 > 集群总结点数量/2 ？  

如果不这样限制，在集群出现脑裂的时候，可能会出现多个子集群同时服务的情况（即子集群各组选举出自己的leader）， 这样对整个zookeeper集群来说是紊乱的。

换句话说，如果遵守上述规则进行选举，即使出现脑裂，集群**最多也只能会出现一个子集群**可以提供服务的情况（**能满足节点数量> 总结点数量/2 的子集群最多只会有一个**）。所以要限制 可用节点数量 > 集群总结点数量/2 。



### 面试题

![1542290103198](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542290103198.png)

##### 为什么集群数一般是奇数？

1、防止由脑裂造成的集群不可用。例如4台，可能会脑裂成2,2情况，导致都不能选举出leader。

2、在容错能力相同的情况下，奇数台更节省资源。例如3台，

##### Zookeeper的ZAB协议是什么？

包含**集群leader选举**和**集群消息广播**

##### Zookeeper的脑裂是什么？

集群的**脑裂：**通常是发生在节点之间**通信不可达**的情况下，集群会分裂成不同的**小集群**，小集群**各自选出自己的leader节点**，导致原有的集群出现**多个master节点**的情况，这就是脑裂。



