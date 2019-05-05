

java面试总结

https://mp.weixin.qq.com/s?__biz=MzI5MzYzMDAwNw==&mid=2247484212&idx=1&sn=b9bcfa3e758ded928b7d07ae451f13d6&chksm=ec6e7a6cdb19f37a9c09383095193e1070e8673c15bcc983cf5fbe6d4286900393af13a8911a&mpshare=1&scene=1&srcid=1119ksuyRAaIdVhmz79Yq3VR#rd

#### 1.你用过哪些集合类？

大公司最喜欢问的Java集合类面试题40个Java集合面试问题和答案java.util.Collections 是一个包装类。它包含有各种有关集合操作的静态多态方法。java.util.Collection 是一个集合接口。它提供了对集合对象进行基本操作的通用接口方法。

**注意：**Collection（List,Queue,Set）和Map接口是并列的

Collection

├List

│├LinkedList

│├ArrayList

│└Vector

│　└Stack

└Set

Map

├Hashtable

├HashMap

└WeakHashMap

还有Queue,实现类为LinkedList和PriorityQueue

ArrayList、HashMap、TreeMap和HashTable类提供对元素的随机访问。

线程安全

VectorHashTable(不允许插空值)

非线程安全

ArrayList LinkedList HashMap(允许插入空值)HashSet TreeSet TreeMap(基于红黑树的Map实现)

#### 2.你说说 arraylist 和 linkedlist 的区别？

ArrayList和LinkedList两者都实现了List接口，但是它们之间有些不同。（1）ArrayList是由Array所支持的基于一个索引的数据结构，所以它提供对元素的随机访问（2）与ArrayList相比，在LinkedList中插入、添加和删除一个元素会更快（3）LinkedList比ArrayList消耗更多的内存，因为LinkedList中的每个节点存储了前后节点的引用

#### 3.HashMap 底层是怎么实现的？还有什么处理哈希冲突的方法？

处理哈希冲突的方法:

解决HashMap一般没有什么特别好的方式，要不扩容重新hash要不优化冲突的链表结构

1.开放定地址法-线性探测法

2.开放定地址法-平方探查法

3.链表解决-可以用红黑树提高查找效率

HashMap简介HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。HashMap 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。HashMap 的实现不是同步的，这意味着它不是线程安全的,但可以用 Collections的synchronizedMap方法使HashMap具有线程安全的能力。它的key、value都可以为null。此外，HashMap中的映射不是有序的。HashMap 的实例有两个参数影响其性能：“初始容量” 和 “加载因子”。初始容量默认是16。默认加载因子是 0.75, 这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本.HashMap是数组+链表+红黑树（JDK1.8增加了红黑树部分）实现的,当链表长度太长（默认超过8）时，链表就转换为红黑树.

Java8系列之重新认识HashMap功能实现-方法确定哈希桶数组索引位置 :这里的Hash算法本质上就是三步：**取key的hashCode值、高位运算、取模运算。**







1234567891011

方法一：static final int hash(Object key) { //jdk1.8 & jdk1.7int h;// h = key.hashCode() 为第一步 取hashCode值// h ^ (h >>> 16) 为第二步 高位参与运算return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);}方法二：static int indexFor(int h, int length) { //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的return h & (length-1); //第三步 取模运算}

分析HashMap的put方法扩容机制：原来的两倍

#### 4.熟悉什么算法，还有说说他们的时间复杂度？

经典排序算法总结与实现

#### 5.ArrayList和Vector的底层代码和他们的增长策略,它们是如何进行扩容的？

ArrayList 和Vector默认数组大小是10，其中ensureCapacity扩容，trimToSize容量调整到适中，扩展后数组大小为（(原数组长度1.5）与传递参数中较大者.Vector的扩容，是可以指定扩容因子，即指定每次扩容长度，同时Vector扩容策略是：1.**原来容量的2倍,**2.原来容量+扩容参数值。

#### 6.jvm 原理。程序运行区域划分

问：Java运行时数据区域？回答：包括程序计数器、JVM栈、本地方法栈、方法区、堆

问：方法区里存放什么？用于存储已被虚拟机加载的类信息，常量、静态变量、即时编译器编译后的代码等。

本地方法栈：和jvm栈所发挥的作用类似，区别是jvm栈为jvm执行java方法（字节码）服务，而本地方法栈为jvm使用的native方法服务。

JVM栈：局部变量表、操作数栈、动态链接、方法出口。

堆：存放对象实例。

#### 7.minor GC 与 Full GC，分别什么时候会触发？ 。分别采用哪种垃圾回收算法？简单介绍算法

GC（或Minor GC）：收集 生命周期短的区域(Young area)。Full GC （或Major GC）：收集生命周期短的区域(Young area)和生命周期比较长的区域(Old area)对整个堆进行垃圾收集。新生代通常存活时间较短基于Copying算法进行回收,将可用内存分为大小相等的两块，每次只使用其中一块；当这一块用完了，就将还活着的对象复制到另一块上，然后把已使用过的内存清理掉。在HotSpot里，考虑到大部分对象存活时间很短将内存分为Eden和两块Survivor，默认比例为8:1:1。代价是存在部分内存空间浪费，适合在新生代使用；老年代与新生代不同，老年代对象存活的时间比较长、比较稳定，因此采用标记(Mark)算法来进行回收,所谓标记就是扫描出存活的对象，然后再进行回收未被标记的对象，回收后对用空出的空间要么进行合并、要么标记出来便于下次进行分配，总之目的就是要减少内存碎片带来的效率损耗。在执行机制上JVM提供了串行GC(Serial MSC)、并行GC(Parallel MSC)和并发GC(CMS)。

Minor GC ，Full GC 触发条件

Minor GC触发条件：当Eden区满时，触发Minor GC。

Full GC触发条件：（1）调用System.gc时，系统建议执行Full GC，但是不必然执行（2）老年代空间不足（3）方法区空间不足（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存（5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

#### 8.HashMap 实现原理

在java编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针（引用），所有的数据结构都可以用这两个基本结构来构造的，HashMap也不例外。HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。

#### 9.java.util.concurrent 包下使用过哪些

1.阻塞队列 BlockingQueue( ArrayBlockingQueue, DelayQueue, LinkedBlockingQueue,SynchronousQueue,LinkedTransferQueue,LinkedBlockingDeque)

2.ConcurrentHashMap

3.Semaphore–信号量

4.CountDownLatch–闭锁

5.CyclicBarrier–栅栏

6.Exchanger–交换机

7.Executor->ThreadPoolExecutor,ScheduledThreadPoolExecutor



12345

Semaphore semaphore = new Semaphore(1);//critical section semaphore.acquire();...semaphore.release();

8.锁 Lock–ReentrantLock,ReadWriteLock,Condition,LockSupport



1234

Lock lock = new ReentrantLock();lock.lock();//critical section lock.unlock();

#### 10.concurrentMap 和 HashMap 区别

1.hashMap可以有null的键，concurrentMap不可以有

2.hashMap是线程不安全的，在多线程的时候需要Collections.synchronizedMap(hashMap),ConcurrentMap使用了重入锁保证线程安全。

3.在删除元素时候，两者的算法不一样。ConcurrentHashMap和Hashtable主要区别就是围绕着锁的粒度以及如何锁,可以简单理解成把一个大的HashTable分解成多个，形成了锁分离。

#### 11.信号量是什么，怎么使用?volatile关键字是什么？

信号量-semaphore：荷兰著名的计算机科学家Dijkstra 于1965年提出的一个同步机制。是在多线程环境下使用的一种设施, 它负责协调各个线程, 以保证它们能够正确、合理的使用公共资源。整形信号量：表示共享资源状态，且只能由特殊的**原子操作**改变整型量。同步与互斥：同类进程为互斥关系（打印机问题），不同进程为同步关系(消费者生产者)。

使用volatile关键字是解决同步问题的一种有效手段。 java volatile关键字预示着这个变量始终是“存储进入了主存”。更精确的表述就是每一次读一个volatile变量，都会从主存读取，而不是CPU的缓存。同样的道理，**每次写一个volatile变量，都是写回主存，而不仅仅是CPU的缓存**。Java 保证volatile关键字保证变量的改变对各个线程是可见的。

#### 12.阻塞队列了解吗？怎么使用

阻塞队列 (BlockingQueue)是Java util.concurrent包下重要的数据结构，BlockingQueue提供了线程安全的队列访问方式：当阻塞队列进行插入数据时，如果队列已满，线程将会阻塞等待直到队列非满；从阻塞队列取数据时，如果队列已空，线程将会阻塞等待直到队列非空。并发包下很多高级同步类的实现都是基于BlockingQueue实现的。

以ArrayBlockingQueue为例，我们先来看看代码：



```
public void put(E e) throws InterruptedException {

if (e == null) throw new NullPointerException();

final ReentrantLock lock = this.lock;lock.lockInterruptibly();

try {

while (count == items.length)

	notFull.await();

enqueue(e);

} finally {

	lock.unlock();

}

}
```



从put方法的实现可以看出，它先获取了锁，并且获取的是可中断锁，然后判断当前元素个数是否等于数组的长度，如果相等，则调用

`notFull.await()`

进行等待，当被其他线程唤醒时，通过enqueue(e)方法插入元素，最后解锁。

```
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    notEmpty.signal();
}
```

插入成功后，通过notEmpty**唤醒正在等待取元素的线程**。

#### 13.Java中的NIO，BIO，AIO分别是什么？

IO的方式通常分为几种，同步阻塞的BIO、同步非阻塞的NIO、异步非阻塞的AIO

1.BIO，同步阻塞式IO，简单理解：**一个连接一个线程**.BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。

在JDK1.4之前，用Java编写网络请求，都是建立一个ServerSocket，然后，客户端建立Socket时就会询问是否有线程可以处理，如果没有，**要么等待，要么被拒绝**。即：一个连接，要求Server对应一个处理线程。

2.NIO，同步非阻塞IO，简单理解：**一个请求一个线程,使用在selector多路复用器上进行轮询，若有消息则处理。**只有需要进行IO操作的才能获取服务端的处理线程进行IO。NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。

NIO本身是基于**事件驱动**思想来完成的，其主要想解决的是BIO的大并发问题： 在使用同步I/O的网络应用中，如果要同时处理多个客户端请求，或是在客户端要同时和多个服务器进行通讯，就必须使用多线程来处理。也就是说，将每一个客户端请求分配给一个线程来单独处理。这样做虽然可以达到我们的要求，但同时又会带来另外一个问题。由于每创建一个线程，就要为这个线程分配一定的内存空间（也叫工作存储器），而且操作系统本身也对线程的总数有一定的限制。如果客户端的请求过多，服务端程序可能会因为不堪重负而拒绝客户端的请求，甚至服务器可能会因此而瘫痪。

3.AIO，异步非阻塞IO，简单理解：**一个有效请求一个线程，即交给OS去处理**.NIO是同步的IO，是因为程序需要IO操作时，必须**获得了IO权限后亲自进行IO操作才能进行下一步操作。**它是基于**Proactor模型**的。每个socket连接在**事件分离器**注册 **IO完成事件** 和 **IO完成事件处理器**。程序需要进行IO时，向分离器发出IO请求并把所用的Buffer区域告知分离器，分离器通知**操作系统进行IO操作**，操作系统自己不断尝试获取IO权限并进行IO操作（数据保存在Buffer区），操作完成后通知分离器；分离器检测到 IO完成事件，则激活 IO完成事件处理器，处理器会通知程序说“IO已完成”，程序知道后就直接从Buffer区进行数据的读写。AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

**AIO是发出IO请求后，由操作系统自己去获取IO权限并进行IO操作；NIO则是发出IO请求后，由线程不断尝试获取IO权限，获取到后通知应用程序自己进行IO操作。**



#### 14.类加载机制是怎样的

JVM中类的装载是由ClassLoader和它的子类（AppClassLoader、ExtensionClassLoader和BootstrapClassLoader）来实现的,Java ClassLoader是一个重要的Java运行时系统组件。它负责在运行时查找和装入类文件的类。类加载的五个过程：加载、验证、准备、解析、初始化。

从类被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期分为7个阶段，加载(Loading)、验证(Verification)、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)、卸载(Unloading)。其中验证、准备、解析三个部分统称为连接。

#### 15.什么是幂等性

所谓幂等，简单地说，就是**对接口的多次调用所产生的结果和调用一次是一致的。**

那么我们为什么需要接口具有幂等性呢？设想一下以下情形：

在App中下订单的时候，点击确认之后，没反应，就又点击了几次。在这种情况下，如果无法保证该接口的幂等性，那么将会出现重复下单问题。在接收消息的时候，消息推送重复。如果处理消息的接口无法保证幂等，那么重复消费消息产生的影响可能会非常大。

#### 16.有哪些 JVM 调优经验

Jvm参数总结：http://linfengying.com/?p=2470

**内存参数**    

参数    作用

-Xmx堆大小的最大值。当前主流虚拟机的堆都是可扩展的

-Xms堆大小的最小值。可以设置成和 -Xmx 一样的值

-Xmn新生代的大小。现代虚拟机都是“分代”的，因此堆空间由新生代和老年代组成。新生代增大，相应地老年代就减小。Sun官方推荐新生代占整个堆的3/8

-Xss每个线程的堆栈大小。该值影响一台机器能够创建的线程数上限

-XX:MaxPermSize=永久代的最大值。永久代是 HotSpot 特有的，HotSpot **用永久代来实现方法区**

-XX:PermSize=永久代的最小值。可以设置成和 -XX:MaxPermSize 一样的值

-XX:SurvivorRatio=Eden 和 Survivor 的比值。基于“复制”的垃圾收集器又会把新生代分为一个 Eden 和两个 Survivor，如果该参数为8，就表示 Eden占新生代的80%，而两个 Survivor 各占10%。默认值为8

-XX:PretenureSizeThreshold=直接晋升到老年代的对象大小。大于这个参数的对象将直接在老年代分配。默认值为0，表示不启用

-XX:HandlePromotionFailure=是否允许分配担保失败。在 JDK 6 Update 24 后该参数已经失效。

-XX:MaxTenuringThreshold=对象晋升到老年代的年龄。对象每经过一次 Minor GC 后年龄就加1，超过这个值时就进入老年代。默认值为15

-XX:MaxDirectMemorySize=直接内存的最大值。对于频繁使用 nio 的应用，应该显式设置该参数，默认值为0

**GC参数**

垃圾收集器    参数    备注



Serial（新生代）-XX:+UseSerialGC虚拟机在 Client 模式下的默认值，打开此开关后，使用 Serial + Serial Old 的收集器组合。Serial 是一个单线程的收集器

ParNew（新生代）-XX:+UseParNewGC强制使用 ParNew，打开此开关后，使用 ParNew + Serial Old 的收集器组合。ParNew 是一个多线程的收集器，也是 server 模式下首选的新生代收集器

-XX:ParallelGCThreads=垃圾收集的线程数

Parallel Scavenge（新生代）-XX:+UseParallelGC虚拟机在 Server 模式下的默认值，打开此开关后，使用 Parallel Scavenge + Serial Old 的收集器组合

-XX:MaxGCPauseMillis=单位毫秒，收集器尽可能保证单次内存回收停顿的时间不超过这个值。

-XX:GCTimeRatio=总的用于 gc 的时间占应用程序的百分比，该参数用于控制程序的吞吐量

-XX:+UseAdaptiveSizePolicy设置了这个参数后，就不再需要指定新生代的大小（-Xmn）、 Eden 和 Survisor 的比例（-XX:SurvivorRatio）以及晋升老年代对象的年龄（-XX:PretenureSizeThreshold）了，因为该收集器会根据当前系统的运行情况自动调整。当然前提是先设置好前两个参数。

Serial Old（老年代）无Serial Old 是 Serial 的老年代版本，主要用于 Client 模式下的老生代收集，同时也是 CMS 在发生 Concurrent Mode Failure 时的后备方案

Parallel Old（老年代）-XX:+UseParallelOldGC打开此开关后，使用 Parallel Scavenge + Parallel Old 的收集器组合。Parallel Old 是 Parallel Scavenge 的老年代版本，在注重吞吐量和 CPU 资源敏感的场合，可以优先考虑这个组合

CMS（老年代）-XX:+UseConcMarkSweepGC打开此开关后，使用 ParNew + CMS 的收集器组合。

-XX:CMSInitiatingOccupancyFraction=CMS 收集器在老年代空间被使用多少后触发垃圾收集

-XX:+UseCMSCompactAtFullCollection在完成垃圾收集后是否要进行一次内存碎片整理

-XX:CMSFullGCsBeforeCompaction=在进行若干次垃圾收集后才进行一次内存碎片整理

附图：可以配合使用的收集器组合

上面有7中收集器，分为两块，上面为新生代收集器，下面是老年代收集器。如果两个收集器之间存在连线，就说明它们可以搭配使用。

**其他参数**

参数    作用



-verbose:class打印类加载过程

-XX:+PrintGCDetails发生垃圾收集时打印 gc 日志，该参数会自动带上 -verbose:gc 和 -XX:+PrintGC

-XX:+PrintGCDateStamps / -XX:+PrintGCTimeStamps打印 gc 的触发事件，可以和 -XX:+PrintGC 和 -XX:+PrintGCDetails 混用

-Xloggc:gc 日志路径

-XX:+HeapDumpOnOutOfMemoryError出现 OOM 时 dump 出内存快照用于事后分析

-XX:HeapDumpPath=堆转储快照的文件路径

#### 17.分布式 CAP 了解吗？

一致性(Consistency)可用性(Availability)分区容忍性(Partition tolerance)，最多只能**同时保证两个特性**。

#### 18.Java中HashMap的key值要是为类对象则该类需要满足什么条件？

需要同时重写该类的hashCode()方法和它的equals()方法。

当程序试图将一个 key-value 对放入 HashMap 中时，程序首先根据该 key 的 hashCode() 返回值决定该 Entry 的存储位置：如果两个 Entry 的 key 的 hashCode() 返回值相同，那它们的存储位置相同。如果这两个 Entry 的 key 通过 equals 比较返回 true，新添加 Entry 的 value 将覆盖集合中原有 Entry 的 value，但 key 不会覆盖。如果这两个 Entry 的 key 通过 equals 比较返回 false，新添加的 Entry 将与集合中原有 Entry 形成 Entry 链，**而且新节点是添加到链表的尾部。**

#### 19.java 垃圾回收会出现不可回收的对象吗？怎么解决内存泄露问题？怎么定位问题源？

一般不会有不可回收的对象，因为现在的GC会回收不可达内存。

#### 20.终止线程有几种方式？终止线程标记变量为什么是 valotile 类型？

1.线程正常执行完毕，正常结束

2.监视某些条件，结束线程的不间断运行

3.使用interrupt方法终止线程

在定义exit时，使用了一个Java关键字volatile，这个关键字的目的**是使exit同步**，也就是说在**同一时刻只能由一个线程来修改exit的值**

```
private volatile int threadStatus = 0;
```

#### 21.用过哪些并发的数据结构？ cyclicBarrier 什么功能？信号量作用？数据库读写阻塞怎么解决

主要有锁机制，然后**基于CAS**的concurrent包。

**CyclicBarrier**的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，**直到最后一个线程到达屏障时，屏障才会开门**，所有被屏障拦截的线程才会继续干活。CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。内部使用**ReentrantLock的condition**实现。

**CountDownLatch**的计数器**只能使用一次**。而CyclicBarrier的计数器可以使用reset() 方法重置。内部使用**Sync类的CAS和AQS的CLH同步队列（双向链表实现）**实现。

**Semaphore**（信号量）是用来控制同时访问**特定资源的线程数量**，它通过协调各个线程，以保证合理的使用公共资源。很多年以来，我都觉得从字面上很难理解Semaphore所表达的含义，只能把它比作是控制流量的红绿灯，比如XX马路要限制流量，只允许同时有一百辆车在这条路上行使，其他的都必须在路口等待，所以前一百辆车会看到绿灯，可以开进这条马路，后面的车会看到红灯，不能驶入XX马路，但是如果前一百辆中有五辆车已经离开了XX马路，那么后面就允许有5辆车驶入马路，这个例子里说的车就是线程，驶入马路就表示线程在执行，离开马路就表示线程执行完成，看见红灯就表示线程被阻塞，不能执行。使用**Sync类的tryAcquireShared()**实现。

#### 22.双重检查锁为什么不一定能保证线程安全(为什么必须使用volatile修饰)

因为指令重排序，即在执行new指令时，分配内存空间、初始化对象和将引用指向内存地址会发生**重排序，**即初始化对象和引用赋值的步骤重排，导致另一个线程**外层判定时非空**直接使用，而实际上并未完全初始化。

#### 23.哪些对象可以当做GC ROOTS？

栈中引用的对象（栈帧中的本地变量表，即局部变量表）、方法区中的静态属性引用的对象、方法区中常量引用的对象

#### 24.对象一定在堆上分配吗？

不一定。有个技术叫**逃逸分析**：它是指当一个对象**在方法中被定义**之后，它如果被外部方法引用，则称为逃逸。那么，如果这个对象不发生逃逸，它的生命周期就和它的引用存活一样长的时间，在引用出栈的时候就可以顺带销毁对象了，这样以来既可以加快访问速度，也可以减小GC机制的压力。

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170326114922686?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjQwMzI5MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 25.类加载三种方式的区别

虚拟机加载类的途径 
1、由 new 关键字创建一个类的实例 
在由运行时刻用 new 方法载入 
如：Dog dog ＝ new Dog（）； 
2、调用 Class.forName() 方法 
通过反射加载类型，并创建对象实例 
如：Class clazz ＝ Class.forName（“Dog”）； 
Object dog ＝clazz.newInstance（）； 
3、调用某个 ClassLoader 实例的 loadClass() 方法 
通过该 ClassLoader 实例的 loadClass() 方法载入。应用程序可以通过继承 ClassLoader 实现自己的类装载器。 
如：Class clazz ＝ classLoader.loadClass（“Dog”）； 
Object dog ＝clazz.newInstance（）； 
三者的区别： 
1和2使用的类加载器是相同的，都是当前类加载器。（即：this.getClass.getClassLoader）。3由用户指定类加载器。如果需要在当前类路径以外寻找类，则只能采用第3种方式。第3种方式加载的类与当前类分属不同的命名空间。另外，1是静态加载，2、3是动态加载

两个异常(exception) 
静态加载的时候如果在运行环境中找不到要初始化的类,抛出的是NoClassDefFoundError,它在JAVA的异常体系中是一个Error 
动态加载的时候如果在运行环境中找不到要初始化的类,抛出的是ClassNotFoundException,它在JAVA的异常体系中是一个checked异常

##### Class.forName与ClassLoader.loadClass区别 

Class的装载包括3个步骤：加载（loading）,连接（link）,初始化（initialize）. 
Class.forName(className)实际上是调用Class.forName(className, true, this.getClass().getClassLoader())。第二个参数，是指Class被loading后是不是必须被初始化。 
ClassLoader.loadClass(className)实际上调用的是ClassLoader.loadClass(name, false)，第二个参数指Class是否被link。 
Class.forName(className)装载的class已经被初始化，而ClassLoader.loadClass(className)装载的class还没有被link。一般情况下，这两个方法效果一样，都能装载Class。但如果程序依赖于Class是否被初始化，就必须用Class.forName(name)了。 
例如，在JDBC编程中，常看到这样的用法，Class.forName(“com.mysql.jdbc.Driver”). 
如果换成了getClass().getClassLoader().loadClass(“com.mysql.jdbc.Driver”)，就不行。 
com.mysql.jdbc.Driver的源代码如下： 
// Register ourselves with the DriverManager 
static { 
try { 
java.sql.DriverManager.registerDriver(new Driver()); 
} catch (SQLException E) { 
throw new RuntimeException(“Can’t register driver!”); 
} 
} 
原来，Driver在static块中会注册自己到java.sql.DriverManager。而static块就是在Class的初始化中被执行。 
所以这个地方就只能用Class.forName(className)。

对于相同的类，JVM最多会载入一次。但如果同一个class文件被不同的ClassLoader载入，那么载入后的两个类是完全不同的。因为已被加载的类由该类的类加载器实例与该类的全路径名的组合标识。设有 packagename.A Class ，分别被类加载器 CL1 和 CL2 加载，所以系统中有两个不同的 java.lang.Class 实例：

#### 26.什么时候会进行full gc？

1.System.gc()方法调用，但不是一定

2.老年代空间不足

3.永生代空间不足，主要存放类信息、常量和静态变量

4.统计得到的Minor GC晋升到旧生代的平均大小大于旧生代的剩余空间

#### 27.如何查看gc日志避免频繁 full gc

先用jps查看有哪些java进程，获取pid,然后jinfo pid 来查看这个进程的java信息，然后查看到参数 -Xloggc,即可找到gc log的存放位置。

查看gc日志，尽量避免创建大对象和大数组，增加survivor区大小，增加老年代大小

#### 28.mysql存储引擎区别

**MyISAM：**

1. 不支持事务，但是每次查询都是原子的；
2. 支持表级锁，即每次操作是对整个表加锁；
3. 存储表的总行数；
4. 一个MYISAM表有三个文件：索引文件、表结构文件、数据文件；
5. 采用非聚集索引，索引文件的数据域**存储指向数据文件的指针**。**辅索引与主索引基本一致**，但是辅索引不用保证唯一性。

**InnoDb：**

1. 支持ACID的事务，支持事务的四种隔离级别；
2. 支持行级锁及外键约束：因此可以支持写并发；
3. 不存储总行数；
4. 一个InnoDb引擎存储在一个文件空间（共享表空间，表大小不受操作系统控制，一个表可能分布在多个文件里），也有可能为多个（设置为独立表空，表大小受操作系统文件大小限制，一般为2G），受操作系统文件大小的限制；
5. **主键索引采用聚集索引（索引的数据域存储数据文件本身），辅索引的数据域存储主键的值**；因此从辅索引查找数据，需要先通过辅索引找到主键值，再访问辅索引；最好使用自增主键，防止插入数据时，为维持B+树结构，文件的大调整。

#### 29.数据库的四种事务隔离级别

开启事务：start transaction,rollback,commit

设置隔离级别明星：set tx_isolation='read-uncommitted',mysql默认为**Repeatable read (可重复读)**

​	① Serializable (串行化)：可避免脏读、不可重复读、幻读的发生。

　　② Repeatable read (可重复读)：可避免脏读、不可重复读的发生。

　　③ Read committed (读已提交)：可避免脏读的发生。

　　④ Read uncommitted (读未提交)：最低级别，任何情况都无法保证。

#### 30.redis和memcache的区别

 **Memcache**

   Memcache可以利用多核优势，单实例吞吐量极高，可以达到几十万QPS,适用于最大程度扛量

​    只支持**简单的key/value**数据结构，不像Redis可以支持丰富的数据类型。

​    无法进行**持久化**，数据不能备份，只能用于缓存使用，且重启后数据全部丢失

 

 **Redis**

​    支持多种数据结构，如string,list,dict,set,zset,hyperloglog

​    **单线程请求**，**所有命令串行执行，并发情况下不需要考虑数据一致性问题**。

​    支持持久化操作，可以进行aof及rdb数据持久化到磁盘，从而进行数据备份或数据恢复等操作，较好的防止数据丢失的手段。

​    支持通过Replication进行数据复制，通过master-slave机制，可以实时进行数据的同步复制，支持多级复制和增量复制.

​    支持pub/sub消息订阅机制，可以用来进行消息订阅与通知。

​    支持简单的事务需求，但业界使用场景很少，并不成熟

#### 31.redis的底层存储结构

redis默认 初始化16个数据库，每一个数据库都一个**redisDb**的结构进行存储，redisDb.id即对应数据库的id，为0-15的整数，**redisDb.dict**存储着该数据库的所有键值对数据；redisDb.expires存储着每一个键的过期时间

redisDb.dict即字典类型（用c实现），使用**哈希表**作为底层实现。dict 类型使用的两个指向哈希表的指针，其中 0 号哈希表（ht[0]）主要**用于存储数据库的所有键值**，而1号哈希表主要用于程序对 0 号哈希表**进行 rehash 时使用**，rehash 一般是**在添加新值时会触发**，这里不做过多的赘述。所以redis 中查找一个key，其实就是对进行该dict 结构中的 ht[0] 进行查找操作。

采用**链地址法**处理哈希碰撞。

#### 32.redis如何根据key查找数据？

1、当拿到一个key后， redis 先判断当前库的0号哈希表是否为空，即：if (dict->ht[0].size == 0)。如果为true直接返回NULL。

2、判断该0号哈希表是否需要rehash，因为如果在进行rehash，那么两个表中者有可能存储该key。如果正在进行rehash，将调用一次_dictRehashStep方法，_dictRehashStep 用于对数据库字典、以及哈希键的字典进行被动 rehash，这里不作赘述。

3、计算哈希表，根据当前字典与key进行哈希值的计算。

4、根据哈希值与当前字典计算哈希表的索引值。

5、根据索引值在哈希表中取出链表，遍历该链表找到key的位置。一般情况，该链表长度为1。

6、当 ht[0] 查找完了之后，**再进行了次rehash判断，如果未在rehashing，则直接结束**，否则对ht[1]重复345步骤。

#### 33.redis的分布式锁设计思路

https://blog.csdn.net/guangbo0301/article/details/73385398

具体实现代码：https://zm10.sm-tc.cn/?src=l4uLj4zF0NCIiIjRnJGdk5CYjNGckJLQk5aKhp6RmM%2FQno2LlpyTmozQycjLy8%2FIydGXi5KT&uid=7c8c484529ff6dec0a1c80a0d920c08b&hid=b13e343fe55970d18f94121bd73d34ee&pos=2&cid=9&time=1542877133983&from=click&restype=1&pagetype=0000000000000000&bu=web&query=redis%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81&mode=&v=1&force=true&wap=false&province=%E5%9B%9B%E5%B7%9D%E7%9C%81&city=%E6%88%90%E9%83%BD%E5%B8%82&uc_param_str=dnntnwvepffrgibijbprsvdsdichei

**实现方式1** 
  **方式**：incr、decr 原子操作 
   **加锁**：在需要使用的地方执行该key的incr操作，如果返回值是1，则获取锁 
   **解锁**：在finally块中将key做decr操作 
   **设置过期时间**：如果进程挂掉，导致锁没有释放，自动过期删除 
   **优点**：操作简单易行 
   **缺点**： 
   1.设置过期时间正确 
​      获取锁的客户端进程在执行过程中挂掉，没有走finally块减一，那其他进程只能等redis的ttl自动删除; 
​      该过期时间设置的长短难以把控，如果我们的请求因为其他原因阻塞了没有处理完，但已经到了redis的过期时间，
​      其他进程可以获得锁进行   处理，结果。。。 
   2.设置过期时间不正确 
​      设置key的自增和设置过期时间不是原子操作，假如前者设置成功了，而过期时间因为各种原因没有设置成功，

​      一旦该锁的计数出现错误，那么所有进程都无法获取到锁，结果。。。

 **总结上面方式所存在的问题：** 
 **1.操作非原子性** 
 **2.网络中断、命令发送失败** 
 **3.死锁** 
 **4.互斥**

**实现方式2**

SETNX key val，若返回1，则表示设置成功，key不存在，若返回0，则表示已存在。

expire:设置锁超时时间，避免死锁未释放

delete key：删除锁，即释放锁



#### 34.如何防止死锁产生？

对于synchronized产生的死锁，似乎我们无能为力，即死锁状态无法解除； 
​    一个线程已经获得了对象锁，其他线程访问共享对象的时就必须无限期等待，不能中断那些获取锁的线 程。 
​    因此我们编码时**让线程按照相同的顺序获得一组锁**进行预防。 
​    而Lock提供了更加灵活的方法，如果当前锁可用则返回true，否则返回false，并且可以**设置获取锁的超时 时间，超时退出，防止死锁。** 
​    对于分布式的锁第一要设置锁的超时时间，让锁能及时释放掉。 

​    其次还要设置客户端请求锁的超时时间，以防止通信过程出现问题，客户端线程一直等待锁响应

#### 35.单线程（多路复用模型）的redis为什么比多线程的memcache快？

​	1、完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)；
　　
　　2、数据结构简单，对数据操作也简单，Redis中的数据结构是专门进行设计的；
　　
　　3、采用**单线程**，**避免了不必要的上下文切换和竞争条件**，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在**加锁释放锁**操作，没有因为可能出现死锁而导致的性能消耗；
　　
　　4、使用**多路I/O复用模型，非阻塞IO**；
　　
　　5、使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求；

而且线程不是影响吞吐量的重要因素。如第一点来说，一般情况下，程序处理内存数据的速度远高于网卡接收的速度。使用多线程好处是可以同时处理多条连接，在极端情况下，可能会提高响应速度。

#### 36.为什么nginx比apache快？

因为nginx采用**异步非阻塞**，而apache采用**同步阻塞**。每一个连接，apache就会创建一个进程，每个进程内单线程，apache最多能创建256个进程。对于一个负载相对较高的网站来说，256的进程，也就是256个线程，因为线程处理请求时，是同步阻塞模式，**接收请求之后，会一直等待该请求读取程序文件（IO）（同步），执行业务逻辑，返回客户端**，所有操作完成之后才能处理下一个请求（阻塞）如果服务器已经达到256的极限，那么接下去的访问就需要排队这也就是为什么某些服务器负载不高的原因了。



**同步和异步的概念描述的是用户线程与内核的交互方式**

**阻塞和非阻塞的概念描述的是用户线程调用内核IO操作的方式**

**异步非阻塞跟IO多路复用的区别就是**：IO多路复用select时会阻塞，同时事件发生后进行IO读写时**是由当前线程读写**，而异步非阻塞则是OS读写并放入指定缓冲区后通知当前线程使用。



#### 37.什么是多路I/O复用模型？

　　多路I/O复用模型是利用 select、poll、epoll 可以同时监察多个流的 I/O 事件的能力，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有 I/O 事件时，就从阻塞态中唤醒，于是程序就会轮询一遍所有的流（epoll 是只轮询那些真正发出了事件的流），并且只依次顺序的处理就绪的流，这种做法就避免了大量的无用操作。

而**非阻塞I/O**则会出现多个线程**等待数据完全准备好导致的“忙等”现象**　　
　　这里“多路”指的是**多个网络连接**，“复用”指的是**复用同一个线程**。采用多路 I/O 复用技术可以让**单个线程高效的处理多个连接请求（尽量减少网络 IO 的时间消耗）**，且 Redis 在内存中操作数据的速度非常快，也就是说**内存内的操作不会成为影响Redis性能的瓶颈**，主要由以上几点造就了 Redis 具有很高的吞吐量。

#### 38.为什么redis是单线程？

因为Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是**机器内存的大小或者网络带宽**。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了（毕竟采用多线程会有很多麻烦！）。但会导致无法充分利用多核CPU（可以在本地创建多个redis解决）

#### 39.分布式事务

https://blog.csdn.net/u010425776/article/details/79516298

考虑到分布式必然会想到CAP理论。分布式事务主要是不同的操作处于不同的子系统中。

**分区容错性**是分布式系统的根本，如果分区容错性不能满足，那使用分布式系统将失去意义。因此，我们只能通过牺牲一致性（能保证弱一致性）来换取系统的**可用性**和**分区容错性**

#### BASE理论

BA：Basic Available 基本可用 
整个系统在某些不可抗力的情况下，仍然能够保证“可用性”，即一定时间内仍然能够返回一个明确的结果。只不过“基本可用”和“高可用”的区别是：
“一定时间”可以适当延长 
当举行大促时，响应时间可以适当延长
给部分用户返回一个降级页面 
给部分用户直接返回一个降级页面，从而缓解服务器压力。但要注意，返回降级页面仍然是返回明确结果。
S：Soft State：柔性状态 
同一数据的不同副本的状态，可以不需要实时一致。
E：Eventual Consisstency：最终一致性 

同一数据的不同副本的状态，可以不需要实时一致，但一定要保证经过一定时间后仍然是一致的。

为了保证分布式情况下的**一致性**，使用了**两阶段提交协议**：

**分布式事务协议：**

两阶段提交协议的每一次事务提交分为**两个阶段**：

　　在第一阶段，**协调者**询问所有的参与者是否可以提交事务（请参与者**投票**），所有参与者向协调者投票。

　　在第二阶段，**协调者**根据所有参与者的投票结果做出是否事务可以全局提交的决定，并通知所有的参与者执行该决定。在一个两阶段提交流程中，参与者不能改变自己的投票结果。两阶段提交协议的可以全局提交的**前提是所有的参与者都同意提交事务，只要有一个参与者投票选择放弃(abort)事务，则事务必须被放弃。**



**分布式事务解决方案：**

- 全局消息
- 基于**可靠消息服务**的分布式事务，即**把发送消息和业务操作封装成一个本地事务，其他系统消费消息即可，失败则阿松一条业务补偿信息，通知生产者进行回滚。**或者RocketMQ支持的事务消息
- TCC
- 最大努力通知

主要考虑**消息中间件**的解决方式：

![title](https://user-gold-cdn.xitu.io/2018/3/10/1620fc30782107d1?w=383&h=474&f=png&s=26178)

![title](https://user-gold-cdn.xitu.io/2018/3/10/1620fc307be6c55d?w=336&h=472&f=png&s=22438)

上面所介绍的Commit和Rollback都属于理想情况，但在实际系统中，**Commit和Rollback指令都有可能在传输途中丢失。**那么当出现这种情况的时候，消息中间件是如何保证数据一致性呢？——答案就是**超时询问机制**。

![title](https://user-gold-cdn.xitu.io/2018/3/10/1620fc307c2d185e?w=417&h=549&f=png&s=27636)

![title](https://user-gold-cdn.xitu.io/2018/3/10/1620fc3078194980?w=440&h=541&f=png&s=30312)

![title](https://user-gold-cdn.xitu.io/2018/3/10/1620fc308ece8d2a?w=432&h=598&f=png&s=32888)

#### 38.Runnable和Callable的区别

1. 两者最大的不同点是：实现Callable接口的任务线程能返回执行结果；而实现Runnable接口的任务线程不能返回结果；
2. Callable接口的call()方法允许抛出异常；而Runnable接口的run()方法的异常只能在内部消化，不能继续上抛；

#### 39.HashMap1.7与1.8的区别

![1543324027225](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543324027225.png)

**为什么使用红黑树？**

因为兼顾插入和查询的速度，插入和查询速度界于完全平衡排序树和链表之间，因为插入调整没那么多。

**为什么从尾部插入？**

因为jdk8中始终要通过遍历+链表检查长度是否 超过8,所以直接插在尾部

HashMap将链表变为红黑树的条件，除了**大于8**外，还必须是**table数组长度超过64，若未超过，则直接resize（）扩容一次**。

红黑树转换为链表时的条件，是小于等于6，

- 1. **处理哈希码**的方式
     1.7处理hashCode采用了4次位运算加5次异或运算
     1.8处理hashCode采用了仅使用了一次位运算和一次异或运算，key如果为null,计算结果为0
- 2. **扩容时链表的插入方式**
     1.7采用头插法，扩容的时候会造成链表逆序，容易出现环形链表
     并发插入时会出现数据丢失，因为并发时拿到的链头可能不是最新的链头，会出现后面的覆盖掉前面数据的情况，且扩容条件：**超过阈值且插入table处不为空**，在扩容是会有**死锁问题**（<http://www.importnew.com/22011.html>）
     1.8采用**尾插法**，不会出现链表逆序，不容易出现环形链表
- 3. **数据结构**
     1.7采用数组+链表
     1.8采用数组+链表+红黑树

#### 40.ConcurrentHashMap和HashTable

- ConcurrentHashMap与HashTable都是线程安全的容器，在面对多线程竞争激烈的情况下，HashTable的性能相比于ConcurrentHashMap要逊色的多，原因在于HashTable采用的是synchronize对整个容器进行加锁，而ConcurrentHashMap采用的是锁分段技术，大大提高了多线程下的读写性能。
- 1. ConcurrentHashMap由多个Segment组成，而每个Segment又由多个HashEntry组成
- 2. 每个Segment都配有一把锁，对某个Segment进行加锁的同时不影响其他Segment的读写
- 3. **get()操作不用加锁**，原因在于HashEntry***享变量count,value都是**volatile**类型的，即使在读的时候有一个线程在写，也能保证读取到的value是最新的，因为volatile保证了Java内存模型中的happen-before原则，即写操作总是优先于读操作的
- 4. get()时会先进行一次再散列运算定位到某个Segment，然后再进行一次再散列定位至某个HashEntry，之后进行读操作即可
- 5. put()操作需要加锁，这是为了防止同时有两个线程同时对某个HashEtry进行写操作
- 6. put()操作同样需要先进行一次再散列定位至某个Segment，然后判断其是否需要扩容，如果需要扩容,待其进行扩容操作后再进行插入操作
- 7. size()操作先采用两次不加锁的方式统计每个Segment的count，如果统计的过程中发现count发生了,变化，再采用加锁的方式统计每个Segment的count

#### 41.ConcurrentHashMap1.7与1.8的区别

1. 数据结构不同
   1.7采用Segment+HashEntry进行存储
   1.8采用数组+链表+红黑树
2. 保证并发的方式不同
   1.7采用的**锁分段**技术保证并发
   1.8大量采用**CAS**保证并发
3. 初始化map的时机不同
   1.7在构造的时候进行初始化
   1.8在插入的时候进行初始化，通过一个volatile变量sizeCtl保证初始化线程安全，通过CAS进行修改值

#### 42.Lock与synchronized的区别

https://blog.csdn.net/u012403290/article/details/64910926

![img](file:///C:\Users\ASUS\AppData\Roaming\Tencent\Users\739868197\TIM\WinTemp\RichOle\[E0HSF6$L546PM8[}%YF817.png)



- **Lock可以中断式的获取锁**：lockInterruptly()，在自旋获取锁或者加入等待队列时可相应中断，取消尝试。或者trylock(3,TimeUnit.Seconds)，在超时时间内相应中断。
- **Lock可以尝试非阻塞的获取锁**：trylock()，立即返回是否可以获取锁。
- **Lock可以超时的获取锁**：trylock(3,TimeUnit.Seconds)

synchronize有一个monitorenter和两个monitorexit指令，对应线程执行的可能的两条路径，即执行完释放和异常时释放。



#### 43.JAVA中多线程的通信方式

1.共享内存机制：即共享变量，例如synchronized修饰，或者线程中while(true)+volatile实现共享条件白能量

2.消息通信机制：例如wait/notify

3.管道通信：使用java.io.PipedInputStream 和 java.io.PipedOutputStream进行通信

#### 44.什么是死锁？如何解决？

进程集合中的每个进程都在等待另一个进程释放资源而造成的无限等待下去的僵持局面
**解决方案**：1. 通过一个监控系统监控死锁的存在，如果存在死锁直接抛异常或报错2. 超时的获取锁，如果一段时间内获取不到锁就放弃获取锁的这个请求

#### 45. Spring中存在什么样的设计模式

- 适配器模式(HandlerAdpter,根据HandlerMapper找到对应的Adpter)
- 单例模式(容器中存在的实例基本都是单例,SpringMVC中的Controller)
- 工厂模式(BeanFactory)
- 装饰器模式(用来设置JavaBean属性的BeanWrapper)
- 模板模式(JDBCTemplate,1.获取数据库连接2.创建执行语句3.执行SQL语句4.处理结果集5.释放资源)

#### 46.使用Statement和PreparedStatement有什么区别？

- 1. 使用Statement**每次查询都会进行编译**，一次查询只会产生一次网络请求，而且安全性很差
- 2. 使用PreparedStatement，对于相似的SQL只会编译一次，编译之后只要**缓存命中**，那么就可以**跳过编译**阶段直接运行，并且SQL语句**使用占位符**，提高了代码的可读性和安全性
- 3. 可以使用preparedStatement.addBatch()对SQL语句**进行批处理**，也就是一次给DB发送多条SQL，这样可以减少网络请求

#### 47.子类继承父类的方法调用过程和变量访问过程

**方法调用（动态绑定）**

如子类引用c,父类引用为b，令b=c，执行c.action，子类未重写该方法

![img](https://images2015.cnblogs.com/blog/924211/201605/924211-20160528161043709-1454488203.jpg)



1. 查看c的对象类型，找到Child类型，在Child类型中找action方法，发现没有，**到父类中寻找**
2. 在父类Base中找到了方法action，开始执行action方法
3. action先输出了start，然后发现需要调用step()方法，就从Child类型开始寻找step方法
4. 在Child类型中找到了step()方法，执行Child中的step()方法，执行完后返回action方法
5. 继续执行action方法，输出end

**寻找要执行的实例方法的时候，是从对象的实际类型信息开始查找的，找不到的时候，再查找父类类型信息。**

我们来看b.action();，这句代码的输出和c.action是一样的，这称之为**动态绑定**，而动态绑定实现的机制，就是根据对象的实际类型查找要执行的方法，子类型中找不到的时候再查找父类。这里，因为b和c指向相同的对象，所以执行结果是一样的。



**变量访问（静态绑定）**

**对变量的访问是静态绑定的**，无论是类变量还是实例变量。代码中演示的是类变量：b.s和c.s，通过对象访问类变量，系统会转换为**直接访问类变量Base.s和Child.s**。

例子中的实例变量都是private的，不能直接访问，如果是public的，则b.a访问的是对象中Base类定义的实例变量a，而c.a访问的是对象中Child类定义的实例变量a。



#### 48.为什么JDK动态代理局限于接口？

因为JDK动态代理proxyTest测试类继承了**Proxy**，然而在Java中只支持**单继承**，但是可以实现多个接口，所以JDK动态代理只能局限于接口。（**实现InnvocationHandler**）

而CGlib动态代理使用了底层的**字节码技术**，其原理是通过字节码技术**为一个类创建子类**，并在**子类中采用方法拦截的技术拦截所有父类方法的调用**，顺势织入和横切逻辑(**实现MethodInterceptor接口**)，因此**final修饰的类无法进行代理和执行，而被final修饰的方法可以执行，但不能代理**。final修饰的方法**只能继承，不能重写**。如何理解？因为final修饰即表示final修饰的方法调用地址固定，因此，执行proxy.save()方法是直接指向被代理类的方位位置。

#### 49.有哪些锁类型

Java中的锁都是基于**队列同步器AQS**实现的

1. 独占锁
   独占锁同一时间只允许一个线程获取到锁
2. 共享锁
   共享锁同一时间可允许多个线程获取到锁
3. 可重入锁  
   可重入锁允许一个线程获取到锁之后再次获取锁，即保证获取到锁的线程不会被自己阻塞
   同时可重入锁支持公平锁和非公平锁
4. 读写锁
   读锁是共享锁，写锁是排他锁，通过对一个32位整形类型的数值表达了读和写的两种状态。

#### 50.LinkedHashMap

**LinkedList(双向链表)+HashMap实现**，默认**不自动删除**过期数据，需要重写**removeEldestEntry**方法确定什么时候执行淘汰策略。

https://blog.csdn.net/zxt0601/article/details/77429150

在每次**插入数据，或者访问、修改数据时，**会调整链表的节点顺序。以决定迭代时输出的顺序。

**accessOrder** ,默认是false，则迭代时输出的顺序是插入节点的顺序。若为true，则输出的顺序是按照访问节点的顺序。为true时，可以在这基础之上构建一个LruCache.
LinkedHashMap并**没有重写任何put方法**。但是其重写了构建新节点的**newNode()**方法.在每次构建新节点时，将新节点链接在内部**双向链表的尾部**
accessOrder=true的模式下,在**afterNodeAccess()**函数中，会将当前被访问到的节点e，移动至内部的双向链表的尾部。值得注意的是，**afterNodeAccess()函数中，会修改modCount**,因此当你正在accessOrder=true的模式下,迭代LinkedHashMap时，如果同时查询访问数据，也会导致**fail-fast**，因为迭代的顺序已经改变。
**nextNode()** 就是迭代器里的next()方法 。 
该方法的实现可以看出，迭代LinkedHashMap，就是从内部维护的双链表的表头开始循环输出。 
而双链表节点的顺序在LinkedHashMap的增、删、改、查时都会更新。以满足按照插入顺序输出，还是访问顺序输出。

它与HashMap比，还有一个小小的优化，重写了containsValue()方法，直接遍历内部链表去比对value值是否相等。



由于HashMap的迭代顺序是**无序**的，因此引入了**LinkedHashMap**通过维护一个运行于所有条目的**双向链表**，LinkedHashMap**保证了元素迭代的顺序**。该迭代顺序可以是**插入顺序**或者是**访问顺序**。

| **关  注  点**                | **结      论**               |
| ----------------------------- | ---------------------------- |
| LinkedHashMap是否允许空       | Key和Value都允许空           |
| LinkedHashMap是否允许重复数据 | Key重复会覆盖、Value允许重复 |
| LinkedHashMap是否有序         | **有序**                     |
| LinkedHashMap是否线程安全     | 非线程安全                   |

默认采用**插入顺序**来维持取出键值对的次序。所有构造方法都是通过**调用父类（即HashMap）的构造方法来创建对象**的。可以设定参数accessOrder，true表示**最近最少使用次序**，false表示**插入顺序**

![img](https://images2015.cnblogs.com/blog/249993/201612/249993-20161213140338917-602479781.png)

![img](https://images2015.cnblogs.com/blog/249993/201612/249993-20161215143120620-1544337380.png)

![img](https://images2015.cnblogs.com/blog/249993/201612/249993-20161215143544401-1850524627.jpg)

第一张图为LinkedHashMap整体结构图，第二张图专门把**循环双向链表**抽取出来，直观一点，注意该循环双向链表的头部存放的是**最久访问的节点或最先插入**的节点，尾部为**最近访问的或最近插入**的节点，迭代器遍历方向是从链表的头部开始到链表尾部结束，在链表尾部有一个**空的header节点**，该节点不存放key-value内容，hash值为-1，为LinkedHashMap类的成员属性，循环双向链表的**入口**。

**从尾部插入节点，从头删除节点。**`LinkedHashMap`重写了HashMap中的put方法调用的`newNode()`,在每次**构建新节点**时，通过`linkNodeLast(p);`将**新节点链接在内部双向链表的尾部**。

`HashMap`专门预留给`LinkedHashMap`的`afterNodeAccess() afterNodeInsertion() afterNodeRemoval()`作为**回调方法**使用。例如调用hashmap的put方法中会调用linkedHashMap中实现的**afterNodeInsertion()**方法，从而实现回调。

在`afterNodeAccess()`函数中，**会将当前被访问到的节点e，移动至内部的双向链表的尾部。**`afterNodeAccess()`函数中，会修改`modCount`,因此当你正在`accessOrder=true`的模式下,迭代`LinkedHashMap`时，如果同时查询访问数据，也会导致`fail-fast`，因为迭代的顺序已经改变。

#### 51.Tomcat默认的GC收集器

http://iamzhongyong.iteye.com/blog/1447314 JVM默认参数

查看方式：运行tomcat7，然后执行命令行**jconsole**可选择查看对应java进程的配置方式，**tomcat7默认为parallel scavenge 和CMS**

一般是parallel scavenge 和parallel old，但由于停顿时间太长，一般手动配置为CMS

JVM若为client模式（根据硬件条件自动判断），则年轻代收集器默认为serial ，老年代默认为serial old

若为server模式，则年轻代收集器默认为parallel scavenge,老年代默认为parallel old

当老年代设置为CMS GC（减少GC执行时的停顿时间，垃圾回收线程和应用线程同时执行）方式时，年轻代默认为parNew.

#### 52.spring 事务处理中，同一个类中:A方法（无事务）调B方法（有事务）,事务会生效吗？

https://blog.csdn.net/liming19890713/article/details/79225894

解决方案：https://blog.csdn.net/dapinxiaohuo/article/details/52092447

不会生效。A如果没有受事务管理：  则线程内的connection 的 **autoCommit为true。**
B得到事务时事务**传播特性依然生效**，得到的还是A使用的connection，但是 **不会改变**autoCommit的属性。
所以B当中是按照每条sql进行提交的。

在一个Service内部，事务方法之间的嵌套调用，普通方法和事务方法之间的嵌套调用，都不会开启新的事务.**是因为spring采用动态代理机制来实现事务控制，**动态代理最终都是要调用原始对象的，而原始对象在去调用方法时，是不会再触发代理了！

因为：**Spring的事务传播策略在内部方法调用时将不起作用**，即目标对象内部的自我调用将无法实施切面中的增强，如图所示

![img](http://dl.iteye.com/upload/attachment/0066/6247/d33ee177-cf3f-39cc-befe-533306b0715f.jpg)

此处的this指向**目标对象**而非代理对象，，因此调用this.b()将不会执行b事务切面，即不会执行事务增强，因此b方法的事务定义“@Transactional(propagation = Propagation.REQUIRES_NEW)”将不会实施，即结果是b和a方法的事务定义是一样的。所以可以将this改为使用代理对象执行b方法

#### 53.Spring引入mybatis导致以及缓存失效

1.mybatis的一级缓存生效的范围是**sqlsession**，是为了在sqlsession没有关闭时，业务需要重复查询相同数据使用的。一旦sqlsession关闭，则由这个sqlsession缓存的数据将会被清空。

2.spring对mybatis的sqlsession的使用是由template控制的，sqlsession又被spring当作resource放在当前线程的上下文里（threadlocal),spring通过mybatis调用数据库的过程如下：

           a,我们需要访问数据
    
           b,spring检查到了这种需求，于是去申请一个mybatis的sqlsession（资源池），并将申请到的sqlsession与当前线程绑定，放入threadlocal里面
    
           c,template从threadlocal获取到sqlsession，去执行查询
    
           d,查询结束，清空threadlocal中与当前线程绑定的sqlsession，释放资源
    
           e,我们又需要访问数据
    
           f,返回到步骤b
**因为每次都进行创建，查询结束，会清空threadlocal中与当前线程绑定的sqlsession，所以就用不上sqlSession的缓存了.**

对于**开启了事务为什么可以用上**呢， 跟入getSqlSession方法

https://blog.csdn.net/ctwy291314/article/details/81938882

如下：

```
public static SqlSession getSqlSession(SqlSessionFactory sessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator) {
  notNull(sessionFactory, NO_SQL_SESSION_FACTORY_SPECIFIED);
  notNull(executorType, NO_EXECUTOR_TYPE_SPECIFIED);
  SqlSessionHolder holder = (SqlSessionHolder) TransactionSynchronizationManager.getResource(sessionFactory);
  // 首先从SqlSessionHolder里取出session
  SqlSession session = sessionHolder(executorType, holder);
  if (session != null) {
   return session;
  }
  if (LOGGER.isDebugEnabled()) {
   LOGGER.debug("Creating a new SqlSession");
  }
  session = sessionFactory.openSession(executorType);
  registerSessionHolder(sessionFactory, executorType, exceptionTranslator, session);
  return session;
 }
```

在里面维护了个**SqlSessionHolder**，关联了事务与session，如果存在则直接取出，否则则新建个session，所以在有事务的里，每个session都是同一个，故能用上缓存了



#### 54.进程间的通信方式

https://www.cnblogs.com/zgq0/p/8780893.html

**1.无名管道**：速度慢，容量有限，只有父子进程能通讯   。即先调用pipe(int fd[2])，开启读写，然后再fork()一个子进程，各自关闭一个读和写即可。

![img](https://images2015.cnblogs.com/blog/323808/201603/323808-20160311094030069-935122142.png)

**2.命名管道**（FIFO）：无关进程的交互。FIFO**有路径名与之相关联，它以一种特殊设备文件形式存在于文件系统中**。

```
// 返回值：成功返回0，出错返回-1
 int mkfifo(const char *pathname, mode_t mode);
```

**3.消息队列**：容量受到系统限制，且要注意第一次读的时候，要考虑上一次没有读完数据的问题    

```
#include <sys/msg.h>
2 // 创建或打开消息队列：成功返回队列ID，失败返回-1
3 int msgget(key_t key, int flag);
4 // 添加消息：成功返回0，失败返回-1
5 int msgsnd(int msqid, const void *ptr, size_t size, int flag);
6 // 读取消息：成功返回消息数据的长度，失败返回-1
7 int msgrcv(int msqid, void *ptr, size_t size, long type,int flag);
8 // 控制消息队列：成功返回0，失败返回-1
9 int msgctl(int msqid, int cmd, struct msqid_ds *buf);
```

**4.信号量**：不能传递复杂消息，只能用来同步    

```
 #include <sys/sem.h>
2 // 创建或获取一个信号量组：若成功返回信号量集ID，失败返回-1
3 int semget(key_t key, int num_sems, int sem_flags);
4 // 对信号量组进行操作，改变信号量的值：成功返回0，失败返回-1
5 int semop(int semid, struct sembuf semoparray[], size_t numops);  
6 // 控制信号量的相关信息
7 int semctl(int semid, int sem_num, int cmd, ...);
```

**5.共享内存区**：能够很容易控制容量，速度快，但要保持同步，比如一个进程在写的时候，另一个进程要注意读写的问题，相当于线程中的线程安全，当然，共享内存区同样可以用作线程间通讯，不过没这个必要，线程间本来就**已经共享了同一进程内的一块内存**

```
#include <sys/shm.h>
2 // 创建或获取一个共享内存：成功返回共享内存ID，失败返回-1
3 int shmget(key_t key, size_t size, int flag);
4 // 连接共享内存到当前进程的地址空间：成功返回指向共享内存的指针，失败返回-1
5 void *shmat(int shm_id, const void *addr, int flag);
6 // 断开与共享内存的连接：成功返回0，失败返回-1
7 int shmdt(void *addr); 
8 // 控制共享内存的相关信息：成功返回0，失败返回-1
9 int shmctl(int shm_id, int cmd, struct shmid_ds *buf);
```

#### 55.从synchronized 到CAS 和 AQS彻底弄懂Java各种并发锁（ReentrantReadWriteLock）

https://www.jianshu.com/p/8c1c047c9169

**ReentrantReadWriteLock**：读写锁，是根据volatile的state变量实现。高16位为读锁，低16位则为写锁。

同时，为了避免**写锁饥饿**，读写锁默认为非公平锁，即判断等待队列的第一个节点是否为请求写锁，若是，则阻塞当前请求线程。且**当前锁若为请求写锁，则会直接尝试获取锁，若为请求读锁，则要进行队头节点判断，从而实现写锁优先。**

其中，会记录firstReader和firstReaderHoldCount以及HoldCounter，主要是用来记录当前线程的重入数。因为有重入限制。**只有当线程获取共享锁后才能对共享锁进行释放、重入操作**。所以**HoldCounter**的作用就是当前线程持有共享锁的数量，这个数量必须要与线程绑定在一起，否则操作其他线程锁就会抛出异常。因此，其实通过ThreadLocal实现。

**注：**HoldCounter只保存了线程id和它对应的重入数。因此写了一个内部类继承了ThreadLocak<HoldCounter>，**重写了initialValue方法，从而实现将HoldCounter绑定到特定线程**。这样在释放锁的时候**才能知道ReadWriteLock里面缓存的上一个读取线程（cachedHoldCounter）是否是当前线程。**这样做的好处是可以**减少ThreadLocal.get()的次数，因为这也是一个耗时操作**。

同时,需要说明的是这样**HoldCounter绑定线程id而不绑定线程对象的原因是避免HoldCounter和ThreadLocal互相绑定而GC难以释放它们（**尽管GC能够智能的发现这种引用而回收它们，但是这需要一定的代价），所以其实这样做只是为了帮助GC快速回收对象而已。

https://www.cnblogs.com/xiaoxi/p/9140541.html

**为什么要将firstReader和firstReaderHoldCount呢？**

主要是效率问题，firstReader是不会放入到readHolds中的，如果**读锁仅有一个的情况下就会避免查找。readHolds**。

**为什么要缓存上一个释放读锁线程？**

里面有一个cachedHoldCounter缓存上过一个释放读锁的线程，可以**减少ThreadLocal.get()的次数，因为这也是一个耗时操作**。因为连续读锁释放很可能是同一个线程释放。例如多次重入就需要多次释放。

![img](https://images2018.cnblogs.com/blog/249993/201806/249993-20180607131704903-887096141.png)



#### 56.自定义类型可以作为HashMap的Key吗？

可以，但自定义类型要重写hashcode()和equals()方法，因为默认为Object的hashcode()方法，其实根据当前对象的物理地址生成的，因此任意两个对象的物理地址都不相等，则一直未false

#### 57.快速排序的优化

1.主要集中在**基准值**的选取上，可以随机选取基准值或者特定策略，分割为两个等长子数组效果最佳；

2.在递归分割数组长度较小时，变为插入排序更好

#### 58.java多线程的实现方式

启动一个线程的**唯一方式**就是调用Thread.start()方法。

1.继承Thread类，重写run()方法

2.当自定义类已继承其他类，则只能实现Runnable接口

3.实现Callable接口，通过FutureTask task=new FutureTask(new Callable());Thread thread = new Thread(task);**FutureTask<Integer>是一个包装器**，它通过接受Callable<Integer>来创建，FutureTask**同时实现了Future和Runnable接口**

4.**使用ExecutorService、Callable、Future实现有返回结果的线程**:

```
int taskSize = 5;  
   // 创建一个线程池  
   ExecutorService pool = Executors.newFixedThreadPool(taskSize);  
   // 创建多个有返回值的任务  
   List<Future> list = new ArrayList<Future>();  
   for (int i = 0; i < taskSize; i++) {  
    Callable c = new MyCallable(i + " ");  
    // 执行任务并获取Future对象  
    Future f = pool.submit(c);  
    // System.out.println(">>>" + f.get().toString());  
    list.add(f);  
   }  
```

####  59.Java线程与进程区别

1.进程是资源调度和分配的基本单位，而线程是任务执行（CPU调度和分配的基本单位）。

2.同一进程的多个线程共享全部资源，只拥有一些寄存器和PC和独立的栈空间。而进程拥有独立的内存单元。

#### 60.OS或java进程之间的通信方式

https://blog.csdn.net/b9x__/article/details/80300224

**1.管道**：管道类似于一种**特殊的文件**(并不是)，它存在于**内存**中，进程可以对它进行读写，它提供流控制，保证进程的正确读写，即管道为空时读进程会阻塞，管道为满时写进程会阻塞，以此实现进程之间的通信。

 管道有三种：1.普通管道（无名管道、也常直接称管道） 2.流管道 3.命名管道(FIFO)

**普通管道**：1）它是半双工的，即只能单向传输。2）它是有进程关系限制的，只能在父子进程之间使用。

**流管道**：相对于普通管道而言，它不止是单向传输，可以双向传输。

**命名管道(FIFO)：** 相对于普通管道而言，它没有进程关系限制，可以在无关进程之间进行数据交换。



**2.消息队列**：类似于用**链表**的结构存储消息。相比于管道，**不止只能传输字节流**，也没有**缓冲区大小的限制**。实现了消息的**随机读取**。



**3.套接字（Socket）**:**套接字可用于不同机器间的进程通信，即可用于网络之间的进程通信。**



**4.信号量**:**信号量用于实现进程间的互斥与同步，而不是用于存储进程间通信数据。****它是一个计数器，用来控制多个进程对共享资源的访问，常作为一种锁机制，实现进程间的同步和互斥。(JUC的Semaphore的设计思想来源吧)**



**5.共享内存：**多个进程共享某块内存，**共享内存是通信方式中最快的一种**。操作系统建立一块共享内存，并将其映射到参与通信的每个进程的地址空间上，进程就可以直接对这块共享内存进行读写。**适合非常庞大的、读写操作频率很高的数据（配合信号量使用），常见于多进程之间。**

**共享内存这种方式为什么是最快的呢？**

        这是因为共享内存的整个通信过程对消息的复制只有两次。
    
                1.从数据来源复制到共享内存 
    
                2.从共享内存复制到数据目的地
    
          而管道、消息队列等方式对消息的复制需要四次，因为有缓冲区的存在，读写都要经过缓冲区。
6**.文件**，共同操作一个文件

7.**Signal**:一个进程给另一个进程发的信号，例如kill命令，就是发送一个中断信号

#### 61.java线程栈空间大小

线程栈的默认大小：根据JVM和OS来确定，linux 32为JVM为320K,64位为1MB。最小值为64K

#### 62.java内存模型

https://www.cnblogs.com/chihirotan/p/6486436.html

![img](https://images2015.cnblogs.com/blog/724399/201703/724399-20170302101629751-1617442155.png)



**注意与JVM内存模型的区分**



#### 63.Java线程和操作系统线程的关系

https://blog.csdn.net/cringkong/article/details/79994511

也就说JDK1.2之前，**程序员们为JVM开发了自己的一个线程调度内核，而到操作系统层面就是用户空间内的线程实现。**而到了JDK1.2及以后，JVM选择了更加稳健且方便使用的操作系统原生的线程模型，**通过系统调用，将程序的线程交给了操作系统内核进行调度**

对于Sun JDK来说，它的Windows版与Linux版都是使用一对一的线程模型实现的**，一条Java线程就映射到一条轻量级进程**之中，因为Windows和[Linux系统](https://www.baidu.com/s?wd=Linux%E7%B3%BB%E7%BB%9F&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)提供的线程模型就是一对一的。



**而对不同的操作系统，由于本身设计思路不一样，对于线程的设计也存在种种差异，所以JVM在设计上，就已经声明：**

```
虚拟机中的线程状态，不反应任何操作系统线程状态
```

#### 64.**java关键字Transient**

​    Java的serialization提供了一种持久化对象实例的机制。当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。       **transient是Java语言的关键字，用来表示一个域不是该对象串行化的一部分**。当一个对象被串行化的时候，transient型变量的值不包括在串行化的表示中，然而非transient型的变量是被包括进去的。  注意static变量也是可以串行化的**。例如非线程安全的集合，用于实现**fail-fast机制的变量modCount**就是使用transient修饰，因为没有必要持久化，只用于在内存中使用和判断。

#### 65.TopK问题

https://blog.csdn.net/z50L2O08e2u4afToR9A/article/details/82837278

TopK，不难；其思路优化过程，不简单：

**全局排序**，O(n*lg(n))

**局部冒泡排序**，只排序TopK个数，O(n*k)，即冒泡排序，冒k个

**堆**，TopK个数也不排序了，O(n*lg(k))

**分治法**，每个分支“都要”递归，例如：快速排序，O(n*lg(n))

**减治法**，“只要”递归一个分支，例如：二分查找O(lg(n))，随机选择O(n)

T**opK的另一个解法：随机选择+partition**



#### 66.多线程处理Socket问题

- **对于UDP**，多个线程读写一个Socket不用加锁，当然最好的做法是每个线程有自己的Socket。因为其实面向报文发送的，**协议栈是以报文整个的形式来传递，所以是原子操作**
- **对于TCP**，多个线程处理一个Socket是错误的设计。通常多线程读写同一个 socket 是错误的设计，因为有 short write 的可能。假如你加锁，而又发生 **short write（短写）**，你是不是要一直等到整条消息发送完才解锁（无论阻塞IO还是非阻塞IO）？如果这样，你的临界区长度由对方什么时候接收数据来决定，一个慢的 peer 就把你的程序搞死了。

- 总结：对于UDP，加锁是多余的，对于TCP，加锁是错误的。因为Socket的底层已经使用了synchronized实现同步

#### 67.NIOI/O复用模型

linux提供select/poll，进程通过将一个或多个fd传递给select或poll系统调用，阻塞在select;这样select/poll可以帮我们侦测许多fd是否就绪。**但是select/poll是顺序扫描fd是否就绪，而且支持的fd数量有限**。linux还提供了一个epoll系统调用，**epoll是基于事件驱动方式，而不是顺序扫描,当有fd就绪时，立即回调函数rollback；**

**AIO，BIO，NIO**
**AIO异步非阻塞IO**，AIO方式适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

**NIO同步非阻塞IO**，适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。

**BIO同步阻塞IO**，适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。

**Java对BIO、NIO、AIO的支持：**
**Java BIO** ： 同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。
**Java NIO** ： 同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。（底层是epoll）

**Java AIO(NIO.2)** ： 异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理。



#### 68.分布式锁实现

https://www.cnblogs.com/seesun2012/p/9214653.html

https://www.cnblogs.com/austinspark-jessylu/p/8043726.html

1.**基于数据库乐观锁**：1）基于表主键唯一性：数据库能保证只有一个操作能成功，即将方法名字段设为唯一值，插入成功则记录当前信息（**方法名，线程信息**），释放锁即删除这条数据。但太依赖数据库可用性，没有失效时间，且不可重入2）基于mysql的MVCC表字段版本号机制 3）基于数据库排它锁，即每条语句都加上for update，通过`connection.commit()`操作来释放锁，但会有单点和重入问题

2.**基于Redis**：1)setnx()和expire 2)**基于 REDLOCK 做分布式锁**。其中有个**死锁**问题，若设置超时时间时tomcat挂掉，则会导致死锁。**解决方法**：将**setnx的value设为currenttime+timeout**，从而实现双重保险。即有另一个线程竞争锁获取失败时，会检查get(key)获得value，比较当前时间是否大于value，如果大于，则出现死锁，即使用getset()方法重新设置。

3.**基于Zookeeper**：在目录树中创建临时有序节点，并自增，每次选择最小节点。当前节点总是监听前一个节点的变化（Watch)，节点记录了主机和线程信息，解决了重入问题，单点问题（集群部署），非阻塞（绑定监听器并自动通知客户端）以及锁无法释放（zk会自动删除挂掉的客户端）。性能不如Redis



#### 69.Mysql的Next Lock锁

默认情况下，InnoDB工作在Repeatable Read隔离级别下（隔离级别详细介绍），对主键索引、唯一索引以记录锁方式对数据进行加锁，对普通索引以Next-Key Lock的方式对数据行进行加锁。如果一个间隙被事务加了锁，其它事务是不能在这个间隙插入记录的，这样可以有效防止幻读的发生。

**对于快照读来说，即普通select,幻读的解决是依赖mvcc解决。而对于当前读则依赖于gap-lock解决。**

**快照读**：简单的select操作(不包括 select ... lock in share mode, select ... for update)

**当前读**：select ... lock in share mode

select ... for update

**当for update的字段为索引或者主键的时候，只会锁住索引或者主键对应的行。**

**而当for update的字段为普通字段的时候，Innodb会锁住整张表。**

insert

update

delete

**行锁(Record Lock)：**也叫记录锁，锁直接加在索引记录上面。

**间隙锁(Gap Lock)：**锁加在不存在的空闲空间，可以是两个索引记录之间，也可能是第一个索引记录之前或最后一个索引之后的空间。

**Next-Key Lock**：行锁与间隙锁组合起来用就叫做Next-Key Lock。

**总结：**在mysql中，提供了两种事务隔离技术，第一个是**mvcc**，第二个是**next-key**技术。这个在使用不同的语句的时候可以动态选择。不加lock inshare mode之类的就使用mvcc。否则使用next-key。mvcc的优势是**不加锁，并发性高**。缺点是**不是实时数据**。next-key的优势是获取实时数据，但是需要加锁。同时需要注意几点：1.事务的快照时间点是以第一个select来确认的。所以即便事务先开始。但是select在后面的事务的update之类的语句后进行，那么它是可以获取后面的事务的对应的数据。2.mysql中数据的存放还是会通过版本记录一系列的历史数据，这样，可以根据版本查找数据。



#### 70.建立全文索引实现

工作原理：

1、索引程序从数据库读取数据，比如上面例子中的数据表，索引程序通过sql语句：select 文章id,文章标题，文章内容 from 文章表.获得文章的相关数据

2、索引程序对需要索引的内容进行“分词”，而这里的分词就是调用分词程序啦！

3、索引程序对分好词的一个个词条加入索引文件。



#### 71.解决CAS的ABA问题

从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查**当前引用是否等于预期引用**，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

#### 72.死锁必须同时满足的条件

1、**互斥等待**，即必须有锁

2、**hold and wait**，即拿着一个锁还在等另一个锁

3、**循环等待**，即A对象拿了A锁等B锁，而B对象拿了B锁等A锁

4、**无法剥夺的等待**，即没有超时自动放弃锁这一说（synchronized关键字就是会无限等待，没有超时自动放弃锁）

  

#### 73.文件断点下载和上传原理

https://www.cnblogs.com/wangzehuaw/p/5610851.html

断点续传和断点下载都是用的**RandomAccessFile,** 它具有移动指定的文件大小的位置的功能seek 。

**断点续传**是由服务器给客户端一个已经上传的位置标记position，然后客户端再将文件指针移动到相应的position，通过输入流将文件剩余部分读出来传输给服务器

断点下载 是由客户端告诉服务器已经下载的大小，然后服务器会将指针移动到相应的position，继续读出，把文件返回给客户端。 当然为了下载的更快一下，也可以多线程下载，那么基本实现就是给每个线程分配固定的字节的文件，分别去读



#### 74.常见海量数据面试题

<https://blog.csdn.net/LLZK_/article/details/53106697>

#### 75.Redis如何保证和MySQL数据一致？实时同步

**1.主动更新**

MySQL binlog增量订阅消费+消息队列+处理并把数据更新到redis,即使用canal伪装为mysql的slave节点获得binlog日志。

![Enosq17N* :  Mysql  Nosql  memcached...)  im Slave  Canal Server  (RhSlaveäÅRbinlog)  Canal Client  jsonEjtbinlog  rabbitmqN  redis) ](file:///C:/Users/ASUS/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)

**2.超时删除：**在设置缓存的时候可以设置**过期时间**，在时间到期之后自动删除。关于key过期删除，最好的方式是使用定时删除，这样可以最快的释放被占用的内存，但很明显**大量的定时器**对CPU来说是非常不友好的。所以需要采用惰性删除、在获取key的时检查是否过期，过期直接删除。**惰性删除**虽然性能最好，但对于冷数据来说还是没解决**缓存污染**的问题，所以还需增加个定期清理和惰性删除配合使用。比如单开个线程每5分钟去遍历检查key是否过期，这个时间策略是可配置的，如果缓存数量较多可分批遍历检查。惰性删除配合定期删除基本上能满足绝大多

**3.LRU策略**：用**链表**结构，即新数据插入到链表头部、被命中时的数据移动到头部，把最少访问的数据给淘汰掉，经常被访问到即是热点数据。添加复杂度O(1)，移动和获取复杂度O(N)。大多数情况下，LRU算法对热点数据命中率是很高的。 但如果**突然大量偶发性的数据访问，会让内存中存放大量冷数据，也即是缓存污染**。因此要配置**过期删除**策略。

**基于 HashMap 和 双向链表实现 LRU 的** 相当于java中的linkedHashMap

如何设计一个LRU缓存，使得**放入和移除都是 O(1)** 的，我们需要把访问次序维护起来，但是不能通过内存中的真实排序来反应，有一种方案就是使用**双向链表**。整体的设计思路是，可以使用 **HashMap 存储 key**，这样可以做到 save 和 get key的时间都是 O(1)，而 HashMap 的 Value 指向双向链表实现的 LRU 的 Node 节点

**Redis的LRU实现**

如果按照HashMap和双向链表实现，需要额外的存储存放 next 和 prev 指针，牺牲比较大的存储空间，显然是不划算的。所以Redis采用了一个近似的做法，就是**随机取出若干个key**，然后**按照访问时间排序后，淘汰掉最不经常使用的，**





#### 76.redis并发读写锁，使用Redisson实现分布式锁

https://blog.csdn.net/l1028386804/article/details/73523810

使用Redisson的可重入锁，来实现分布式锁。解决**多个Tomcat同时调用一个Redis**会线程冲突的问题。尽量每一个线程新建一个redis实例，因为redis本身是**单线程**的，能保证安全。如果共用1个连接，那么返回的结果无法保证被哪个进程处理。持有连接的进程理论上都可以对这个连接进行读写，这样数据就发生错乱了



#### 77.HTML页面缓存相关

***CDN***的全称是**Content Delivery Network**，即内容分发网络

相应头中会有**expire**字段，用于表明过期时间。同时也有一个**last-modified**。如果超过last-modified时间，则浏览器会发起请求，服务器检查是否修改，若没有修改，则只返回**一个304响应头**

**缓存刷新方式的不同：**

**1.重新输入url回车**：浏览器以最少的请求来获取网页的数据，浏览器会对所有没有过期的内容直接使用本地缓存，从而减少了对浏览器的请求。所以，Expires，max-age标记只对这种方式有效。

**2.F5或者浏览器刷新按钮**：浏览器会在请求中附加必要的缓存协商，但不允许浏览器直接使用本地缓存，它能够让 Last-Modified、ETag发挥效果，但是对Expires无效

**3.ctrl+f5：**这种方式就是强制刷新，总会发起一个全新的请求，不使用任何缓存。



#### 78.java序列化过程

![æè·¯å¾](https://img-blog.csdn.net/20180531172434883?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTcyMzU0NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 79.Netty的ByteBuf缓冲区的种类

ByteBuf支持**堆缓冲区**和**堆外直接缓冲区**，根据经验来说，底层IO处理线程的缓冲区使用堆外直接缓冲区，减少一次IO复制。业务消息的编解码使用堆缓冲区，分配效率更高，而且不涉及到内核缓冲区的复制问题。

ByteBuf的**堆缓冲区**又分为**内存池缓冲区PooledByteBuf**和**普通内存缓冲区UnpooledHeapByteBuf**。PooledByteBuf采用二叉树来实现一个内存池，集中管理内存的分配和释放，不用每次使用都新建一个缓冲区对象。UnpooledHeapByteBuf每次都会新建一个缓冲区对象。在高并发的情况下推荐使用PooledByteBuf，可以节约内存的分配。在性能能够保证的情况下，可以使用UnpooledHeapByteBuf，实现比较简单。



#### 80.java序列化和反序列化

https://blog.csdn.net/xlgen157387/article/details/79840134

**序列化：**

（1）将对象实例相关的类元数据输出。 
（2）递归地输出类的超类描述直到不再有超类。 
（3）类元数据完了以后，开始从最顶层的超类开始输出对象实例的实际数据值。 
（4）从上至下递归输出实例的数据



#### 81.java值传递问题

java都是**值传递**，因此当传递int等基本数据类型或者不可变类型（String），在方法调用时并不会改变本身的值，因为在java中是传值，若为基本数据类型则只是一个**值拷贝**，若是对象类型，则传递了一个**引用拷贝**，因此可以修改值。



#### 82.java内存泄漏的可能情况

**1、静态集合类引起内存泄漏**

　　像HashMap、Vector等的使用最容易出现内存泄露，这些静态变量的生命周期和应用程序一致，他们所引用的所有的对象Object也不能被释放，因为他们也将一直被Vector等引用着。

**2、当集合里面的对象属性被修改后，再调用remove()方法时不起作用。**例如自定义类忘记重写hashcode()和equals方法，导致集合调用remove()方法将属性修改前后的对象当做两个对象。

**3、监听器**

　　在释放对象的时候却没有去删除这些监听器，增加了内存泄漏的机会。

**4、各种连接**

　　比如数据库连接（dataSourse.getConnection()），网络连接(socket)和io连接，除非其显式的调用了其close（）方法将其连接关闭，否则是不会自动被GC 回收的。

**5、内部类和外部模块的引用**

　　内部类的引用是比较容易遗忘的一种，而且一旦没释放可能导致一系列的后继类对象没有释放。此外程序员还要小心外部模块不经意的引用，例如程序员A 负责A 模块，调用了B 模块的一个方法如： public void registerMsg(Object b); 这种调用就要非常小心了，传入了一个对象，很可能模块B就保持了对该对象的引用，这时候就需要注意模块B 是否提供相应的操作去除引用。

**6、单例模式**

　　不正确使用单例模式是引起内存泄漏的一个常见问题，单例对象在初始化后将在JVM的整个生命周期中存在（以静态变量的方式），如果单例对象持有外部的引用，那么这个对象将不能被JVM正常回收，导致内存泄漏。



#### 83.保证java线程的顺序性

1.thread.join()

2.线程池ExcutorService的newSingleThreadExecutor()

3.lock+volotile变量，即先获取锁，再判断变量值，若等于特定值，则继续执行，执行完修改为另一个值

4.zk临时节点

5.synchronize（thread1）{thread1.start} ;synchronize（thread2）{thread2.start} 

6.线程内接收多个countdownlatch

7.blockingqueue存放多个线程，接口按序取线程，再调用thread.start();

#### 84.java线程的通信方式

1.同步，Condition的await()/signal()以及synchronize

2.while()轮询一个volotile修饰的变量

3.wait/notify

4.管道通信 PipedInputStream和PipedOutputStream

5.ThreadLocal

6.BlockingQueue

7.AtomicInteger

#### 85.Spring的AOP实现原理

AOP 代理主要分为**静态代理**和**动态代理**两大类，静态代理以 AspectJ 为代表；而动态代理则以 **Spring AOP** 为代表，其中Spring AOP分为**jdk动态代理和CGLib动态代理**

**静态代理**是指使用 AOP 框架提供的命令进行编译，从而在**编译阶段就可生成 AOP 代理类**，因此也称为**编译时增强**；而动态代理则在运行时借助于 **JDK 动态代理、CGLIB** 等在内存中“临时”生成 AOP 动态代理类，因此也被称为**运行时增强。**

**JDK动态代理与CGLib动态代理的区别**

JDK动态代理是面向接口，在创建代理实现类时比CGLib要快，创建代理速度快。

CGLib动态代理是通过**字节码底层继承要代理类**来实现（如果被代理类被final关键字所修饰，那么抱歉会失败），在创建代理这一块没有JDK动态代理快，但是运行速度比JDK动态代理要快。**ajc.exe** 当成一个增强版的 javac.exe 命令，用于编译指定的aspectJ程序，生成了一个整整的Hello.class文件，但并不是Hello.java得到，而是继承编译出的一个新类。



**Spring AOP与AspectJ的区别**

Spring  aop依然采用运行时生成**动态代理**的方式来增强目标对象，所以它不需要增加额外的编译，也不需要 AspectJ 的织入器支持,但借用了**@aspectJ的注解**（注：只是使用了aspectJ注解）支持，因此跟aspectJ使用一致；而 AspectJ 在采用编译时增强，所以 AspectJ 需要**使用自己的编译器来编译 Java 文件，还需要织入器。**



#### 86.Spring的事务传播性

https://blog.csdn.net/zcl_love_wx/article/details/80275087

**总结：**

spring的事务传播属性**不适用于同类中的方法调用**，即**事务方法在被同类中的方法调用时会被当作非事务方法**。

**事务方法调用非事务方法（包括本类中事务不生效的方法）时，事务方法捕获了被调用方法里的异常时就都不会回滚，不捕获就都回滚。**

非事务方法调用非事务方法（包括本类中事务不生效的方法）和调用嵌套事务时一样都不会互不影响，即都会自动提交。

事务方法多级嵌套调用非事务方法，和调用一级非事务方法结果是一致的，即只要事务方法捕获了异常就都不回滚，不捕获就都回滚。

一个方法b加入到事务方法a中时，不论b抛出的异常是否被a捕获，结果是整个事务都会回滚。因为始终是一个事务。

事务方法a调用一个新开的事务方法b，在b的最后抛出异常，a捕获异常。结果是只有b回滚。因为都不在同一个事务当中了。

#### 87.事务在数据库中本质

数据库执行事务的时候，是**先将数据插入到日志中**，如果没有遇到回滚，则在**提交事务的时候将日志操作同步到数据库。如果回滚的话，则日志的操作不再插入数据库中**。

如果发生回滚，则主键还是会增大的即**主键会变得不连续**。例如，本应该插入的数据id为100，但是发生了回滚，则后面再正确插入的数据的主键会是101。

JDBC对事务的支持是放在Connection连接中的。



#### 88.自定义类加载器与双亲委派模型的破坏

**1.重写findclass()方法：**new一个自定义类加载器后调用loadclass()方法，系统内找不到则会调用findclass()方法。findClass()方法中调用defineClass()用于将字节数组转化为Class对象。

**2.重写loadClass()方法：**该方法不建议，因为会破坏双亲委派模型，但依然能保证核心api类不被篡改。因为使用需要使用defineClass()方法用于将字节数组转化为Class对象，该方法为final方法，不可重写，因此始终会检查是否为java.*开头的全限定名类，不然就抛出异常。

**java中破坏双亲委派模型的情况：**

**线程上下文加载器（contextClassLoader）**例如jdbc.jar中的Driver类时，没有手动调用Class.forName()反射加载注册驱动类，而是在**DriverManager.getConnection()**中进行加载。由于加载操作的关键**serviceLoader**是在BootClassLoader中，因此就是使用**contextClassLoader**进行加载，从而实现核心代码去调用外部的SPI代码，也就是**父类加载器请求子类加载器去加载**。

![img](https://upload-images.jianshu.io/upload_images/2154124-d5859f8e79069128?imageMogr2/auto-orient/strip%7CimageView2/2)



#### 89.查询时间字段，如距离当前时间大于几天的数据

datediff(day,[date],now())>7,即查询大于7天的数据

#### 90.J.U.C体系结构图

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160330214458630)

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160330214841804)

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160330215547041)

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160330220653889)

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160330221748909)

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160330222608552)



#### 91.ArrayBlockingQueue和LinkedBlockingQueue的区别

LinkedBlockingQueue用了两个ReentrantLock，即putLock和takeLock。因为入队操作时加载队尾，即last节点，而出队只针对head，因此不用公用一把锁，从而支持多线程同时入队和出队操作。此外，为了避免count线程不安全，因此使用了AtomicInteger类型。



#### 92.银行家算法

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20180508204335770?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDE0Mjcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

假设资源P1申请资源，银行家算法先**试探的分配给它**（当然先要看看当前资源池中的资源数量够不够），**若申请的资源数量小于等于Available，然后接着判断分配给P1后剩余的资源，能不能使进程队列的某个进程执行完毕，若没有进程可执行完毕，则系统处于不安全状态**（即此时没有一个进程能够完成并释放资源，随时间推移，系统终将处于死锁状态）。

若**有进程可执行完毕**，则假设回收已分配给它的资源（剩余资源数量增加），**把这个进程标记为可完成，并继续判断队列中的其它进程，若所有进程都可执行完毕，则系统处于安全状态，并根据可完成进程的分配顺序生成安全序列**（如{P0，P3，P2，P1}表示将申请后的剩余资源Work先分配给P0–>回收（Work+已分配给P0的A0=Work）–>分配给P3–>回收（Work+A3=Work）–>分配给P2–>······满足所有进程）。



#### 93.排查服务器高负载，响应慢

1.top命令查看资源消耗情况

2.df -l 查看磁盘使用率

3.free 查看内存使用率

4.查看qps流量情况

5.查看JVM GC情况：1）在启动参数进行配置  2）Jconsole命令可视化线程信息  3）Jmap -heap [pid]查看某个java应用进程号对应的堆占用信息

6.ps -ef|grep tomcat 查看tomcat的进程ID

7.ps -aux|grep [pid] 查看该进程号的启动信息，如是哪个用户启动该进程

8.su xxxx切换到tomcat启动用户（不然有可能导致无法查看栈信息）

9.jstack -l [pid] 导出到文件

10.top -H -p [pid] 查看该进程号的所有线程占用情况

11.将线程号转换为十六进制 printf "%x\n" [tid]

12.在导出的栈文件中查看对应信息  cat xxx.stack | grep 0xxxxx



#### 94.Union和Union  all的区别

union即两个查询结果集，但结果**集的列类型和数量和顺序**必须一致（不要求列名一致），union all即为不去重的union



#### 95.Tomcat的组成和加载机制

https://www.jianshu.com/p/c94dc6c64ec5

![img](https://upload-images.jianshu.io/upload_images/10649427-dcee5fb533394829.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

![img](https://upload-images.jianshu.io/upload_images/10649427-b26366b477d0ccae.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

![1552458886851](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1552458886851.png)

**server:**

**engine:**对应多个host，即域名前缀（localhost部分）

**host**:对应多个context

**context:**对应一个web应用

**service:**对应一个engine和多个connector

**connector:**一个Connector将在某个指定端口上侦听客户请求，并将获得的请求交给Engine来处理，从Engine处获得回应并返回客户
TOMCAT有两个典型的Connector，**一个直接侦听来自browser的http请求，一个侦听来自其它WebServer的请求**
Coyote Http/1.1 Connector 在端口8080处侦听来自客户browser的http请求
Coyote JK2 Connector 在端口8009处侦听来自其它WebServer(Apache)的servlet/jsp代理请求

**tomcat类加载器加载顺序**

boostrap类加载器

system类加载器

common类加载器（容器内公有部分,所有web应用依赖一个容器，故可以共用）

catalina类加载（容器内私有，对webapp不可见）和shared类加载器（webapp共享，对于**上层tomcat容器不可见**）

webapp类加载器（webapp私有）

**tomcat 为了webapp之间的实现隔离性**，没有遵守这个约定，每个**webappClassLoader**加载自己的目录下的class文件，不会传递给父类加载器。因此，**如果tomcat 的 Common ClassLoader 想加载 WebApp ClassLoader 中的类,可以使用线程上下文加载器。**



![1552459340736](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1552459340736.png)



#### 96.TCP和IP报文是否会分片，分别在什么情况下会分片。 TCP分包之后是否还会进行ip分片?

IP数据报的分片与重组是在**网络层**进完成的,TCP报文段的分段与重组是在**运输层**完成的。

**TCP分段的原因是MSS**（最大分段大小），IP分片的原因是MTU(最大分片大小，限制数据帧的)，由于一直有**MSS<=MTU**，很明显，分段后的每一段**TCP报文段再加上IP首部后的长度不可能超过MTU**，因此也就**不需要在网络层进行IP分片了**。因此TCP报文段很少会发生IP分片的情况。

UDP数据报，**由于UDP数据报不会自己进行分段，因此当长度超过了MTU时，会在网络层进行IP分片**。同样，ICMP（在网络层中）同样会出现IP分片情况。



#### 97.多态在jvm是如何实现的？

实现多态，即动态绑定。需要有继承和重写方法。例如父子类中共有的方法，Father father = new Son()，当调用father.test()共有方法时，是根据father引用指向的堆中的实际对象，该实际对象指向方法区中的Son类的方法表，即为特殊类型指针，从而找到son的test()方法，而非father的test()方法。



#### 98.Spring的ApplicationContext和BeanFactory初始化的过程区别

https://www.jianshu.com/p/3944792a5fff



**ApplicationContext**容器启动时会对**所有scope为singleton且为非懒加载的bean进行实例化**，而**BeanFactory**则**不会去实例化所有Bean**，包括为singleton的非懒加载的bean，而是**在调用时实例化**



**ApplicationContext作为容器的bean生命周期**



![img](E:\Pandoc笔记\webp)

**BeanFactory作为容器的bean生命周期**

![img](https://upload-images.jianshu.io/upload_images/3131012-249748bc2b49e857.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/746/format/webp)



#### 99.序列化和反序列化

核心就是ObjectOutputStream()和oos.writeObject(object)和ObjectInputStream(new ByteArrayInputStream(bytes))和ois.readObject()。

```java
public class SerializeUtil {
public static byte[] serialize(Object object) {
ObjectOutputStream oos = null;
ByteArrayOutputStream baos = null;

try {
//序列化
baos = new ByteArrayOutputStream();
oos = new ObjectOutputStream(baos);
oos.writeObject(object);
byte[] bytes = baos.toByteArray();

return bytes;

} catch (Exception e) {
}

return null;

}


public static Object unserialize(byte[] bytes) {
ByteArrayInputStream bais = null;

try {

//反序列化
bais = new ByteArrayInputStream(bytes);
ObjectInputStream ois = new ObjectInputStream(bais);
return ois.readObject();
} catch (Exception e) {

}

return null;

}

}
```

如果需要**自定义序列化**方式，则需要自定义实现一个序列化解析类（***Util.serialize()），然后传入特定对象，调用重写的writeObject()和readObject()方法。从而实现序列化本不能序列化的属性cityAndState对象。当多个类都需要时，可以将**不可序列化的对象分离出来写一个新的装饰器对象**，将原CityAndState对象传入，然后重写writeObject和readObject()方法。

**阿里巴巴FastJson原理**

![1555337396274](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1555337396274.png)

<https://blog.csdn.net/CSDN_LQR/article/details/51464338>

   **private void writeObject(final ObjectOutputStream out) throws IOException**  
   {  
​      out.writeUTF(this.lastName);  
​      out.writeUTF(this.firstName);  
​      out.writeUTF(this.cityAndState.getCityName());  
​      out.writeUTF(this.cityAndState.getStateName());  
   }  
   **private void readObject(final ObjectInputStream in) throws IOException, ClassNotFoundException**  
   {  
​      this.lastName = in.readUTF();  
​      this.firstName = in.readUTF();  
​      this.cityAndState = new CityState(in.readUTF(), in.readUTF());  
   }  

#### 100.输入网址的整个过程

https://www.cnblogs.com/kongxy/p/4615226.html

分为**网络通信**和**页面渲染**两部分

#### 101.Spring的IOC源码相关

Resource -> BeanDefinition（xml中bean的配置信息） -> BeanWrapper（**类型转换以及setter参数调用和实例化**） -> Object（beanFactory.getBean()方法获得）

在 Spring 进行 依赖注入的时候，首先把这种资源转化成 **Resource 抽象**，通过里面的 IO 流读取定义的 bean。然后再转化成 **BeanDefinitioin**，里面定义了包括构造器注入，以及 setter 注入的定义。最后通过 **BeanWrapper** 这个接口，首先获取定义的构造器注入属性，通过反射中的 Constructor 来创建对象。基于这个对象，通过 java 里面的内省机制获取到定义属性的属性描述器(PropertyDescriptor)，调用属性的写入方法完成依赖注入，最后再调用 Spring 的自定义初始化逻辑，主要包括以下三个扩展点：

- BeanPostProcess，Spring aop 就是基于此扩展。
- Init-method，可以在 `bean` 标签通过 init-method 定义，也可以实现 InitializingBean
- XXXAware，**Spring 容器感知类**，可以在 bean 里面获取到 Spring 容器的内部属性。



#### 102.Tomcat处理Http请求的内部处理流程

假设来自客户的请求为：
http://localhost:8080/wsota/wsota_index.jsp

1) 请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 **Connector**获得
2) Connector把该请求交给它所在的**Service**的**Engine**来处理，并等待来自Engine的回应
3) Engine获得请求localhost/wsota/wsota_index.jsp，匹配它所拥有的所有虚拟主机**Host**
4) Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机）
5) localhost Host获得请求/wsota/wsota_index.jsp，匹配它所拥有的所有**Context**
6) Host匹配到路径为/wsota的Context（如果匹配不到就把该请求交给路径名为""的Context去处理）
7) path="/wsota"的Context获得请求/wsota_index.jsp，在它的mapping table中寻找对应的**servlet**
8) Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类
9) 构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法
10)Context把执行完了之后的HttpServletResponse对象返回给**Host**
11)Host把HttpServletResponse对象返回给**Engine**
12)Engine把HttpServletResponse对象返回给**Connector**
13)Connector把HttpServletResponse对象返回给客户**browser**



#### 103.为什么MappedByteBuffer比较快

因为使用了虚拟内存映射，get()方法底层就是调用了directByteBuffer.get()。实质上跟directByteBuffer有区别，后者是直接读入内存，前者是利用**虚拟内存**，实质上是在硬盘上，**并没有拷贝数据到内存中去**，**而是当进程代码第一次引用这段代码内的虚拟地址时，触发了缺页异常，这时候OS根据映射关系直接将文件的相关部分数据拷贝到进程的用户私有空间中去**

从代码层面上看，从硬盘上将文件读入内存，都要经过文件系统进行数据拷贝，并且数据拷贝操作是由文件系统和硬件驱动实现的，理论上来说，拷贝数据的效率是一样的。
 但是通过内存映射的方法访问硬盘上的文件，效率要比read和write系统调用高，这是为什么？

- read()是系统调用，首先将文件从硬盘**拷贝到内核空间的一个缓冲区**，再将这些数据拷贝到**用户空间**，实际上进行了两次数据拷贝；
- map()也是系统调用，但没有进行数据拷贝，当缺页中断发生时，直接将文件从硬盘拷贝到用户空间，只进行了一次数据拷贝。

所以，采用内存映射的读写效率要比传统的read/write性能高。



因为是映射虚拟内存，因此不受JVM的参数限制。同时，在处理大文件时可实现特定位置读写。其中，**RandomAccessFile随机访问见是没有实现内存直接读取的，也是两次数据拷贝。**

链接：https://www.jianshu.com/p/f90866dcbffc



#### 104.java函数调用值传递问题

`Java`中函数调用时不会改变基本类型参数的值的。

用 `Integer`代替也不行，因为 `Integer`和 `String`一样，都是不可变对象，所有的值更改操作在底层都是拆箱和装箱生成新的 `Integer`



#### 105.java中Integer.parseInt()和Integer.valueOf()区别

Integer.parseInt()返回一个int对象，而Integer.valueOf()返回一个Integer对象，除此之外，还会根据得到的值生成-128-127的缓存，所以尽量用parseInt()

#### 106.海量数据的敏感词查询

考虑使用字典树，即根据敏感词进行分字建立字典树，除了根节点，每一个节点都按序存放一个字符。

**字典树可用于**：1.:关键词快速加锁，2.字典序排序，3.两个字符串最长公共前缀 4.搜索引擎热词提示

#### 107.UDP和TCP的最大包大小

最大包大小主要受限于**MTU**(网络层的最大传输单元)，一般大于**1500字节**则会被分片。因此

考虑到I**P报文头长度固定20字节，TCP头20字节，UDP头8字节**，因此TCP最大为1460字节，UDP为1472字节。

同时，考虑到网络的路由转发的MTU设置不同可能会有变化。



#### 108.Tomcat集群的同步

tomcat同步主要是session同步。tomcat实现会话同步的过程中大致会使用如下组件，现在假设中间的tomcat实例的会话改变了，它会通过**会话管理器Manager**将改变的动作消息封装成消息然后调用**集群对象Cluster**，通过Cluster将消息发送出去，同时Cluster又依赖于tribes，最后消**息其实是交由tribes**真正发送的，通信过程是以**ClusterMessage为对象**传输的，它会**先被序列化进行传输，到达左边和右边的tomcat实例时会被反序列化**，消息由tribes接收后往Cluster上传，最后到达会话管理器Manager，Manager根据动作消息去同步会话。

所以Cluster其实就是实现了ChannelListener的监听类，当tribes接收到消息后就会调用此监听器的messageReceived方法处理逻辑，此方法又会继续往上通知Manager的messageDataReceived方法，此方法内完成会话同步处理逻辑。关于会话具体的同步机制tomcat提供了两种，分别是**“集群增量会话管理器——DeltaManager**”和“**集群备份会话管理器——BackupManager**”。

**集群增量会话管理器：**a.这是一种**全节点复制模式**，即其中有一个节点发生变化后，将同步到其他节点。b.其次，该复制模式还有一个特点就是**只同步会话增量**的特点，增量是以一个会话为周期，即在一个请求被响应之前同步到各个节点上

**集群备份会话管理器**：上面的集群增量会话管理器有一个缺点：当集群节点数增大时，会导致网络通信量急剧增加。备份会话管理器则是**每个会话只有一个备份**。



#### 109.超卖问题的方案

1.将库存放入Redis中，单线程保证安全性。同时，根据key值变化发送到消息队列，然后一步写入本地DB。

2.所有写DB的操作都放入单个队列中排队，然后DB按序取出消费。

#### 110.字节与字符的区别

字节主要用于读取非文本数据，如声音，图片，其数据体现在二进制位 上，而字符则是根据特定编码集对应得到的。

#### 111.Tomcat和Apache区别以及JBoss

**Apache:**使用C编写，是http服务器，**只支持html静态页面**。而动态页面的支持如jsp则是通过tomcat整个实现。即请求动态页面时，tomcat相应请求，然后解析jsp页面解析后回传给apache。**apache+tomcat模式减少了tomcat的开销，即不用单独进行http解析。**

**Tomcat:**java编写，支持servlet和jsp，本质是可以替代apache，主要是apache应用广泛。

**JBoss**：开源的应用服务器，比较受人喜爱，免费（文档要收费）。Jboss**内嵌Tomcat**，处理静态页面Jboss的速度要比较快。**JBoss Web达到了可扩展性，性能参数匹配甚至超越了本地Apache HTTP服务器或者IIS**。譬如JBoss Web能够提供数据库连接池服务，不仅支持 JSP 等 Java 技术，同时还支持其他 Web 技术的集成，譬如 PHP、.NET 两大阵营

 

#### 112.HTTPS的加密方式

非对称加密+对称加密

**非对称加密**：即浏览器根据证书中的**公钥对协商好的秘钥进行加密**后传输给服务器，服务器**用私钥解密得到回话秘钥**。

**对称加密：**然后浏览器和服务器之间的通信则根据**协商的秘钥进行对称加密通信。**

非对称加密特点是**私钥加密后的密文，只要是公钥，都可以解密，但是反过来公钥加密后的密文，只有私钥可以解密**。私钥只有一个人有，而公钥可以发给所有的人。



#### 113.如何在遍历集合时修改元素并保证线程安全？

在遍历时修改元素应使用迭代器，Iterator,但迭代器本身不能保证线程安全。因此，在获取迭代器时应当在外层对集合加锁实现线程安全。

```
final static List asList = Collections.synchronizedList(new ArrayList(Arrays.asList("a","b")));

public static void main(String[] args) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {Thread.sleep(150); } catch (InterruptedException e) { }
            reload();
        }
    }).start();
    synchronized (asList) {
      for(String e:asList){
        try {Thread.sleep(100);} catch (InterruptedException e1) {}
        System.out.println(e);
      }
    }
}
public static void reload(){
  asList.clear();
  asList.addAll(Arrays.asList("a","b","c"));
}
```



即添加**synchronized（list）**同步快实现线程安全。

#### 114.Reentranlock可中断获取锁的意义？lock和trylock()的区别

可中断获取锁是指在**尝试获取锁时或者开始休眠挂起时可以被中断**，从而取消获取锁，**而非相应已经获取锁后被中断**。

lock获取锁会一直阻塞，而trylock()则不会阻塞，因为是立即返回。且trylock(long,timeUnit)可以设置时间，即在超时时间内一直尝试获取锁，**且可以相应中断**。



#### 115.Java对象的克隆

1.可以实现Clonable接口并实现clone()方法

**2.先序列化再反序列化。**（new ObjectOutputStream(new ByteArrayOutputStream())）

后者可以实现深度克隆，且更加科学，因为若原对象不支持序列化，则会在编译时抛出异常。而前者会导致运行时抛出异常。

#### 116.深拷贝与浅拷贝

浅拷贝只复制了待赋值对象中内部依赖对象的引用地址。而深拷贝则是将对象引用和值都赋值一份。修改时不会影响后者

#### 117.外部类与内部类变量的访问权限

外部类**不能任意访问内部类的成员**，要做到这一点，外部类必**须创建其内部类的对象**（**静态**
**内部类除外，因为静态内部类就相当于静态成成员变量，相当于已经初始化**）。内部类可以访问 all 外部类的成员，因为内部类就像是一个类的成员



#### 118.concurrentHashMap的get操作不加锁的原因与协助扩容

1）tabAt()方法保证了table[]的可见性，即通过Unsafe.getObjectVolatile来获取table[i]的值。只是volatile本身不能保证线程每次都是拿到最新的值，因此需要该方法保证。

2）当hash<0时，说明是红黑树节点（-2）或者已经移动了（-1）,则需要调用相应节点类型的find()方法，如TreeNode和ForwadingNode，后者保存了一个指针nextTable指向新table表

**协助扩容的条件**：put操作当访问到节点hash=-1时，则会调用helpTransfer（）方法进行判断。当检测到hash表正在扩容时，即sizeCtl<0，则协助扩容。当一个tab[i]迁移完成，则会将头节点变为ForwardingNode。

#### 119.在SQL标准中，RR是无法避免幻读问题的，但是InnoDB实现的RR避免了幻读问题。

#### 120.MYSQL分页查询语句

 查询第 20 条到第 30 条的是：**select * from table limit 20,10**;对应的就是第三页的数据。

**总结**： select * from table limit （页数-1） *每页条数, 每页条数;

####  121.DNS既使用TCP也使用UDP

**区域传送时使用TCP，主要有一下两点考虑：** 
1.辅域名服务器会定时（一般时3小时）向主域名服务器进行查询以便了解数据是否有变动。如有变动，则会执行一次区域传送，进行数据同步。区域传送将使用TCP而不是UDP，**因为数据同步传送的数据量比一个请求和应答的数据量要多得多。** 
2.TCP是一种可靠的连接，保证了数据的准确性。 

**域名解析时使用UDP协议：** 
客户端向DNS服务器查询域名，一般返回的内容都不超过512字节，用UDP传输即可。不用经过TCP三次握手，这样DNS服务器负载更低，响应更快。虽然从理论上说，客户端也可以指定向DNS服务器查询的时候使用TCP，但事实上，很多DNS服务器进行配置的时候，仅支持UDP查询包。

122.静态代理与动态代理的区别

**静态代理**：AspectJ，其是依赖于特定的编译工具生成了特定的代理类，在编译阶段对目标类进行了更改。

**动态代理：****1.jdk动态代理**：底层就是反射调用特定方法

​		    **2.CGlib动态代理**：底层实质是继承重写调用。采用了底层字节码技术，在运行时通过继承目标类，通过目标类的字节码文件穿件一个子类，即代理对象，在子类中字节码文件中生成了两个方法，一个是原方法，一个是代理修改后的方法

![1554014656850](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1554014656850.png)

#### 122.数据库的四大范式

1NF： 属性不可分（即没有内容重复或者相似的两个列，例如地址可拆分为不同的省，市。。）； 

2NF： 非主键属性完全依赖于主键属性（例如下订单开房间，一张表会导致大量联系人，电话重复，故拆分）； 3NF： 非主键属性之间无传递依赖（例如学生表，根据学号可以推出学校名，学校地址，存在依赖）； 

4NF：主键属性之间无传递依赖



#### 123.线程的阻塞状态：同步阻塞和等待阻塞的区别

同步阻塞：是指没能进入synchronized修饰的代码块或者方法，放入了监视器对象的竞争队列

等待阻塞：是指进入synchronized后主动调用this.wait()，放入wait等待集合中。一旦满足条件则会继续向下执行。

#### 124.TreeMap的排序问题

若一个 treemap集合中放的 student，按 student的 age排序，把 10个 student放入 treemap中（即treemap.put(student1,value),通过TreeMap构造器传入了comparator），他们已经排好序了，若修改一个 student 的 age， 他们顺序不会变

#### 125.byte、 short、 int、 long、 char、 boolean 包装类实现了的常量池（ float、 double没有）也就是说，Double d1=2.3,Double d2=2.3任何情况都为false

#### 126.SQL的on和where的区别

on用于**表连接**时的条件限定，然后再用where进行筛选。即on是用来生成临时表时使用，where是在表已存在时使用

#### 124.哈夫曼树与哈夫曼编码

**哈夫曼树（最优二叉树）**：

1.根据给定的n个权值{w1,w2,…,wn}构成二叉树集合F={T1,T2,…,Tn},其中每棵二叉树Ti中只有一个带权为wi的根结点,其左右子树为空.
2.在F中选取两棵根结点权值最小的树作为左右子树构造一棵新的二叉树,且置新的二叉树的根结点的权值为左右子树根结点的权值之和.
3.在F中删除这两棵树,同时将新的二叉树加入F中.
4.重复2、3,直到F只含有一棵树为止.(得到哈夫曼树)

![1554274271173](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1554274271173.png)

![1554274276140](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1554274276140.png)

#### 125.读写锁锁降级的原因

读写锁的锁降级的原因：为了保证写数据的**可见性**。若写锁必须先释放再获取读锁，则可能会有其他写锁在释放时获取写锁，然后再次更改数据。从而导致第一次写锁的写数据不可见



锁降级就是一种特殊的锁重入机制，JDK 使用 `先获取写入锁，然后获取读取锁，最后释放写入锁` 这个步骤，是为了提高获取锁的效率，**而不是所谓的可见性。**



#### 126.Top K问题完整解析

<https://www.cnblogs.com/qlky/p/7512199.html?tdsourcetag=s_pctim_aiomsg>

#### 127.canal 数据库增量订阅/消费中间件

![1554904491777](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1554904491777.png)

![1554904694268](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1554904694268.png)

![1554904813346](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1554904813346.png)

#### 128.Zookeeper版本号即乐观锁保证创建数据节点（Znode）节点信息的线程安全性

#### 129.java外部类和内部类的关系

所谓内部类，其实就能看作外部类的一个成员变量。**其内部变量和方法可以相互访问**。

 java外部类**可访问内部类的私有方法**：因为反编译会有一个静态方法访问内部类的私有属性，即**内部类中都有一个缺省的外部类的引用**，印证java设计思想，把内部类当做外部类的一个属性。

 在内部类Outer$Inner中， 存在一个名字为**this$0** ， 类型为Outer的成员变量， 并且这个变量是final的。 其实这个就是所谓的“在内部类对象中存在的指向外部类对象的引用”



#### 130.Concurrent的读操作不加锁与volatile

ConcurrentHashMap的读操作不需要加锁，因为结点**Node的val和key**用volatile修饰，保证了可见性，防止读到脏数据。**对数组的volatile是保证了扩容的可见性**

#### 131.SOA与微服务的区别

SOA就相当于**单一系统下的服务化**，多个服务运行在同一个web容器中，用于功能调用和增加。微服务则是将**系统功能独立拆分，分为多个子系统。不同子系统采用不同架构，运行在自己的web容器中，应用升级不会相互影响。**

![1555317140753](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1555317140753.png)

![1555318036548](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1555318036548.png)

#### 131.SpringMVC如何保证DAO层线程安全？

<https://blog.csdn.net/liou825/article/details/17363265>

数据库连接**Connection在不同线程中是不能共享**的，事务管理器为不同的事务线程利用ThreadLocal类提供独立的Connection副本。

实现service和DAO对象时使用了ThreadLocal类解决了线程安全问题。**Spring中DAO和Service都是以单实例的bean形式存在，Spring通过ThreadLocal类将有状态的变量（例如数据库连接Connection）本地线程化，从而做到多线程状况下的安全。在一次请求响应的处理线程中， 该线程贯通展示、服务、数据持久化三层，通过ThreadLocal使得所有关联的对象引用到的都是同一个变量。**



#### 132.保证缓存和数据库的一致性

1.先删除缓存，再更新数据库，而不是更新缓存。有个问题就是还未更新数据库时，另一个查询请求在缓存中未找到，直接查询数据库，再放入缓存，导致数据不一致。在并发量特别高的情况才会出现。

2.将删除缓存和更新数据库放入消息队列，串行读取。但可能导致更新请求太多读请求被阻塞太长。

3.使用canal异步更新



#### 133.保证主从数据库读写分离的实时一致性

由于主从数据库同步时间的延时性，当写数据库后，做以下操作：

1.在redis做一个标记字段，数据库名表名和业务id，每次访问都查询该字段，若存在，就走主库。

2.在本地cookie中添加标记，服务端取出遍历选择主库。

3.本地缓存。利用前端将本地提交内容和从库获取内容进行拼接，并设置过期时间为从库更新时长



#### 134.JDBC为什么使用Class.forName()加载驱动而不是使用ClassLoader？

Class.forName()默认是执行类加载以及初始化，即执行static代码块。而classloader只执行了类加载。

因为使用Class.forName(classname)才能在反射回去类的时候执行static块。
复制代码
static {
​    try {
​        java.sql.DriverManager.registerDriver(new Driver());
​    } catch (SQLException E) {
​        throw new RuntimeException("Can't register driver!");
​    }
}



#### 135.java中保存图信息

使用**邻接表**保存图信息。在java中，需要定义一个**Graph类**（保存Vertex数组和顶点数和边数）和**Vertex类**（保存顶点名和下一个顶点）。再定义一个**图的实现类**，提供**getVertex方法**用于在Graph中遍历找到对应的顶点以及创建图的方法**createGraph**。然后取出边的两个顶点，使用getVertex()方法**，加到链表头部。**



#### 136.如何实现业务的幂等性？

1.token机制防止重复提交：token+resis,即数据提交时需要申请token,服务器将其放到redis或jvm内存（单jvm），并设置有效时间。用户提交请求，在后台尝试删除token,删除成功则校验通过。

2.悲观锁，即获取数据时for update行锁。

3.乐观锁，即新增版本号字段，每次提交匹配版本号。

4.分布式锁：由于乐观锁的更新操作依赖于全局唯一索引，这样是行锁，效率高。但在分布式环境下失效，因此可以利用分布式锁，从而实现同一时间只有一个执行成功。

5.请求附带来源和序列号，作为数据库唯一索引，避免多次付款。即根据唯一索引现在系统内查询是否已经处理过，若处理，则直接返回结果。避免直接插入业务系统。

#### 137.如何反射获取方法的参数名？

通过反射可以获取方法名，方法参数类型，获取参数名在jdk1.8支持，例如spring绑定url参数名就是通过这实现。1.8之前，可以绑定注解，在注解里面写入参数，然后获取注解即可。

#### 138.如何保证分布式的一致性？

1.数据库原生支持的**XA接口**，但mysql支持不好，未记录prepare阶段日志。分为本地事务管理器和全局事务管理器，分预备就绪，提交成功两个阶段。

2.**消息事务（RocketMQ）**，即将发送消息的代码放在本地事务中。首先发送预备消息，返回成功后执行本地事务，发送提交消息。只要成功提交消息，则B系统就不断尝试消费直到成功。若MQ未收到commit消息则会导致不一致性，此时是通过**超时询问机制**实现。超时则进行询问

3.**TCC**。try阶段未执行业务，只进行业务一致性检查和预留资源，confirm阶段执行业务，如果失败则进入cancel，恢复库存。（例如转账，需要一个协调者。try阶段首先创建一个转账流水，A账户扣除100，即预留资源，然后confirm阶段B账户增加100，并更新流水状态为已完成。若发生异常，则进入cancel阶段，A账户增加100，并设为交易失败。）



#### 139.如何保证成功发送消息到MQ？

1.开启MQ事务 

2.开启confirm模式，即分配一个唯一id，写入MQ会回传一个ack。否则会回调本地提供的回调nack接口，表示失败，重新发送，并可维护一个消息id的状态。

#### 140.ThreadLocal的应用场景

1.Spring事务管理器中将DataSource中获取的Connection对象放入ThreadLocal中。保证了执行事务时获取的connection是同一个。

2.Session管理



#### 141.HTTP Server的模型分类

<https://blog.csdn.net/a772304419/article/details/79510217>

当前比较主流的实现 HTTP Server 的模型主要有两类——「**线程模型**」以及「**事件模型**」。

**线程模型服务器**：Tomcat，其支持nio，bio和apr（本身也是nio，只是给予了原生方法调用，基于OS级别的支持，可以提高Tomcat对静态文件以及https的处理性能。）模式

![img](https://img-blog.csdn.net/20180310175019698)

「接收者线程」（Accpter thread）接受客户端的 HTTP 请求，然后将这些请求分配给「请求处理线程」进行处理。

这种模型的弊端就是，「工作线程」（也就是上面提到的「请求处理线程」）是有限的，而客户端发来的请求数量往往会大于「工作线程」的数量。当此种情况发生时，那些没有得到处理的线程就会一直处于阻塞和等待状态，反映到用户层面就是页面迟迟得不到响应，如果等待时间过长，耐心的用户最终会看到请求超时（Request timeout）的信息，急性子的用户就会关掉这个页面。

另外采用线程这种方式也非常地耗费资源，如果某个请求很耗费时间，那么处理该请求的工作线程大概是这样工作的：

![img](https://img-blog.csdn.net/20180310175020010)

**事件模型服务器**:Play框架，其基于netty或者akkaHTTP（最新版），具有异步非阻塞的有点。

![img](https://img-blog.csdn.net/20180310175020585)



当用户发送一个请求的时候，往往这个请求包含了许多操作，而 Play! 则能将这些请求分割为一个一个的事件，然后异步去处理这些事件。例如，当某一个事件正在被操作系统处理的时候，这个过程可能会花费一些时间，之前说过，如果线程一直等待这个事件执行完然后再去执行下一个事件就有点浪费资源了，所以在这个等待时间里，event loop（消息线程）可以去执行事件队列中的其他事件。当某个事件执行完之后，就会发出一个中断，这个中断也算一个事件，然后加入到事件队列中，等待执行。这种异步非阻塞的模式使得 Play! 能够以较少的资源应对大流量的访问。反映在用户层面就是，Play! 能够快速地对用户的行为作出响应。

![img](https://img-blog.csdn.net/20180310175020863)

图中绿色的部分为事件的执行时间，橙色部分为「空闲时间」，注意这里是「空闲时间」而非前面所说的「等待时间」，在这个空闲时间内，event loop 可以去执行其他事件而不必等待前面某个事件执行完成，当某个事件执行完成之后，会发出中断，这个中断也会产生一个新的事件，最终 event loop 也会执行这个事件。这就是「事件模型」处理某个请求的流程，可以看到，没有了等待时间，大大提高了程序运行的效率，也使得系统能够以较少的资源处理大量的请求。

