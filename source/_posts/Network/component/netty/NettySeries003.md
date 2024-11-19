---
title: NettySeries003 bind执行过程
date: 2024-11-08 09:57:01
mathjax: true
categories:
- [网络编程,netty]
tags:
- 原理
- 实现
description: "netty系列基础部分：bind执行过程 <br>
- 通过分析bind了解eventloop是怎么运作的"
---

## 背景

Netty将服务启动过程抽象成了bootstrap,通过配置ServerBootStrap并调用bind方法实现监听指定端口，下面通过分析bind的执行流了解线程以及事件是如何在Netty中实现的。

## 功能

典型的netty服务端启动过程如下，创建一个ServerBootstrap对象b，配置b的参数，并通过调用b.bind方法实现绑定指定端口，启动监听接受请求。

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

