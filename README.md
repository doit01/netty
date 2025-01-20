WebSocket本身是依赖tomcat,然而tomcat的并发量不大，连接数低，会导致出现断连的情况，因此对于WebSocket通信要求不高的，可以直接依赖tomcat。但是遇到高并发就难以支撑，这时候，netty该上场了

netty用于
高并发服务器：用于构建需要处理大量并发连接的服务器，如聊天服务器、游戏服务器等。
2. 代理服务器和网关：作为中间层，用于代理请求、负载均衡和安全控制。
3. RPC 框架：用于构建远程过程调用（RPC）系统，提升分布式系统的通信效率。
4. 自定义协议实现：用于实现自定义的通信协议，例如专用的传输协议或应用层协议。


                        
原文链接：https://blog.csdn.net/xyy1028/article/details/139527648
Rsocket > websocket
 RSocket (the application protocol) targets WebSockets, TCP, and Aeron, and is expected to be usable over any transport layer with TCP-like characteristics, such as QUIC.
    Netty‌：Netty通过高效的NIO机制实现了高并发处理，适用于需要处理大量并发连接和高数据吞吐量的场景‌23。 应用场景‌：Netty适用于服务器端应用、聊天服务器、游戏服务器等需要高性能网络通信的场景‌3。

‌适用场景对比‌：

    ‌RSocket‌：由于其支持多种交互模式和响应式编程的特性，RSocket更适合需要复杂交互和响应式处理的场景，如微服务架构中的服务间通信、实时数据流处理
    
Perhaps more importantly though, it makes TCP, WebSockets, and Aeron usable without significant effort. For example, use of WebSockets is often appealing, but all it exposes is framing semantics, so using it requires the definition of an application protocol. This is generally overwhelming and requires a lot of effort. TCP doesn’t even provide framing. Thus, most applications end up using HTTP/1.1 and sticking to request/response and missing out on the benefits of interaction models beyond synchronous request/response.

For this reason, RSocket defines application-layer semantics over these network transports to allow choosing them when they are appropriate. Later in this document is a brief comparison with other protocols that were explored while trying to leverage WebSockets and Aeron before determining that a new application protocol was wanted.



当web页面聊天应用时候，用 undertow的websocket方法即可。
当摄像头发信息到服务端时候可以用netty 。

netty 是java的客户端和服务端网络通信的 应用框架。java 实现的客户端和服务端的时候 用netty来通信。
它简化了 TCP 和 UDP 客户端/服务器系统的编程，支持http ，websocket协议。即创建TCP UDP ,WEBSOCKET 的服务端和客户端
在web应用上的使用示例
WebSocket是一种在单个TCP连接上进行全双工通讯的协议。Netty提供了对WebSocket的支持，可以实现高效的实时通信
public class WebSocketServer {
    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new HttpServerCodec());
                        ch.pipeline().addLast(new HttpObjectAggregator(65536));
                        ch.pipeline().addLast(new WebSocketFrameHandler());
                    }
                });
            ChannelFuture f = b.bind(8080).sync();
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
}
 HTTP服务器
 Netty也可以用来实现高性能的HTTP服务器
 public class HttpServer {
    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new HttpServerCodec());
                        ch.pipeline().addLast(new HttpObjectAggregator(65536));
                        ch.pipeline().addLast(new HttpRequestHandler());
                    }
                });
            ChannelFuture f = b.bind(8080).sync();
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
}




nio中的网络通道是非阻塞Io实现是基于事件驱动的，适用于那些即时通信的服务，相比java socket服务,一个客户端一个连接，或者线程连接池的开销，
new io使用非阻塞方式处理，可以一个线程处理大量客户端连接请求。nio种定义了一些网络事件，比如读写事件连接事件等，selector选择器会不停的检测所有的客户端连接，有事件发生才会针对每个事件响应处理，这样一个单线程管理多个通道连接，不必为每个连接建立线程，避免多线程的上下文切换开销
Netty 适用的场景包括：
    服务器与客户端之间的通信
    内部系统间的通信
    大规模分布式系统
    异步通信
    实时系统
    基于 TCP 的协议
    基于 HTTP 的协议
    io.netty.codec.dns
io.netty.codec.haproxy
io.netty.codec.http
io.netty.codec.http2
io.netty.codec.memcache
io.netty.codec.mqtt
io.netty.codec.redis
io.netty.codec.smtp
io.netty.codec.socks
io.netty.codec.stomp





而websocket是一个客户端一个连接，高效传输，解决轮询，双向通信。



volatile仅仅解决了可见性，不能保证互斥性，多线程并发修改某个变量时仍然产生多线程问题。他适合一个线程写，多个线程读的场景
jdk的bytebuffer长度固定，只有一个位置指针，flip容易出错
bytebuf两个位置指针，读用readerindex，写用writerindex，自动扩展长度。不用关心底层的校验。
unpooleadHeapByteBuf基于堆内存进行内存分配的字节缓冲区。没有对象池技术实现。每次io都会创建一个新的unpooleadHeapByteBuf
满足性能时候推荐使用。不容易出现内存管理问题。

NioEventLoopGroup 的两个实例，分别是 bossGroup（acceptor） 和 workerGroup（i/o 读写线程池）
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

