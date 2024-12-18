---
title: NettySeries003 Netty基础JAVA NIO
date: 2024-11-26 11:47:01
mathjax: true
categories:
- [网络编程,netty]
tags:
- 原理
- 实现
description: "netty系列基础部分：JAVA NIO <br>
- JAVA网络编程方法"
---

## 背景

netty本质上是一个网络IO框架，对JAVA网络请求做了封装。要了解netty实现的功能，需要先对JAVA的网络功能做一定的了解。

## 功能

### IO模型

IO(Input/Output)是指数据在内存和外部设备的传输过程，常见的如文件操作，例如从文件读取内容(Input)和将内容写入文件(Output),更一般的，IO指代某个任务需要等待某个条件的执行完成来进行下一步操作，例如这里的文件读操作，从cpu发出读指令开始，由于cpu速度比文件设备快得多，cpu需等待文件设备将数据从文件设备复制到内存指定位置，直到文件设备复制完成，读操作就完成。IO不仅适用于文件操作，网络数据读写也适用。

这里涉及两个问题，由于cpu速度比文件设备快，那么cpu在文件设备准备数据的过程中，处于什么状态呢？第二个问题是数据准备完成后，cpu怎么感知到数据已经准备好。

前一个问题称为阻塞问题，如果cpu发出读取指令后，在数据没有准备好之前，cpu可以什么也不干，阻塞着等待数据完成，则称为阻塞；cpu也可以转而去处理其他任务，则称为非阻塞。后一个问题称为同步问题，如果cpu去主动查询数据是否准备好，则称为同步；如果cpu等待数据完成后通知到cpu，则称为异步。

基于上述两种方式以及实际情况，共分为以下几类IO模型：

1. 阻塞式IO模型
{% asset_img nettySeries003_001.png 阻塞式IO模型%}
阻塞式IO模型的特点是任务会一直阻塞等待直到数据完成。

2.非阻塞式IO模型
{% asset_img nettySeries003_002.png 非阻塞式IO模型%}
相比阻塞模型，此时任务不会阻塞在等待数据，任务可以执行其他的操作，然后定时去轮询数据是否准备好。

3.I/O复用模型
{% asset_img nettySeries003_003.png IO复用模型%}
IO复用模型多了一个概念：select，select类似于一个IO操作集合体，多个IO操作可以阻塞在一个select上，当select中有至少一个IO数据准备好时，select返回。select本质上也是阻塞IO模型，只不过阻塞在select上的是多个IO操作。

4.异步IO模型
{% asset_img nettySeries003_004.png 异步IO模型%}
异步IO模型则是任务在数据准备完成后得到通知。

前三种IO模型都是同步IO模型，依赖任务去查询数据是否准备好，第四种则是异步模型。

### JAVA NIO

了解了IO模型之后，再来看一下JAVA NIO.JAVA NIO采用了IO复用模型，共有三个核心组件：

+ Channels
+ Buffers
+ Selectors

Channel是连接的抽象，表示一个连接，连接的对象可以是设备、文件、socket等。在传统客户/服务器模型中，Channel表示客户与服务器建立的一个连接。

Buffer则是数据的载体，所有数据的写入与读取都是通过Buffer写入Channel，或者从Channel读入Buffer。

Selector则类似于IO复用模型中的select，多个Channel注册到selector中，当任意一个Channel有数据时，selector都能从阻塞中返回。

{% asset_img nettySeries003_005.png NIO%}
三个核心组件串联起来，大致构成了上面的IO模型，所有的Channel都注册在selector上，包括表示服务端监听的ServerSocketChannel，当有客户端连接过来时，ServerSocketChannel从select返回，并生成表示客户端连接的SocketChannel，该Channel也注册到selector上，当SocketChannel有数据过来时，也从select返回。

结合上面的说明，再来看JAVA NIO的例子。

```JAVA .{line-numbers}
//code segment :java_nio_num1
public class NBTimeServer {
    private static final int DEFAULT_TIME_PORT = 8900;

    // Constructor with no arguments creates a time server on default port.
    public NBTimeServer() throws Exception {
        acceptConnections(this.DEFAULT_TIME_PORT);
    }

    // Constructor with port argument creates a time server on specified port.
    public NBTimeServer(int port) throws Exception {
        acceptConnections(port);
    }

    // Accept connections for current time. Lazy Exception thrown.
    private static void acceptConnections(int port) throws Exception {
        // Selector for incoming time requests
        Selector acceptSelector = SelectorProvider.provider().openSelector();

        //创建一个ServerSocketChannel并设置为非阻塞模型
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);

        //绑定端口和地址
        InetAddress lh = InetAddress.getLocalHost();
        InetSocketAddress isa = new InetSocketAddress(lh, port);
        ssc.socket().bind(isa);

        //将serverSocketChannel注册到selector，从而实现IO复用
        SelectionKey acceptKey = ssc.register(acceptSelector,
            SelectionKey.OP_ACCEPT);

        int keysAdded = 0;

        //当注册到selector中的channel有数据准备时，select返回
        while ((keysAdded = acceptSelector.select()) > 0) {
            // Someone is ready for I/O, get the ready keys
            Set<SelectionKey> readyKeys = acceptSelector.selectedKeys();
            Iterator<SelectionKey> i = readyKeys.iterator();

            // Walk through the ready keys collection and process date requests.
            while (i.hasNext()) {
                SelectionKey sk = (SelectionKey) i.next();
                if(sk.isAcceptable()){
                    // The key indexes into the selector so you
                    // can retrieve the socket that's ready for I/O
                    ServerSocketChannel nextReady = (ServerSocketChannel) sk
                    .channel();
                    //从ServerSocketChannel中获取SocketChannel并注册到selector
                    SocketChannel s = nextReady.accept();
                    s.register(acceptSelector, SelectionKey.OP_READ);
                }else if(sk.isReadable()){
                    SocketChannel client = (SocketChannel) sk.channel();
                    //ignore channel read
                }
                i.remove();    
            }
        }
    }

    // Entry point.
    public static void main(String[] args) {
        // Parse command line arguments and
        // create a new time server (no arguments yet)
        try {
            NBTimeServer nbt = new NBTimeServer();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### netty server端过程

netty server端大致遵循着如下的框架,通过bootstrap启动类配置参数，然后调用bind()方法绑定在指定端口上。

```JAVA .{line-numbers}
//code segment
public void run(String... args) throws Exception {
    EventLoopGroup bossGroup = new NioEventLoopGroup();
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    try {
        ServerBootstrap b = new ServerBootstrap();
        b.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new SjMessageDecoder())
                                .addLast(new SjMessageEncoder())
                                .addLast(new SjServerHandler(registerMap))
                                .addLast(new SjServerReportDataHandler());
                    }
                })
                .option(ChannelOption.SO_BACKLOG, 128)
                .childOption(ChannelOption.SO_KEEPALIVE, true);

        ChannelFuture f = b.bind(16001).sync(); // 绑定到15001端口
        log.info("Netty TCP server started and listening on " + f.channel().localAddress());
        f.channel().closeFuture().sync(); // 等待服务器关闭
    } finally {
        workerGroup.shutdownGracefully();
        bossGroup.shutdownGracefully();
    }
}
```

bind()函数的调用流程如下：
{% asset_img nettySeries003_006.png bind调用过程%}
bind()通过channel->pipeline->context->NioServerSocketChannel的过程最终调用了ServerChannel的实现类，ServerChannel的实现类不止一种，但这里主要考虑的是JAVA NIO的对应实现NioServerSocketChannel。

NioServerSocketChannel中实现了真正的绑定端口，javaChannel方法返回了基类中的SelectableChannel对象ch，然后调用ch的bind方法,该方法是java NIO中的方法，对应java_nio_num1代码片段中的27行。

```JAVA .{line-numbers}
//code segment:NioServerSocketChannel
protected void doBind(SocketAddress localAddress) throws Exception {
        if (PlatformDependent.javaVersion() >= 7) {
            javaChannel().bind(localAddress, config.getBacklog());
        } else {
            javaChannel().socket().bind(localAddress, config.getBacklog());
        }
    }
```

当客户端请求过来是，从ServerSocketChannel监听生成连接的socket也在NioServerChannel中，生成SocketChannel后，SocketChannel会将自己注册到selector中，等待客户端传输数据过来或者写数据到客户端。

```JAVA .{line-numbers}
//code segment
protected int doReadMessages(List<Object> buf) throws Exception {
        SocketChannel ch = SocketUtils.accept(javaChannel());

        try {
            if (ch != null) {
                buf.add(new NioSocketChannel(this, ch));
                return 1;
            }
        } catch (Throwable t) {
            logger.warn("Failed to create a new channel from an accepted socket.", t);

            try {
                ch.close();
            } catch (Throwable t2) {
                logger.warn("Failed to close a socket.", t2);
            }
        }

        return 0;
    }
```

CS模式服务端功能大致如上：首先ServerSocketChannel监听在指定端口，当有客户端请求连接时，ServerSocketChannel生成代表连接信息的Channel，该客户端的读写请求都通过该Channel完成。netty完成上述功能的基础上，引入了事件循环机制，ChannelHandler等机制使得该过程能快速高效的进行。
