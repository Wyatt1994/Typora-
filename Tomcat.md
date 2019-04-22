#### Tomcat容器结构

tomcat，核心就是Catalina，也就是servlet容器。

Catalina主要分为两个模块：

1.**连接器connetctor**：负责将一个请求与容器关联起来，包括将每个HTTP请求创建一个request对象和response对象。然后交给容器container。

2.**容器container**：接收到request和response对象，并负责调用相应的service()方法。



#### 连接器Connector



#### 容器Container



#### Servlet容器步骤（只会实例化一次，因此会有线程安全问题）

Servlet线程安全问题详解：单实例的servlet和多线程调用service()方法

<https://blog.csdn.net/xiaojiahao_kevin/article/details/51781946>

```
Servlet容器默认是采用单实例多线程的方式处理多个请求的：
1.当web服务器启动的时候（或客户端发送请求到服务器时），Servlet就被加载并实例化(只存在一个Servlet实例)；
2.容器初始化化Servlet主要就是读取配置文件（例如tomcat,可以通过servlet.xml的<Connector>设置线程池中线程数目，初始化线程池通过web.xml,初始化每个参数值等等。
3.当请求到达时，Servlet容器通过调度线程(Dispatchaer Thread) 调度它管理下线程池中等待执行的线程（Worker Thread）给请求者；
4.线程执行Servlet的service方法；
5.请求结束，放回线程池，等待被调用；
（注意：避免使用实例变量（成员变量），因为如果存在成员变量，可能发生多线程同时访问该资源时，都来操作它，照成数据的不一致，因此产生线程安全问题）局部变量是安全的的，因为是在栈中分配，每个线程有独立的栈。
```

![1555509022411](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1555509022411.png)

![1555830634338](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1555830634338.png)



HTTPServer的await()方法中，会根据**request中的URI进行解析判断是否为静态资源或者servlet**，交给相应的处理器。其中，servlet类的加载使用**URLClassLoader.loadClass(servletName)**方法进行加载，然后newInstance()并强转为Servlet。内部依然调用的是ClassLoader父类的loadClass()方法



若要保证servlet的线程安全，**

1.实现SingleThreadModel接口

2.使用synchronize对共享数据进行同步，即锁servlet自身

#### Tomcat集群同步原理

<https://blog.csdn.net/hty46565/article/details/73302769?utm_source=itdadao&utm_medium=referral>

**1.首先各个节点有三个组件：**

Manager：将操作的信息记录下来

Cluster：序列化信息

Tribes：将信息发送出去

信息发送出去的格式是ClusterMessage，其他节点的接受操作刚好相反。

![img](https://upload-images.jianshu.io/upload_images/14270041-548dbe15128af76b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/687/format/webp)

**2.同步方式有两种：**

**集群增量会话管理器**：

a.这是一种全节点复制模式，即其中有一个节点发生变化后，将同步到其他节点。

b.其次，该复制模式还有一个特点就是只同步会话增量的特点，增量是以一个会话为周期，即在一个请求被响应之前同步到各个节点上。

**集群备份会话管理器：**

全节点复制模式有一个缺点：当集群节点数增大时，会导致网络通信量急剧增加。备份会话管理器则是**每个会话只有一个备份。**

3.ClusterMessage实现了Serializable接口，所以它可以序列化Cluser发送给其他节点。

#### 

#### 与Jetty的区别

jetty 默认NIO 灵活轻巧，插件化，在docker上部署容易

| 名称   | 上手度                             | 性能                       | 更新频率 | 扩展性 |
| ------ | ---------------------------------- | -------------------------- | -------- | ------ |
| Tomcat | 容易                               | 从Tomcat6支持nio，性能优秀 | 普通     | 不好   |
| Jetty  | 比较慢。灵活性同时带来一定的复杂度 | 默认是NIO，性能优秀        | 快       | 好     |

1. 在web容器上，我们要与时俱进，不能只追求现在，在高并发下，我们要有相关的经验及应对措施。
2. NIO要优于BIO，而jetty同时也是推荐用NIO
3. jetty的灵活小巧，加载速度快，方便调试等都是促使我们去选择 tomcat公司很多不屑于用一样，其实tomcat还是很不错的。用tomcat支持并发2000的，也是经常干的事。之所以选择jetty，那原因就不多说了，jboss不给力而且又大，tomcat公司不支持，所以jetty就这样成为项目中的不二选择，当然他也是非常优秀的产品。 综上所述，在性能上tomcat与jetty差距并不大，可以说没有。只是**jetty相对来说由于其灵活性，插件化，导致jetty某些场合(如虚拟机、嵌入式)更节约资源**，当然对于我们现在的应用可以忽略这个因素。



#### Tomcat错误信息处理

Tomcat所有错误消息存储在一个properties文件中。根据模块不同，放在了不同的包模块下。通过一个单例的StringManager进行错误信息查找。

**tomcat错误日志位置**

tomcat/logs。其中有一个.out文件和.log文件。catalina.out文件记录了运行时的各种信息

**tomcat异常情况分类**

​    1.并发用户数目过大，也会导致tomcat自动停止服务。 调整tomcat的jvm内存配置参数
​    2.系统本身的网络负载平衡没有做好，导致tomcat自动停止服务； 
​    3.程序迭代不合理也是一个原因； 查看运行日志，锁定代码位置
​    4.数据库连接未关闭，导致资源损耗过重，会引起服务停止； 

​    5.程序严重错误，也会引起tomcat停止服务！

**异常排查步骤**

1.首先，如果有nginx，查看nginx日志，使用curl命令查看是否有代理转发问题。

2.查看gc日志或者使用tomcat内置gc脚本，例如堆转储日志。

3.top查看内存负载

4.若是应用程序本身问题，则使用jstack查看线程运行日志。

5.大量tcp连接的close_wait,则使用neestat配合awk命令进行筛选

netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

6.查过最大连接数，默认为200个，则需要修改配置文件，增加最大连接数。