dubbo：RPC框架

<https://blog.csdn.net/kingcat666/article/details/78577079>

RPC:远程过程调用协议，基于Tcp协议。

![img](https://gss3.bdstatic.com/7Po3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=88d234b10b46f21fdd395601974d0005/18d8bc3eb13533fadd93e964a9d3fd1f41345b56.jpg)

更底层原理：**动态代理**

客户端会生成一个RPCProxy代理，用来模拟方法调用，在RPCInvoker中通过TCP的Channel连接进行通信获得数据。

RPC 服务方通过 RpcServer 去导出（export）远程接口方法，而客户方通过 RpcClient 去引入（import）远程接口方法。客户方像调用本地方法一样去调用远程接口方法，**RPC 框架提供接口的代理实现**，实际的调用将委托给代理**RpcProxy** 。代理封装调用信息**并将调用转交给RpcInvoker 去实际执行。在客户端的RpcInvoker 通过连接器RpcConnector 去维持与服务端的通道RpcChannel，并使用RpcProtocol 执行协议编码（encode）并将编码后的请求消息通过通道发送给服务方。**

#### RPC结构

RPC架构主要分为四种组件：

**Client**

**Server:**

**Client Stub**:存放服务端的地址消息，再将客户端的请求接口、方法和参数按照约定协议打包成网络消息，然后通过网络远程发送给服务方的实例

**Server Stub**:服务端实例接收客户端发送过来的消息，将消息解包，并调用本地的方法。

为什么用RPC而不用HTTP呢？

因为需要维护长连接，HTTP三次握手效率太低，减少网络开销。而且RPC都有注册中心，有着丰富的监控管理。

#### 序列化方式

使用不同的序列化编码，如java原生序列化（ObjectOutputStream..）xml，json序列化（将对象转为json串转为byte数组，常用fastJson）或者二进制。

