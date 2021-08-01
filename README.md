NioEventLoopGroup 的两个实例，分别是 bossGroup 和 workerGroup
是两个线程池, 它们默认线程数为 CPU 核心数乘以 2，bossGroup 用于接收客户端传过来的请求，接收到请求后将后续操作交由 workerGroup 处理。
etty 线程模型是典型的 Reactor 模型结构，其中常用的 Reactor 线程模型有三种，分别为：Reactor 单线程模型、Reactor 多线程模型和主从 Reactor 多线程模型。

而在 Netty 的线程模型并非固定不变，通过在启动辅助类中创建不同的 EventLoopGroup 实例并通过适当的参数配置，就可以支持上述三种 Reactor 线程模型。
Reactor 单线程模型

Reactor 单线程模型指的是所有的 IO 操作都在同一个 NIO 线程上面完成。作为 NIO 服务端接收客户端的 TCP 连接，作为 NIO 客户端向服务端发起 TCP 连接，读取通信对端的请求或向通信对端发送消息请求或者应答消息。

由于 Reactor 模式使用的是异步非阻塞 IO，所有的 IO 操作都不会导致阻塞，理论上一个线程可以独立处理所有 IO 相关的操作
![图片](https://user-images.githubusercontent.com/9653509/127757865-071d0085-4010-4bb6-8549-b8466986a535.png)
Netty 使用单线程模型的的方式如下：

EventLoopGroup bossGroup = new NioEventLoopGroup(1);
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup)
 .channel(NioServerSocketChannel.class)
...

 

在实例化 NioEventLoopGroup 时，构造器参数是 1，表示 NioEventLoopGroup 的线程池大小是 1。然后接着我们调用 b.group(bossGroup) 设置了服务器端的 EventLoopGroup，因此 bossGroup和 workerGroup 就是同一个 NioEventLoopGroup 了。


Reactor 多线程模型

对于一些小容量应用场景，可以使用单线程模型，但是对于高负载、大并发的应用却不合适，需要对该模型进行改进，演进为 Reactor 多线程模型。

Rector 多线程模型与单线程模型最大的区别就是有一组 NIO 线程处理 IO 操作。

在该模型中有专门一个 NIO 线程 -Acceptor 线程用于监听服务端，接收客户端的 TCP 连接请求；而 1 个 NIO 线程可以同时处理N条链路，但是 1 个链路只对应 1 个 NIO 线程，防止发生并发操作问题。

网络 IO 操作-读、写等由一个 NIO 线程池负责，线程池可以采用标准的 JDK 线程池实现，它包含一个任务队列和 N 个可用的线程，由这些 NIO 线程负责消息的读取、解码、编码和发送。
![图片](https://user-images.githubusercontent.com/9653509/127757909-eb45e482-42ec-45e1-92fb-24fbb976af2c.png)
Netty 中实现多线程模型的方式如下：
复制代码

EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
 .channel(NioServerSocketChannel.class)
 ...


bossGroup 中只有一个线程，而 workerGroup 中的线程是 CPU 核心数乘以 2，那么就对应 Recator 的多线程模型。




主从 Reactor 多线程模型

在并发极高的情况单独一个 Acceptor 线程可能会存在性能不足问题，为了解决性能问题，产生主从 Reactor 多线程模型。

主从 Reactor 线程模型的特点是：服务端用于接收客户端连接的不再是 1 个单独的 NIO 线程，而是一个独立的 NIO 线程池。

Acceptor 接收到客户端 TCP 连接请求处理完成后，将新创建的 SocketChannel 注册到 IO 线程池（sub reactor 线程池）的某个 IO 线程上，由它负责 SocketChannel 的读写和编解码工作。

Acceptor 线程池仅仅只用于客户端的登陆、握手和安全认证，一旦链路建立成功，就将链路注册到后端 subReactor 线程池的 IO 线程上，由 IO 线程负责后续的 IO 操作。
EventLoopGroup bossGroup = new NioEventLoopGroup(4);
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
 .channel(NioServerSocketChannel.class)
 ...

复制代码

 

但是，在 Netty 的服务器端的 acceptor 阶段，没有使用到多线程, 因此上面的主从多线程模型在 Netty 的实现是有误的。

服务器端的 ServerSocketChannel 只绑定到了 bossGroup 中的一个线程，因此在调用 Java NIO 的 Selector.select 处理客户端的连接请求时，实际上是在一个线程中的，所以对只有一个服务的应用来说，bossGroup 设置多个线程是没有什么作用的，反而还会造成资源浪费。
最后小结一下：(如果你还没明白，可以看一下群里面的视频解析)

    NioEventLoopGroup 实际上就是个线程池，一个 EventLoopGroup 包含一个或者多个 EventLoop；
    一个 EventLoop 在它的生命周期内只和一个 Thread 绑定；
    所有有 EnventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理；
    一个 Channel 在它的生命周期内只注册于一个 EventLoop；
    每一个 EventLoop 负责处理一个或多个 Channel；

