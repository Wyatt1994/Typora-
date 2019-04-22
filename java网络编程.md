## NIO

http://www.iteye.com/magazines/132-Java-NIO

三大核心组件：

- Buffer
- Channel
- Selector

### Buffer

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。 

Java NIO 还有个 Mappedyteuffer，用于表示内存映射文件

**position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式，capacity的含义总是一样的。** 

这里有一个关于capacity，position和limit在读写模式中的说明，详细的解释在插图后面。 

![img](http://dl2.iteye.com/upload/attachment/0096/4782/b8a7bad8-ec65-36dc-bb11-4f352e00cd67.png)

**capacity**

作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类**型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。** 

**position**

当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为**capacity – 1**。 

当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0。当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。 

**limit**

在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。 

当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position） 



  **flip()方法** 

flip方法将Buffer从**写模式切换到读模式**。调用flip()方法会将**position设回0，并将limit设置成之前position的值**。 

换句话说，**position现在用于标记读的位置，limit表示之前写进了多少个byte、char等 —— 现在能读取多少个byte、char等**。   



  **rewind()方法** 

Buffer.rewind()将position设回0，所以你可以**重读Buffer中的所有数据**。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。  

  **clear()与compact()方法** 

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。 

如果调用的是**clear()方法**，**position将被设回0，limit被设置成 capacity的值**。换句话说，Buffer 被清空了。**Buffer中的数据并未清除**，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。 

如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。 

如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。

**compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面**。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是**不会覆盖未读的数据**。   

### 通道Channel

| 区别                 | 流             | 通过Channel                                      |
| -------------------- | -------------- | ------------------------------------------------ |
| 支持异步             | 不支持         | 支持                                             |
| 是否可双向传输数据   | 不能，只能单向 | 可以，既可以从通道读取数据，也可以向通道写入数据 |
| 是否结合 Buffer 使用 | 不             | 必须结合 Buffer 使用                             |
| 性能                 | 较低           | 较高                                             |

```java
public interface Channel extends Closeable {

    /**
     * 判断此通道是否处于打开状态。 
     */
    public boolean isOpen();

    /**
     *关闭此通道。
     */
    public void close() throws IOException;

}
```

- FileChannel：一个用来写、读、映射和操作文件的通道
- DatagramChannel：能通过 UDP 读写网络中的数据
- SocketChannel: 能通过 TCP 读写网络中的数据
- ServerSocketChannel：可以监听新进来的 TCP 连接，像 Web 服务器那样。对每一个新进来的连接都会创建一个 SocketChannel

### 多路复用器 Selector

多路复用器 Selector，它是 Java NIO 编程的基础，它提供了**选择已经就绪的任务**的能力。从底层来看，Selector 提供了询问通道是否已经准备好执行每个 I/O 操作的能力。简单来讲，Selector 会**不断地轮询注册在其上的 Channel**，如果某个 Channel 上面发生了读或者写事件，这个 Channel 就处于就绪状态，会被 Selector 轮询出来，然后通过 **SelectionKey 可以获取就绪 Channel 的集合**，进行后续的 I/O 操作。

Selector 允许一个线程处理多个 Channel ，也就是说只要一个线程复杂 Selector 的轮询，就可以处理成千上万个 Channel ，相比于多线程来处理势必会减少线程的上下文切换问题。

**服务端**

```java
public class NIOServer {

    /*接受数据缓冲区*/
    private ByteBuffer sendbuffer = ByteBuffer.allocate(1024);
    /*发送数据缓冲区*/
    private  ByteBuffer receivebuffer = ByteBuffer.allocate(1024);

    private Selector selector;

    public NIOServer(int port) throws IOException {
        // 打开服务器套接字通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 服务器配置为非阻塞
        serverSocketChannel.configureBlocking(false);
        // 检索与此通道关联的服务器套接字
        ServerSocket serverSocket = serverSocketChannel.socket();
        // 进行服务的绑定
        serverSocket.bind(new InetSocketAddress(port));
        // 通过open()方法找到Selector
        selector = Selector.open();
        // 注册到selector，等待连接
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("Server Start----:");
    }

    //
    private void listen() throws IOException {
        while (true) {
            selector.select();
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey selectionKey = iterator.next();
                iterator.remove();
                handleKey(selectionKey);
            }
        }
    }

    private void handleKey(SelectionKey selectionKey) throws IOException {
        // 接受请求
        ServerSocketChannel server = null;
        SocketChannel client = null;
        String receiveText;
        String sendText;
        int count=0;
        // 测试此键的通道是否已准备好接受新的套接字连接。
        if (selectionKey.isAcceptable()) {
            // 返回为之创建此键的通道。
            server = (ServerSocketChannel) selectionKey.channel();
            // 接受到此通道套接字的连接。
            // 此方法返回的套接字通道（如果有）将处于阻塞模式。
            client = server.accept();
            // 配置为非阻塞
            client.configureBlocking(false);
            // 注册到selector，等待连接
            client.register(selector, SelectionKey.OP_READ);
        } else if (selectionKey.isReadable()) {
            // 返回为之创建此键的通道。
            client = (SocketChannel) selectionKey.channel();
            //将缓冲区清空以备下次读取
            receivebuffer.clear();
            //读取服务器发送来的数据到缓冲区中
            count = client.read(receivebuffer);
            if (count > 0) {
                receiveText = new String( receivebuffer.array(),0,count);
                System.out.println("服务器端接受客户端数据--:"+receiveText);
                client.register(selector, SelectionKey.OP_WRITE);
            }
        } else if (selectionKey.isWritable()) {
            //将缓冲区清空以备下次写入
            sendbuffer.clear();
            // 返回为之创建此键的通道。
            client = (SocketChannel) selectionKey.channel();
            sendText="message from server--";
            //向缓冲区中输入数据
            sendbuffer.put(sendText.getBytes());
            //将缓冲区各标志复位,因为向里面put了数据标志被改变要想从中读取数据发向服务器,就要复位
            sendbuffer.flip();
            //输出到通道
            client.write(sendbuffer);
            System.out.println("服务器端向客户端发送数据--："+sendText);
            client.register(selector, SelectionKey.OP_READ);
        }
    }

    /**
    * @param args
    * @throws IOException
    */
    public static void main(String[] args) throws IOException {
        int port = 8080;
        NIOServer server = new NIOServer(port);
        server.listen();
    }
}
```

**客户端**

```java
public class NIOClient {
    /*接受数据缓冲区*/
    private static ByteBuffer sendbuffer = ByteBuffer.allocate(1024);
    /*发送数据缓冲区*/
    private static ByteBuffer receivebuffer = ByteBuffer.allocate(1024);

    public static void main(String[] args) throws IOException {
        // 打开socket通道
        SocketChannel socketChannel = SocketChannel.open();
        // 设置为非阻塞方式
        socketChannel.configureBlocking(false);
        // 打开选择器
        Selector selector = Selector.open();
        // 注册连接服务端socket动作
        socketChannel.register(selector, SelectionKey.OP_CONNECT);
        // 连接
        socketChannel.connect(new InetSocketAddress("127.0.0.1", 8080));

        Set<SelectionKey> selectionKeys;
        Iterator<SelectionKey> iterator;
        SelectionKey selectionKey;
        SocketChannel client;
        String receiveText;
        String sendText;
        int count=0;

        while (true) {
            //选择一组键，其相应的通道已为 I/O 操作准备就绪。
            //此方法执行处于阻塞模式的选择操作。
            selector.select();
            //返回此选择器的已选择键集。
            selectionKeys = selector.selectedKeys();
            //System.out.println(selectionKeys.size());
            iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                selectionKey = iterator.next();
                if (selectionKey.isConnectable()) {
                    System.out.println("client connect");
                    client = (SocketChannel) selectionKey.channel();
                    // 判断此通道上是否正在进行连接操作。
                    // 完成套接字通道的连接过程。
                    if (client.isConnectionPending()) {
                        client.finishConnect();
                        System.out.println("完成连接!");
                        sendbuffer.clear();
                        sendbuffer.put("Hello,Server".getBytes());
                        sendbuffer.flip();
                        client.write(sendbuffer);
                    }
                    client.register(selector, SelectionKey.OP_READ);
                } else if (selectionKey.isReadable()) {
                    client = (SocketChannel) selectionKey.channel();
                    //将缓冲区清空以备下次读取
                    receivebuffer.clear();
                    //读取服务器发送来的数据到缓冲区中
                    count=client.read(receivebuffer);
                    if(count>0){
                        receiveText = new String( receivebuffer.array(),0,count);
                        System.out.println("客户端接受服务器端数据--:"+receiveText);
                        client.register(selector, SelectionKey.OP_WRITE);
                    }

                } else if (selectionKey.isWritable()) {
                    sendbuffer.clear();
                    client = (SocketChannel) selectionKey.channel();
                    sendText = "message from client--";
                    sendbuffer.put(sendText.getBytes());
                    //将缓冲区各标志复位,因为向里面put了数据标志被改变要想从中读取数据发向服务器,就要复位
                    sendbuffer.flip();
                    client.write(sendbuffer);
                    System.out.println("客户端向服务器端发送数据--："+sendText);
                    client.register(selector, SelectionKey.OP_READ);
                }
            }
            selectionKeys.clear();
        }
    }
}
```

## Netty

**零拷贝 内存池 Reactor**

http://cmsblogs.com/?p=2467

https://www.jianshu.com/p/ac7fb5c2640f

https://www.jianshu.com/p/a4e03835921a

Netty 是一款提供**异步非阻塞的、事件驱动的网络应用程序框架和工具**，用以快速开发高性能、高可靠性的网络服务器和客户端程序

也就是说，Netty 是一个基于 NIO 的客户、服务器端编程框架，使用 Netty 可以确保你快速和简单地开发出一个网络应用，例如实**现了某种协议的客户，服务端应用**。Netty 相当简化和流线化了网络应用的编程开发过程，例如，TCP 和 UDP 的 socket 服务开发。

### 特性

| 分类     | Netty的特性                                                  |
| -------- | ------------------------------------------------------------ |
| 设计     | 统一的API，支持多种传输类型，阻塞和非阻塞的  简单而强大的线程模型 真正的无连接数据报套接字支持 链接逻辑组件以支持复用 |
| 易于使用 | 详实的 Javadoc 和大量的示例集 不需要超过JdK 1.6+的依赖       |
| 性能     | 拥有比 Java 的核心 API 更高的吞吐量以及更低的延迟 得益于池化和复用，拥有更低的资源消耗 最少的内存复制 |
| 健壮性   | 不会因为慢速、快速或者超载的连接而导致 OutOfMemoryError  消除在高速网络中 NIO 应用程序常见的不公平读/写比率 |
| 安全性   | 完整的 SSL/TLS 以及 StartTLs 支持 可用于受限环境下，如 Applet 和 OSGI |
| 社区驱动 | 发布快速而且频繁                                             |

### Netty核心组件

- Channel
- ChannelFuture
- EventLoop
- ChannelHandler
- ChannelPipeline

#### Netty的ByteBuf缓冲区的种类

**DirectBuffer**

**HeapBuffer**

**PooledBuffer**

**UnPooledBuffer**

ByteBuf支持**堆缓冲区**和**堆外直接缓冲区**，根据经验来说，底层IO处理线程的缓冲区使用堆外直接缓冲区，减少一次IO复制。业务消息的编解码使用堆缓冲区，分配效率更高，而且不涉及到内核缓冲区的复制问题。

ByteBuf的**堆缓冲区**又分为**内存池缓冲区PooledByteBuf**和**普通内存缓冲区UnpooledHeapByteBuf**。

PooledByteBuf**采用完全二叉树**来实现一个内存池，集中管理内存的分配和释放，不用每次使用都新建一个缓冲区对象。UnpooledHeapByteBuf每次都会新建一个缓冲区对象。在**高并发**的情况下推荐使用PooledByteBuf，可以**节约内存的分配**。在性能能够保证的情况下，可以使用UnpooledHeapByteBuf，**实现比较简单**。

#### NIO中的ByteByffer的分类

1. HeapByteBuffer是基于堆上字节数组为存储结构的缓冲区。
2. DirectByteBuffer是基于直接内存上的内存区域为存储结构的缓冲区。DirectByteBuffer里**维护了一个引用address**指向了数据，从而操作数据。但由于申请效率低，因此**netty结合引用计数进行管理，即引用计数为0时，则将其会受到内存池，下一次申请时会进行复用。**

DirectByteBuffer**继承了MappedByteBuffer**，主要是实现了byte获得函数get等。由于**已经通过map0()函数返回内存文件映射的address，这样就无需调用read或write方法对文件进行读写**，通过address就能够操作文件。底层采用unsafe.getByte方法，通过（address + 偏移量）获取指定内存的数据。

![img](https://img-blog.csdn.net/20170607225947116?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlueGRjbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



1. MappedByteBuffer主要是文件操作相关的，它提供了一种**基于虚拟内存映射**的机制，使得我们可以像操作文件一样来操作文件，而不需要每次将内容更新到文件之中，同时读写效率非常高。

![img](https://img-blog.csdn.net/20170607224313512?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlueGRjbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



注：**外设之所以要把jvm堆里的数据copy出来再操作，不是因为操作系统不能直接操作jvm内存，而是因为jvm在进行gc（垃圾回收）时，会对数据进行移动，一旦出现这种问题，外设就会出现数据错乱的情况**

#### **传统IO问题：**

**1.线程资源受限**：线程是操作系统中非常宝贵的资源，同一时刻有大量的线程处于阻塞状态是非常严重的资源浪费，操作系统耗不起

**2.线程切换效率低下**：单机cpu核数固定，线程爆炸之后操作系统频繁进行线程切换，应用性能急剧下降。

3.数据读写是以**字节流为单位，效率不高**

解决上述三个问题，NIO的解决手段：

#### **NIO**



对于第一个问题，NIO采用多路复用器的手段，即所有连接注册到一个Selector上。

对于第二个问题，NIO线程数量大幅降低，因此切换效率较高。

对于第三个问题，NIO不是以**字节**为单位进行读取，而是**字节块**。IO模型中，每次都是从操作系统底层**一个字节一个字节**地读取数据，而NIO引入了 **Channel** 和 **Buffer** 的概念. 在 NIO 中, 我只能从 Channel 中读取数据到 Buffer 中或将数据从 Buffer 中写入到 Channel.维护一个**缓冲区**，每次可以从这个缓冲区里面读取一块的数据，



**JDK的NIO底层由epoll实现，**该实现饱受诟病的空轮训bug会导致cpu飙升100%

#### **Netty**

Netty是一个**异步事件驱动**的网络应用框架，用于快速开发可维护的高性能服务器和客户端。跟NIO一样，netty服务端代码也有**两个线程**，一个用来**接收新连接**，一个用来负责**读取数据**

##### 零拷贝

零拷贝”是指计算机操作的过程中，CPU不需要为数据在内存之间的拷贝消耗资源。而它通常是指计算机在网络上发送文件时，不需要将文件内容拷贝到用户空间（User Space）而直接在内核空间（Kernel Space）中传输到网络的方式。

##### **内存池**

http://www.cnblogs.com/wxd0108/p/6681623.html

**基于ThreadLocal的内存池技术**

![img](http://static.oschina.net/uploads/space/2016/0212/173950_fqm8_1759553.png)



随着JVM虚拟机和JIT即时编译技术的发展，对象的分配和回收是个非常轻量级的工作。但是对于缓冲区Buffer，情况却稍有不同，特别是对于**堆外直接内存的分配和回收**，是一件耗时的操作。而且这些实例随着消息的处理朝生夕灭，这就会给服务器带来沉重的GC压力，同时消耗大量的内存。为了尽量重用缓冲区，Netty提供了**基于内存池的缓冲区重用**机制。性能测试表明，采用内存池的ByteBuf相比于朝生夕灭的ByteBuf，性能高23倍左右

。

##### **Reactor模型**

https://segmentfault.com/a/1190000007282628

可以这样理解，Reactor就是一个执行while (true) { selector.select(); ...}循环的线程，会源源不断的产生新的事件，称作**反应堆**很贴切。
事件又分为**连接事件、IO读和IO写事件**，一般把连接事件单独放一线程里处理，即主Reactor（MainReactor，对应**Acceptor**），IO读和IO写事件放到另外的一组线程里处理，即从Reactor（SubReactor），从Reactor线程数量一般为2*(CPUs - 1)。
所以在运行时，MainReactor只处理Accept事件，连接到来，马上按照策略转发给从Reactor之一，只处理连接，故开销非常小；每个SubReactor管理多个连接，负责这些连接的读和写，属于IO密集型线程，读到完整的消息就丢给业务线程池处理业务，处理完比后，响应消息一般放到队列里，SubReactor会去处理队列，然后将消息写回。

Reactor 的线程模型有三种:

- **单线程模型**:即 **acceptor** 处理和 **handler** 处理都在一个线程中处理。**一个线程**(单线程)来处理**CONNECT**事件(Acceptor)和**转发Event**，一个线程池（多线程）来处理read,一个线程池（多线程）来处理write,那么从Reactor Thread到handler都是异步的，从而IO操作也多线程化。

  ![clipboard.png](https://segmentfault.com/img/bVFed3?w=2623&h=653)

  ![img](https://upload-images.jianshu.io/upload_images/7240015-7f11538ccf0f1c66?imageMogr2/auto-orient/strip%7CimageView2/2/w/613/format/webp)

- **多线程模型**:Reactor 的多线程模型与单线程模型的区别就是 **acceptor 是一个单独的线程处理**, 并且有一组特定的 NIO 线程来负责各个客户端连接的 IO 操作. Reactor 多线程模型如下:

  通过Reactor Thread Pool来提高**event的分发能力**

  ![clipboard.png](https://segmentfault.com/img/bVFed5?w=2748&h=600)

  ![img](https://upload-images.jianshu.io/upload_images/7240015-7741efe3f0c0b068?imageMogr2/auto-orient/strip%7CimageView2/2/w/640/format/webp)

- **主从多线程模型**

- 如果我们的服务器需要同时处理大量的客户端连接请求或我们需要在客户端连接时, 进行一些权限的检查, 那么单线程的 Acceptor 很有可能就处理不过来, 造成了大量的客户端不能连接到服务器.
  Reactor 的**主从多线程模型**就是在这样的情况下提出来的, 它的特点是: 服务器端接收客户端的连接请求不再是一个线程, 而是主Reactor（MainReactor，对应**Acceptor**），IO读和IO写事件放到另外的一组线程里处理，即从Reactor（SubReactor），从Reactor线程数量一般为2*(CPUs - 1)。 它的线程模型如下:

- 如果我们的服务器需要同时处理大量的客户端连接请求或我们需要在客户端连接时, 进行一些权限的检查, 那么单线程的 Acceptor 很有可能就处理不过来, 造成了大量的客户端不能连接到服务器.
  Reactor 的主从多线程模型就是在这样的情况下提出来的, 它的特点是: 服务器端接收客户端的连接请求不再是一个线程, 而是由一个独立的线程池组成. 它的线程模型如下:

- ![clipboard.png](https://segmentfault.com/img/bVFed4?w=3198&h=600)

- 

- 

- 

  ![img](https://upload-images.jianshu.io/upload_images/7240015-e2a4ee766f05c98c?imageMogr2/auto-orient/strip%7CimageView2/2/w/556/format/webp)

  **为什么要单独分一个Reactor来处理监听呢？**因为像TCP这样需要经过**3次握手**才能建立连接，这个建立连接的过程也是要**耗时间和资源**的，单独分一个Reactor来处理，**可以提高性能**。

