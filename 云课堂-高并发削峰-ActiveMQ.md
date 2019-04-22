同步/异步是针对于客户端

阻塞非阻塞是针对于服务端

![1542024877592](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542024877592.png)

nio：利用多路复用器实现

![1542025245300](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542025245300.png)

 ![1542028786492](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542028786492.png)

tomcat默认采用http-apr-8080，利用操作系统优化的io方式，可以修改配置文件进行改动。

**apr**(Apache Portable Runtime/Apache可移植运行时)，是Apache HTTP服务器的支持库。你可以简单地理解为，Tomcat将以JNI的形式调用Apache HTTP服务器的核心动态链接库来处理文件读取或网络传输操作，从而大大地提高Tomcat对静态文件的处理性能。 Tomcat apr也是在Tomcat上运行高并发应用的首选模式。

8009是apache的通信端口

![1542025291561](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542025291561.png)



![1542025869412](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542025869412.png)



![1542026064373](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542026064373.png)



![1542026463523](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542026463523.png)

点对点模式：无时间耦合（任何时候查看即可），一对一，消费者需要应答已消费数据

发布/订阅

![1542026714235](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542026714235.png)



![1542026728824](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542026728824.png)

![1542027327166](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542027327166.png)

![1542027359177](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542027359177.png)



![1542027444264](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542027444264.png)

![1542027466277](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542027466277.png)

![1542027709544](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542027709544.png)

ActiceMQ默认采用 KahaDB模式，可修改配置文件进行更改



Mycat用于进行高并发下分库分表

集群环境下

![1542028118242](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542028118242.png)

![1542028131969](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542028131969.png)

第二种：意思是共享同一个收到的消息文件库，即kahadb目录共享，并设置一个排它锁，拿到的就成为master，其他节点进行循环争锁

第三种：共用一个数据库，里面有一个activemq-lock表，拿到该表即锁定为master

第三种：zookeeper进行选举master，但知道要有3台节点

![1542028440345](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542028440345.png)

其中，Kafka不支持JMS规范



应用程序卡死分析原 jstack dump查看fullgc日志，发现大量线程获取数据库连接被阻塞。然后向DBA生成AWR报告

top1:sql，磁盘io占用太高，busy,然后分析sql，调用explain或者profile，

不适用索引的原因：

1.最左前缀不符合

