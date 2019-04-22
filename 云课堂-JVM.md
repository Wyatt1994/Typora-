## java类的三种状态：

SOURCE:源码状态（静态）

CLASS:二进制的字节码状态（静态，即javac命令进行编译得到）

RUNTIME:运行时状态，即加载到JVM中的状态（即对应java xxx命令）

![1541685847991](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541685847991.png)

链接包含**验证、准备、解析**三部分

![1541685879783](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541685879783.png)

加载：读取class文件数据到运行时数据区：即使用ClassLoader.loadClass（String className）加载，例Class.forName(com.jdbc.Driver)，即相当于调用ClassLoader类的loadClass（String className）方法

1.当有线程加载在加载类的时候，别的线程不能加载。即利用Synchronized方法进行同步，保证线程安全

2.进入同步块判断类是否被加载

3.当这个类没被加载时才会被加载，即Class c=findloadedClass(className),c==null

4.ClassLoader类中有一个private final ClassLoader parent;

5.如果父类不为空，则递归加载父类。findBootstrapClassOrNull会调用一个findBootstrapClass()本地方法。bootstrap Classloader即为根类加载器

**源代码分析如下：**

![1541685925144](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541685925144.png)



## JVM类加载执行顺序



![1541685948626](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541685948626.png)

![1541685955541](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541685955541.png)

![1541685965010](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541685965010.png)

### 双亲委派：

下述过程即为**双亲委派**：1.父类能加载就不给子类加载 2.子类是加载过的，则不进行装载。即对应上述代码

正因为第二点，所以java文件的类名不能跟java自带的类名相同。只能被编译，不能被执行。

#### 原因：

保证核心api库不被篡改和避免重复加载。

![1541686022984](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686022984.png)

![1541686055527](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686055527.png)

## JVM运行时数据区结构

![1541686078923](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686078923.png)

### JAVA堆

![1541686136862](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686136862.png)

### JAVA栈

#### 栈帧的结构

栈帧中数据分为局部变量表，操作数栈和返回地址（return指令)

![1541686151980](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686151980.png)

![1541686262561](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686262561.png)

#### 栈的局部变量表

局部变量表存放了八种**基本数据类型**和**引用类型**和**return**指令

![1541686328521](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686328521.png)





### 方法区

类结构信息和方法信息（即对应生成的字节码指令）都放入方法区，例如Person p=new Person("wyatt"),字符串“wyatt”是常量，放在了方法区中常量池中，而new Person() 即将实例对象放在堆里面，而Person p则是将p...这些引用是放入栈中的局部变量表中。**当调用p.sayHello()时，会从栈的中的局部变量表中的p引用找到堆中的实例，而通过实例指向了方法区中sayHello()字节码的具体位置，然后执行该方法获得结果。**

具体见JVM执行流程。

#### 生成的字节码指令：

![1541686247436](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686247436.png)

## JVM内存结构

![1541686189937](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686189937.png)

### JVM堆、栈和方法区

![1541686409080](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686409080.png)

#### 为什么String s跟new String“==”为false?

如下图，因为String s="1234"的引用是直接保存的的常量池中的地址，而new String的引用s1是指向了堆中的String实例的地址，再由实例中找到常量池中的“1234”的具体地址。

![1541686838086](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686838086.png)

## JVM执行流程

![1541686468625](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686468625.png)

## JVM内存区域示意图

注：在JDK8之后，逐渐淡化永久代。

![1541686496393](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1541686496393.png)



![1542267757282](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542267757282.png)

 JDK1.8之后，将永久代从堆中移除，变为元空间（MeteData,直接内存,包含静态变量，常量，类信息），而不是使用堆内存。

新生代：老年代默认比例为1:2

![1542273209122](E:\Pandoc笔记\%5CUsers%5CASUS%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1542273209122.png)

## JVM垃圾收集器

https://mp.weixin.qq.com/s?__biz=MzI3ODg2OTY1OQ==&mid=2247483973&idx=1&sn=aceedab6b71c209c4d1dc590623329d5&chksm=eb5121b1dc26a8a710d69c2914f9fffd38325c12622147a42a8da9ecb9a9dfeacf75b6e30f6e&mpshare=1&scene=1&srcid=1028cO0zmCpR688nxmtgiDPi#rd

jdk9采用**G1收集器**，jdk8采用parallel scvange和parallel old。

#### Serial收集器（新生代收集器）：

采用复制算法、Serial：串行的意思。由名字就可知这是一个单线程的收集器，“单线程”的意义并不仅仅说明它只会使用一个cpu或是一条收集线程去完成垃圾收集工作，更重要的是在它进行垃圾收集时，**必须暂停其他所有的工作线程**，**直到垃圾收集结束**。“Stop the world”是由虚拟机在后台自动发起和完成的，在用户不可见的情况下把用户正常工作的线程全部停掉，意味着“你的计算机每工作一小时就会暂停响应5分钟。但是实际上它依然是**虚拟机运行在client模式下的默认新生代收集器**。它也有着优于其他收集器的地方：简单而高效。在用户的桌面应用场景中，分配各虚拟机管理的内存一般不会很大，收集几十兆甚至一两百兆的新生代，停顿时间可以控制在几十毫秒最多一百多毫秒以内，只要是不平凡发生这点停顿是可以接收的。所以Serial收集器对于运行在Client模式下的虚拟机来说是一个很好的选择。

#### ParNew收集器**（新生代收集器）:**

**它其实就是Serial收集器的多线程版本，**复制算法**，除了使用多条线程进行垃圾收集外，其余行为包括Serial收集器可用的所有控制参数。收集算法、Stop The World、回收策略等都与Serial收集器完全一样。Serial和parNew两个收集器都可以并且只可以与老年代的CMS和serial old GC一起工作。

#### **Parallel Scavenge收集器**（新生代）：

它是使用**复制算法**的收集器，它可以和parallel old和serial old一起工作。它的关注点与其他收集器不同，CMS等收集器是尽可能的缩短垃圾收集时用户线程的停顿时间。Parallel Scavenge收集器**关注的是吞吐量**，目标是达到一个可控制的吞吐量，**吞吐量=运行用户代码时间 /（运行用户代码时间+垃圾收集时间）**，即为CPU运行用户代码的时间与CPU总消耗时间的网速。停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验；而高吞吐量则可以高效率的利用CPU时间，尽快完成程序的运算任务，适合在后台运算运算并且不需要太多交互的任务。Parallel Scavenge收集器提供了设置最大垃圾收集停顿时间:-XX：MaxGCPauseMills(收集器将尽量保证内存回收时间不超过设定值，但是注意这是以牺牲吞吐量和新生代空间为代价的：把它设置得太小：系统将会调整新生代空间，因为回收300M新生代肯定比回收500M快，但是GC的频率也随之增大了)和吞吐量大小:-XX:GCTimeRatio的参数以及一个开关参数UseAdaptiveSizePolicy，可以自动优化调整新生代（-xnm）大小、Eden与Survivor比值（-XX:SurvivorRatio）、晋升老年代大小（-XX：PretenuredThreshold）等细节参数，虚拟机会根据当前系统运行情况当太调整这些参数已提供最合适的停顿时间或者最大的吞吐量，这种方式称为“GC自适应”的调节策略。如果对收集器运作原理不太了解，手动优化存在困难时，使用Parallel Scavenge收集器把内存优化管理的任务交给虚拟机（只需要设置基本内存数据：-Xmx、最大垃圾收停顿时间（更关注停顿时间）或者吞吐量（更关注吞吐量））。自适应调节策略”也是Parallel Scavenge收集器与ParNew收集器的一个重要区别。

#### **Serial Old**

是Serial收集器的**老年代**版本，它同样是是一个单线程收集器。使用**“标记-整理”**算法，主要意义也是给Client模式下的虚拟机使用。

#### Parallel Old

是Parallel Scavenge收集器的老年代版本，采用**“标记-整理算法”**。在注重吞吐量以及CPU资源敏感的场合可以优先考虑：Parallel Scavenge + Parallel Old组合。

#### CMS

（Concurrent Mark Sweep）收集器：一种以获取**最短回收停顿时间**为目标的收集器（希望系统停顿时间最短，以给用户带来较好的体验）。从名字中的“Mark Sweep”可以看出CMS收集器是基于“标记-清除”算法实现的，它的运作过程可分为4个步骤：**初始标记、并发标记、重新标记、并发清除**。其中，**初始标记、重新标记这两个步骤仍然需要“Stop the World”**。初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快；并发标记阶段就是进行GC Roots Tracing的过程；而重新标记阶段，则是为了修正并发标记期间因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这一阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记时间短。**整个过程只有初始标记和重新标记需要“stop the world”**，具有并发、低停顿优点。但是它由三个明显缺点：1.CMS收集器对CPU资源非常敏感：在并发阶段虽然不会导致用户线程停顿，但是会因为占用了一部分线程（CPU资源）而导致应用程序变慢，总吞吐量降低；CMS收集器**无法收集浮动垃圾**：可能出现“Concurrent Mode Failure”失败而导致来一次Full GC的产生（这时会使用**serial old**作为CMS的临时替代收集器）。CMS并发清理阶段用户线程还在运行，期间自然会有新的垃圾产生，只能**等待下一次GC时在清理**，这部分垃圾称为“浮动垃圾”。另外，由于在垃圾收集阶段用户线程还需要运行，那也就是还需要预留足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎被完全填满了在进行收集，需要预留一部分空间供并发收集期间的程序运作使用；CMS是一款**基于“标记-清除”**算法实现的收集器，这意味着GC后会有**大量的空间碎片**产生。空间碎片过多将会给大对象分配带来很大的麻烦，往往会出现老年代还有很大的空间剩余，但是无法找到足够大的连续空间分配给当前对象，从而不得不提前触发一次Full GC。对此CMS提供了一个参数，用于在触发Full GC时开启内存碎片的合并整理过程，内存整理过程是无法并发的，空间碎片问题没有了，但是停顿时间不得不变长。

#### G1收集器

是当今收集器技术发展的最前沿超过之一。G1是一款**面向服务端应用**的垃圾收集器，具有如下特点：并发与并行：可以**充分利用多CPU、多核环境**来缩短“Stop the world”的时间；**1 分代收集**：G1可以不需要其他收集器配合就**可以独立管理整个GC堆**，但它能够采取不同的方式去处理新创建的对象和已经存活了一段时间、熬过多次GC的旧的对象以获得更好的收集效果；**2 空间整合**：与CMS的“标记-清理”算法不同，G1从整体来看是基于“标记-整理算法”，从局部（两个region之间）看是基于“复制”算法实现的。但无论如何，这两种算法意味着G1运作期间**不会产生内存碎片**，这种特性有利于程序长时间运行；**3 可预测的停顿**：这是G1相对于CMS的另一大优势，降低停顿时间是G1和CMS共同的关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在在垃圾收集上的时间不得超过N毫秒，这几乎是实时的java垃圾收集器的特征了。在G1之前的其他收集器进行收集的范围都是整个新生代或者老年代，使用G1收集器时，它将整个java堆划分为多个大小相等的独立区域，虽然还有**新生代和老年代的区别，但是新生代和老年代不再是物理隔离**了，它们都是一部分Region（不需要连续）的集合。**G1优先回收价值最大的Region**（有限时间内获取尽可能高的效率）。G1收集器的运作大致可划分为以下几个步骤：**初始标记、并发标记、最终标记、筛选回收**。初试标记阶段仅仅是只是标记下GC Roots能直接关联到的对象，这阶段需要**停顿线程，**但是耗时很短；并发标记：从GC Roots开始对堆中对象进行可达性分析，找出活的对象，这部分耗时较长，但是可以与用户程序并发执行。最终标记：为了修正在并发标记期间因用户程序继续运行而导致标记产生变动的那一部分标记记录，这阶段需要**停顿线程**，但是可以并行执行。

#### CMS垃圾收集器有哪些缺点？

- 1. **CMS垃圾收集器对CPU资源非常敏感**
     虽然并发标记和垃圾回收这两个阶段不会导致用户线程停顿，但是由于垃圾收集器会占用相当一部分的CPU资源会导致用户的应用程序变慢。
- 2. **CMS垃圾收集器无法处理浮动垃圾**
     可能出现"Concurrent Mode Failure"失败而导致另一次FullGC的产生。因为在垃圾收集阶段用户线程还在执行，这将导致在该阶段用户线程产生的垃圾不能进行回收，只能等到下次回收，因此CMS垃圾收集器每次都需要预留一部分空间用户线程使用，而不能等到老年代几乎满了再进行收集，如果预留的内存不够用，那么就会出现一次"Concurrent Mode Failure"失败，此时JVM会采用预备方案--临时使用Serial Old垃圾收集器进行老年代的垃圾回收，这样停顿时间就很长了
- 3. **CMS垃圾收集器会产生内存碎片**
     CMS垃圾收集器采用的是标记-清除算法进行垃圾回收，这样不可避免的会产生内存碎片的问题，可以使用-XX:+UseCMSCompactAtFullCollection参数开启内存碎片的合并整理过程，这个过程也是需要STW的。还可以通过-XX:CMSFullGCsBeforeCompaction设置执行多少次不压缩的FullGC之后跟着来一次带压缩的FullGC



