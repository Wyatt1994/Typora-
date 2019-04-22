**Zookeeper：提供了高效且可靠的分布式协调服务，例如统一命名服务、集中配置管理和分布式锁、数据发布/订阅、负载均衡、分布式队列等分布式基础服务。**

配置文件：zoo.cfg，java编写，通信端口2181，选举端口3888，leader,follower,observer,learner(除leader节点后所有其他服务器，用于感知集群所有情况)

![1542522688029](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542522688029.png)

https://blog.csdn.net/wzk646795873/article/details/79706627

#### Zookeeper如何保证分布式环境下的多请求安全性？

创建Znode节点的乐观锁机制（版本号）+客户端/服务端维护的发送请求和等待响应的请求的先进先出队列（每次从队列中按序获取）+每个请求的唯一xid（保证顺序性）+每个客户端建立的会话Sessionid(根据时间戳移位得到)。

#### SendThread

负责客户端和服务端网络请求I/O调度线程。一方面，在一定周期频率中发送PING包实现心跳检测，维持回话和重现，一方面负责请求发送和响应接收，并将响应时间转发给EventThread.

#### EventThread

负责响应事件的处理，触发watcher对象回调。

#### zookeeper深度解析

核心就**是ZAB,即原子广播协议，分为两种模式：leader选举和消息同步广播**，分别实现异常宕机选主和数据事务的同步一致性。



**1.leader选主**(当leader**崩溃**或者leader**失去大多数的follower**)：

**1）basic paxos**；即当前server向其他server询问，收到其他server的zxid，推荐zxid最大（**说明同步状态是最新的，通过epoch和后32位判断**）的作为leader再进行投票，重复直到选出**。**

**2）fast paxos**:即当前server自荐为leader，其他server收到提议变开始解决epoch和zxid的冲突（**zxid为64位数字，高32即为epoch，代表当前leader统治**），不断重复，若**自荐server解决了其他server的epoch和zxid的冲突**即为leader.



**2.消息同步广播：**类似于二阶段提交2PC，只是**移除了中断逻辑**。这意味着**一旦过半ACK则进行事务提交**，但会导致不一致问题。因此，可以通过**崩溃恢复模式**解决这个问题。

（1）Leader等待server连接；

（2）Follower连接leader，将最大的zxid发送给leader（即ACK)；

（3）Leader根据follower的zxid确定同步点，完成同步后通知follower 已经成为uptodate状态(doCommit)；

（4）Follower收到uptodate消息后，又可以重新接受client的请求进行服务了。

![1554730775575](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1554730775575.png)



#### Zookeeper 下 Server工作状态

每个Server在工作过程中有三种状态： 
**LOOKING**：当前Server不知道leader是谁，正在搜寻
**LEADING**：当前Server即为选举出来的leader

**FOLLOWING**：leader已经选举出来，当前Server与之同步



https://www.jianshu.com/p/8bf3b7ce3eaa

#### zookeeper如何判断leader失效

Leader节点与Follower节点使用**心跳检测**来感知对方的存在；**当Leader节点在超时时间内收到来自Follower的心跳检测那Follower节点会一直与该节点保持连接**；**若超时时间内Leader没有接收到来自过半Follower节点的心跳检测或TCP连接断开，那Leader会结束当前周期的领导，切换到Looking状态**，所有Follower节点也会放弃该Leader节点切换到Looking状态，然后开始新一轮选举。



#### 三种角色

Leader：提供读写服务（处理事务请求）

Follower：提供读服务

Observer：提供读服务（即非事务服务），即同步zookeeper集群的最新状态，区别在于不参加Leader选举投票和过半写成功的策略，**从而在不影响写性能的情况下提升集群的读性能。**

zk客户端与zk服务器建立的是**TCP长连接**，默认2181开始



#### 分布式读写锁

![1554733273950](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1554733273950.png)

#### 创建ZNode节点原子性原理

乐观锁版本号机制，每个ZNode节点都有相应的版本号信息。

#### Watcher实现原理

**注册watcher监听是一次性的**

**总体流程**

客户端向服务端注册一个watcher（取出**WatchRegistration**对象中的特定属性，例如请求，请求头进行序列化，避免网络过于紧张）并在客户端的**WatchManager**中注册一个watcher对象，一旦服务端触发watcher事件后会向客户端发起通知，客户端从watchManager中取出watcher对象执行**回调逻辑**。

**WatchedEvent(watch事件类)**封装类型包括：通知状态（是否连接，是否认证等），事件类型和节点路径等

**客户端：**

1.注册watcher

2.反序列化，将字节流转为watcherEvent独享

3.节点路径解析，补全完整路径

4.将**watcherEvent**转为**watchedEvent**对象

5.回调watcher：将watchedEvent独享交给EventThread。然后调用相应方法，即**串行的**从等待时间队列中取出真正的watche对象，完成后删除当前watcher对象。

**服务端：**

获取watcher注册请求，并存储到watchManger中的watcheTable中。一旦事件触发，则从watcheTable中取出对应的watcher,并删除，然后调用watcher对象process方法，用于封装watcherEvent进行网络序列化传输。

**权限控制**

ACL（访问控制表）：

1.权限模式:通过ip模式、用户名/密码等

2.授权对象id：ip地址、用户名密码的Base64j加密串

3.权限类别：创建删除等



#### zk客户端

![1555417760088](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1555417760088.png)



##### 保证zk单个客户端多次请求的顺序性：

通过请求头中的xid字段标记单个客户端的响应顺序。同时响应头中也会包含xid和zxid，即最新事务id。

**zookeeper根据不同的请求类型会定义不同的请求体**

##### 如何随机获取一个可用的zk服务端地址？

将解析出的server地址集合进行随机打散，然后拼装成一个**环形的循环队列**。

##### 对于zk server IP地址写死的情况，一旦zk服务端集群迁移怎么解决？

自定义实现一个配置管理中心HostProvider去解析zk服务器地址列表。同时，也可以通过HostProvider实现**就近服务**



#### 会话管理

将所有会话分配在不同的区块中，分配的原则是每个会话的**下次超时时间点**。即**“分桶策略”**，从而避免一次检查，提高效率。

#### 会话激活

**客户端通过PING请求**保证与服务端的会话有效性，**服务端接收**心跳检测并**重新激活**会话，称作TouchSession，用于保证客户端的长期存活性。**一旦重复激活，则会将当前会话迁移到下一个会话桶。**

#### 会话超时检查

通过一个检查线程，逐个一次对会话桶（包含多个session）中（会话桶，即代表不同过期时间点的多个session）的回话进行清理。

#### 重连

在session过期时间内重连则直接重连，**否则视为非法重连，需要重新实例化zk对象。**

### Zookeeper服务端

#### 初始化

![1555422057369](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1555422057369.png)

#### leader选举

**发生场景：**

1.服务器集群初始化

2.leader挂掉



至少两台才能实现选举，因为发起投票需要进行通信。

同时，接收到相同的投票的机器必须过半，即**大于等于n/2+1**

**原则：**zxid>myid(zk服务器的标识)，**若zxid相同，选择myid大的作为leader，即修改自己的自我投票，改投zxid大的。**此时收到相同的投票机器为2台，故完成选举。

#### 事务处理

针对每个事务请求，通过事务日志记录下来，同时通道follower中，follower需要发送ACK行相应。

1.发起投票：是否过半机器正常，zxid是否有效

2.生成proposal：生成一个新的zxid并封装一个proposal对象

3.广播proposal

4.收集投票：进行跟leader的事务日志的同步，然后返回ACK。一旦机器过半就会通过提议，进入commit阶段

5.广播commit:只需向follwer发送zxid即可。

