[TOC]

# Netty 入门

## 1. 概述

### Netty是什么？
Netty 是一个**异步**的，**基于事件驱动**的**网络应用框架**，用于快速开发可维护，高性能的网络服务端和客户端

### Netty的地位
Netty在Java 网络应用框架中的地位就好比：Spring 框架在JavaEE开发中的地位

以下的框架都使用了Netty，因为它们有网络通信需求！
* Cassandra-nosql 数据库
* Spark-大数据分布式计算框架
* Hadoop-大数据分布式存储框架
* RocketMQ-ali 开源的消息队列
* ElasticSearch -搜索引擎
* gRPC-rpc框架
* Dubbo-rpc框架
* Zookeeper - 分布式协调框架

### Netty的优势
* NIO 的类库和API 繁杂，需要熟练掌握Selector， ServerSocketChannel，SocketChannel，ByteBuff等，使用非常麻烦。且开发工作量和难度都非常大：例如客户端面临断线重连，网络闪断，心跳处理，半包读写，网络拥塞和异常流的处理等等。
* **Netty 对 JDK自带的NIO的API 进行了良好的封装**，解决了上述问题。且Netty拥有高性能，吞吐量更高，延迟更低，减少资源消耗，**最小化不必要的内存复制**等优点。



## Netty 线程模型

### 1. 线程模型架构图

![](2021-04-16-13-05-49.png)

对比NIO 的线程模型，NIO的模型其实就一个 **Selector 同时 需要处理连接事件和读写事件** 。如果读写事件太多造成积压，这就造成连接事件得不到及时处理。一种升级的模型其实可以理解成有**两个 Selector**，一个专门用于处理连接事件，一个专门用于处理读写事件。所以一个 Selector 连接建立后，获取**通道并注册到另一个 Selector 上**，专门用于读写事件，这样就舒服多了。这种就可以成**为一主一从**结构。

其实Netty一般情况是**一主多从**架构。可以**设置有多个Selector**。 也可以是**多主多从**架构。



### 2. 线程模型解释
Netty 抽象出两组线程池 **BossGroup** 和 **WorkerGroup**，**BossGroup** 专门接收客户端的连接， **WorkerGroup** 专门负责网络的编写。

BossGroup和WorkerGroup 类型都是 NioEventLoopGroup。 NioEventGroup 相当于一个**事件循环线程组**。这个组中含有多个事件循环线程，每一个事件循环线程是 NioEventLoop。一个NioEventLoop **可以理解为一个线程**。 每个NioEventLoop可以理解为**一个线程**。每个NioEventLoop都有一个selector， 相当于要给线程就有一个Selector，用于监听注册在其上的socketChannel 的网络通讯。

每个BossNioEventLoop线程内部循环执行的步骤有3步：
* 处理 ==accept事件==，与client 建立连接，生成**NioSocketChannel通道**。
* **将NioSocketChannel** 注册到某个Worker NioEventLoop 的 **selector**之上。
* 处理任务队列的任务，即runAllTasks。


每个Worker NioEventLoop 线程内部循环执行的步骤有3步：
* **轮询注册**到自己Selector上的所有 NioSocketChannel 的**read，write**事件(只处理读写事件)。
* 处理I/O事件，即read，write事件，并在对应NioSocketChannel 处理业务。
* runAllTasks 处理 任务队列 TaskQueue 的任务，一些耗时的业务处理一般可以放入TaskQueue中慢慢处理，这样不影响数据在pipeline中的流动处理。

注意： 一个NioEventLoop就是一个**线程池**，只不过里面只有**一个**线程，**每个线程**都有一个TaskQueue ，用于执行比较耗时的任务。



## Netty模块组件
### 1. Bootstrap， ServerBootstrap

Bootstrap 意思是**引导类**，一个Netty应用通常由一个Bootstrap开始，主要作用是 **配置整个Netty程序并串联各个组件**。 Bootstrap类 是客户端引导类，ServerBootstrap 是**服务端启动引导类**。


### 2. Future, ChannelFuture
Netty 中 所有的IO操作都是**异步**的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过**Future和ChannelFutures** 注册一个监听，当**操作执行成功或失败监听会自动触发注册的监听事件**。

* 注意，这里的Future 与 jdk中的Future同名，但是是两个接口
  * jdk Future 只能同步等待任务结束(获成功，或失败)才能得到结果。
  * netty Future 可以同步等待任务结束得到结果，也可以异步方式得到结果，但都是要等任务结束

### 3. Channel
Netty 网络通信的组件，能够用于执行网络I/O操作。Channel 为用户提供：
* 当前网络连接的**通道的状态**(例如是否打开?是否已连接?)
* 网络连接的**配置参数**(例如接收缓冲区大小）
* 提供异步的 **网络I/O操作** (如建立连接，读写，绑定窗口)，异步调用意味着任何I/O 调用都将立即返回，并且不保证在调用结束锁清秋的I/O操作已完成。
* 调用立即返回一个ChannelFuture 实例， 通过注册监听器到ChannelTuture上，可以I/O操作成功，失败或取消时回调通知调用放。
* 支持关联I/O操作与对应的处理程序。

**不同协议、不同的阻塞类型的连接都有不同的 Channel 类型与之对应。**

下面是一些常用的 Channel 类型：
```java
NioSocketChannel // 异步的客户端 TCP Socket连接
NioServerSocketChannel 	// 异步的服务器端TCP Socket连接
NioDatagramChannel		// 异步的UDP连接
NioSctpChannel	// 异步的客户端 Sctp 连接
NioSctpServerChannel	// 异步的Sctp服务器端连接，这些通道涵盖了UDP和TCP网络IO以及文件IO
```

channel 的主要作用：
* close()可以用来关闭channel
* closeFuture()可以用来处理channel关闭
  * sync 方法作用使同步等待channel关闭
  * 而addListener 方法是异步等待channel关闭
* pipeline() 方法添加处理器
* write()方法将数据写入
* writeAndFlush()方法将数据写入并刷出

### Future & Promise
在异步处理时，经常用到这两个接口
首先在这里说明，netty中的Future 与jdk中的Future 同名，但是是两个接口，netty的Future 继承自jdk的Future，而Promise 又对netty Future进行了扩展
* jdk Future 只能同步等待结束(才能得到结果)
* netty Future 可以同步等待任务结束(get())得到结果，也可以异步方式(addListerner)得到结果，但都是要等任务结束
![](2021-04-16-19-38-46.png)
![](2021-04-16-19-40-45.png)
* netty Promise 不仅有netty Future 的功能，而且脱离了任务独立存在，只作为两个线程传递结果的容器
![](2021-04-16-19-41-15.png)
### 4. Selector
Netty 基于 **Selector 对象实现I/O多路复用**，通过**Selector 一个线程可以监听多个连接的Channel 事件**， 当向一个Selector 中**注册**Channel 后，Selector内部机制可以**自动不断地查询**(select)是否有已就绪的I/O事件(例如可读，可写，网络连接完成等)，这样程序就可以很简单地使用一个线程高效地管理多个Channel。


### 5. NioEventLoop
NioEventLoop维护了**一个线程和任务队列**，支持异步提交执行任务，线程启动会调用NioEventLoop地run**方法**，执行I/O任务和非I/O任务：
* **IO 任务**， 即selectionKey 中 ready 地事件，如 accept，connect，read,write等，由 processSelectedKsys 方法触发。
* **非IO 任务**， 添加到 taskQueue 中的任务，如 register(),bind()等任务，由runAllTasks **方法触发**。


### 6. NioEventLoopGroup
NioEventLoopGroup主要管理EventLoop的**生命周期**， 可以理解为 一个**线程池**，内部维护了**一组线程**，每个线程(NioEventLoop)负责处理多个Channel 上的事件， 而一个 Channel值对应于一个线程。


### 7. ChannelHandler 
ChannelHandler是一个**接口** ，**处理I/O事件或拦截I/O操作**，并将其转发到其ChannelPipeline(业务处理链)中的下一个处理程序。
```java
ChannelInboundHandler 	// 用于处理入站I/O事件
ChannelOutboundHandler 	// 用于处理出站I/O操作
```
或者使用以下**适配器**类：
```java
ChannelInboundHandlerAdapter  // 用于处理入站I/O事件
ChannelOutboundHandlerAdapter // 用于处理出站I/O操作
```
### 8. ChannelPipline
用于**保存**ChannelHandler 实现类的List，用于**处理和拦截Channel 的入站事件和出站操作**。 ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式， 以及 Channel各个的ChannelHandler 如何**相互交互**。

在Netty 中每个 Channel都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下：
![](2021-04-16-14-57-29.png)

一个Channel 包含了一个 ChannelPipleline，而ChannelPipeline中又维护了一个由 ChannelHandlerContext组成的双向链表，并且每个ChannelHandlerContext中又关联着一个ChannelHandler。

**read** 事件(入站事件)和 **write**事件(出站事件)在一个**双向链表**中，**入站**事件会从链表head **往后传递**到最后一个入站的handler，出站事件会从链表tail**往前传递**到最前一个出站的handler，**两种类型的handler 互不干扰**。
#### Handler & Pipeline
ChannelHandler 用来处理Channel上的各种事件，分为入站，出站两种。所有CHannelHandler 被连成一串，就是Pipeline

* 入站处理器通常是 ChannelInboundHandlerAdapter 的子类，主要用来读取客户端数据，写回结果
* 出站处理器通常是 ChannelOutboundHandlerAdapter 的子类，主要对写回结果进行加工

##### 服务端
```java
new ServerBootstrap()
    .group(new NioEventLoopGroup())
    .channel(NioServerSocketChannel.class)
    .childHandler(new ChannelInitializer<NioSocketChannel>() {
        protected void initChannel(NioSocketChannel ch) {
            ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) {
                    System.out.println(1);
                    ctx.fireChannelRead(msg); // 1
                }
            });
            ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) {
                    System.out.println(2);
                    ctx.fireChannelRead(msg); // 2
                }
            });
            ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) {
                    System.out.println(3);
                    ctx.channel().write(msg); // 3
                }
            });
            ch.pipeline().addLast(new ChannelOutboundHandlerAdapter(){
                @Override
                public void write(ChannelHandlerContext ctx, Object msg, 
                                  ChannelPromise promise) {
                    System.out.println(4);
                    ctx.write(msg, promise); // 4
                }
            });
            ch.pipeline().addLast(new ChannelOutboundHandlerAdapter(){
                @Override
                public void write(ChannelHandlerContext ctx, Object msg, 
                                  ChannelPromise promise) {
                    System.out.println(5);
                    ctx.write(msg, promise); // 5
                }
            });
            ch.pipeline().addLast(new ChannelOutboundHandlerAdapter(){
                @Override
                public void write(ChannelHandlerContext ctx, Object msg, 
                                  ChannelPromise promise) {
                    System.out.println(6);
                    ctx.write(msg, promise); // 6
                }
            });
        }
    })
    .bind(8080);
```
##### 客户端
```java
new Bootstrap()
    .group(new NioEventLoopGroup())
    .channel(NioSocketChannel.class)
    .handler(new ChannelInitializer<Channel>() {
        @Override
        protected void initChannel(Channel ch) {
            ch.pipeline().addLast(new StringEncoder());
        }
    })
    .connect("127.0.0.1", 8080)
    .addListener((ChannelFutureListener) future -> {
        future.channel().writeAndFlush("hello,world");
    });

```
服务器端打印：
```
1
2
3
6
5
4
```

可以看到，ChannelInboundHandlerAdapter 是按照addLast的顺序执行的，
ChannelOutboundHandlerAdapter是按照addLast的逆序执行的。而ChanelPipeline 的实现是一个 ChannelHandlerContext(包装了ChannelHandler)组成的双向链表

![](2021-04-16-21-11-22.png)

* 入站处理器中,ctx.fireChannelRead(msg)是调用下一个入站处理器
  * 如果注释掉1处代码，则仅会打印1
  * 如果注释掉2处代码，则仅会打印1，2
* 3处的ctx.channel.write(msg) 会 从尾部开始触发  后续出站处理器的执行
  * 如果注释掉3处代码，则仅会打印 1，2，3
* 类似的，出站处理器， ctx.write(msg,promise)的调用也会触发上一个出站处理器
  * 如果注释掉6处代码，则仅会打印1 2 3 6

* ctx.channel().write(msg) vs  ctx.write(msg)
    * 都是触发出战处理器的执行
    * ctx.channel().write(msg)从尾部开始查找出站处理器
    * ctx.write(msg)是从当前节点找上一个出站处理器。
    * 3处的 ctx.channel().write(msg)如果改为 ctx.write(msg)仅会打印 1 2 3，因为节点3 之前没有其他出站处理器了
    * 6处的 ctx.write(msg,promise)如果改为ctx.channel.write(msg)会打印 1 2 3 6 6 6 ... 因为一直从尾部找



### 9. ChannelHandlerContext
保存Channel相关的所有上下文信息，同时关联一个ChannelHandler 对象。



### ByteBuf

从结构上来说，ByteBuf由一串**字节数组**构成，数组中每个**字节用来存放信息**

####1) 创建
```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(10);
log(buffer);
```
上面代码创建了一个默认的ByteBuf(池化基于直接内存的ByteBuf)，初始容量是10

输出
```java
read index:0 write index:0 capacity:10
```

其中log 方法参考如下
```java
private static void log(ByteBuf buffer) {
    int length = buffer.readableBytes();
    int rows = length / 16 + (length % 15 == 0 ? 0 : 1) + 4;
    StringBuilder buf = new StringBuilder(rows * 80 * 2)
        .append("read index:").append(buffer.readerIndex())
        .append(" write index:").append(buffer.writerIndex())
        .append(" capacity:").append(buffer.capacity())
        .append(NEWLINE);
    appendPrettyHexDump(buf, buffer);
    System.out.println(buf.toString());
}
```

####2) 直接内存vs 堆内存
可以使用下面的代码来创建池化基于堆的ByteBuf
```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.heapBuffer(10);
```

也可以使用下面的代码来创建池化基于直接内存的ByteBuf
```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.directBuffer(10);
```
* 直接内存创建和销毁的代价昂贵，但读写性能高(少一次内存复制)，适合配合池化功能一起用
* 直接内存对GC压力小，因为这部分内存不受JVM垃圾回收的管理，但也要注意及时主动释放
* 直接内存的初始分配较慢；没有JVM直接帮助管理内存，容易发生内存溢出
* 为了避免一直没有FUll gc，导致直接内存把物理内存被消耗完。

#### 3）池化 vs 非池化

池化的最大意义在于可以重用ByteBuf,优点有
* 没有池化，则每次都得创建新的ByteBuf实例，这个操作对直接内存代价昂贵，就算是堆内存，也会增加GC压力
* 有了池化，则可以重用池中ByteBuf 实例，并且采用了与jemalloc类似的内存分配算法提升分配效率
* 高并发时，池化功能更节约内存，减少内存溢出的可能

池化功能是否开启，可以通过下面的系统环境变量来设置
```java
-Dio.netty.allocator.type={unpooled|pooled}
```

#### 4)组成
ByteBuf由四部分组成：
![](2021-04-17-15-27-51.png)

ByteBuf 提供了两个索引，一个用于读取数据，一个用于写入数据。这两个索引通过在**字节数组中移动**，来定位需要读或者写信息的位置。

当从ByteBuf读取时，它的readerIndex (读索引) 将会根据读取的字节数递增。

当写ByteBuf时，它的writeIndex 也会根据写入的字节数进行递增。

而如果readerIndex 超过了writerIndex 的时候，Netty 会抛出IndexOutOfBoundsException异常。


方法列表
![](2021-04-17-15-33-56.png)



#### 零拷贝

Netty 的接收和发送ByteBuffer 采用  DIRECT BUFFERS, 使用 堆外 **直接内存** 进行Socket 读写， **不需要进行字节缓冲区的二次拷贝**。 使用直接内存 使得**数据可以不用拷贝到JVM内存中**,便可以直接写入Socket 中，这样就减少了拷贝的次数，这就是**零拷贝**。


如果使用传统的 JVM堆内存(HEAP BUFFERS)进行Socket 读写，JVM 会将堆内存中的数据拷贝一份到直接内存中，然后才能写入Socket中。 JVM堆内存的数据是**不能直接写入** Socket中的。相比于堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝。 JVM 对操作系统来说就是用户态，所以零拷贝只是不用把IO放在内核态的数据再拷贝到引用进程的内存中。

1） slice
零拷贝的体现之一，对原始 ByteBuf 进行切片成多个 ByteBuf，切片后的 ByteBuf 并没有发生内存复制，还是使用原始 ByteBuf 的内存，切片后的 ByteBuf 维护独立的 read，write 指针
![](2021-04-17-16-51-00.png)

2） duplicate
零拷贝的体现之一，就好比截取了原始 ByteBuf 所有内容，并且没有 max capacity 的限制，也是与原始 ByteBuf 使用同一块底层内存，只是读写指针是独立的


#### ByteBuf 优势
* 池化-可以重用池中的ByteBuf实例，更节约内存，减少内存溢出的可能
* 读写分离，不需要像ByteBuffer 一样切换读写模式
* 可以自动扩容
* 支持链式调用，使用更流畅
* 很多地方体现零拷贝，例如slice，duplicate，CompositeByteBuf



## 双向通信，Netty 服务器返回 客户端的内容

服务器
```java
public class NettyServer {
    public static void main(String[] args) {
//        System.out.println(Runtime.getRuntime().availableProcessors());

        EventLoopGroup bossGroup=new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup(8);


        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap.group(bossGroup, workerGroup);
        serverBootstrap.channel(NioServerSocketChannel.class);
        serverBootstrap.childHandler(new ChannelInitializer<NioSocketChannel>() {
            @Override
            protected void initChannel(NioSocketChannel ch)  {
                ch.pipeline().addLast(new ChannelInboundHandlerAdapter() {
                    @Override
                    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

                        ByteBuf byteBuffer = (ByteBuf) msg;
                        System.out.println(byteBuffer.toString(Charset.defaultCharset()));

                        ByteBuf responce = ctx.alloc().buffer();

                        responce.writeBytes(byteBuffer);
                        ctx.writeAndFlush(responce);


                    }
                });

            }
        });
        serverBootstrap.bind(8088);


    }

}

```

客户端
```java

public class NettyClient {
    public static void main(String[] args) throws InterruptedException {

        NioEventLoopGroup group = new NioEventLoopGroup();
        Channel channel = new Bootstrap()
                .group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new StringEncoder());
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) {
                                ByteBuf buffer = (ByteBuf) msg;
                                System.out.println(buffer.toString(Charset.defaultCharset()));

                                // 思考：需要释放 buffer 吗
                            }
                        });
                    }
                }).connect("127.0.0.1", 8088).sync().channel();

        channel.closeFuture().addListener(future -> {
            group.shutdownGracefully();
        });

        new Thread(() -> {
            Scanner scanner = new Scanner(System.in);
            while (true) {
                String line = scanner.nextLine();
                if ("q".equals(line)) {
                    channel.close();
                    break;
                }
                channel.writeAndFlush(line);
            }
        }).start();

    }
}
```



