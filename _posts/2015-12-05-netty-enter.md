---
layout: post
title: 依赖调度系统设计
date: 2015-11-22
categories: blog
tags: [netty]
description: 任务的调度和依赖关系
---
# netty 入门
## 什么是netty
    Netty 是一个利用java的高级网络的的能力，隐藏了背后的复杂性而提供一个易用的api
    的客户端/服务器端框架。Netty提共高性能和高扩展性。让你可以专注自己感兴趣的东西。
    
## 构成部分
    非阻塞IO不迫使我们等待操作的完成。在这种能力的基础上，真正的异步IO是一个重要的一步：
    一个异步方法完成时立即返回并直接或稍后通知用户
    在一个网络环境的异步模型可以更有效的利用资源，可以快速连续执行多个调用。
### Channle 
    一个Channel是NIO的基本结构，它代表了一个开放的链接到一个实体如硬件设备，文件，
    网络套接字，或程序组件。能够执行一个或多个不同的IO操作，例如读写。
### Callback
    CallBack(回调)是一个简单的方法，提供给另一种方法作为引用，这个后者就可以在某个合适
    的时间调用前者。这种技术被广泛采用，最常用的场景是通知其他人操作已完成。
    Netty 内部使用回调处理事件，一旦这样回调被触发，事件可以用接口ChannelHandler来处理。
    如下面的代码，一个连接被建立后，调用ChannelActive（），打印出一条消息
    
```
 public class ConnectHandler extends ChannelHandlerAdpter{ 
     public void channelActive(ChannelHandlerContext ctx){ //1
         System.out.println("client:"+ctx.channel().remoteAddress()+"connected")
     }
 }

```  
    1.当建立一个新连接时调用ChannelActive()
    
### Future
    future 提供了另外一种通知应用操作已经完成的方式。这个对象作为一个异步操作结果的占位符。它将在将来某个时候完成并提供结果。
    JDK附带的java.util.concurrent.Future,但提供的实现，只允许您手动检查操作是否完成或阻塞。这很麻烦，故Netty提供了自己的实现：ChannelFuture。用于在执行异步操作的时候使用。
    Netty 提供多个附件方法来允许一个或多个ChannelFuture实列。这个回调方法operationComplete()会在操作完成时调用。
    每个Netty的outBound I/O 操作都会返回一个ChannelFuture；这样就不会阻塞。这就是Netty所谓的“自底向上的异步和事件驱动”
    
### Event 和 Handler
    Netty使用不同的事件来通知我们更改的状态或操作的状态。这让我们能够根据发生的事件触发适当的行为。这些行为可能包括：
        1.日志
        2.数据转换。
        3.流控制。
        4.应用程序逻辑。
    由于Netty是一个网络框架，时间很清晰的和入站和出站数据流相关。因为一些事件，可能触发传入的数据或状态的变化。包括：
        1。活动或非活动连接。
        2.数据的读取。
        3.用户事件
        4.错误。

    出站事件是由于在未来操作将触发一个动作。这些包括：
        1.打开或关闭一个连接。
        2.写或冲刷数据到socket
    每个事件都可分配给用户实现处理程序类的方法。这说明了事件驱动的范例可直接转化为应用程序构建块
    
    
    
下图显示了一个事件可以由一连串的事件处理器来处理
![MacDown netty](/Users/hao.su/Desktop/nettyIo.png)

### 整合 Future,CallBack和Handler
    Netty 的异步编程模型是建立在future和callBack的概念上。
    拦截操作和转化出站或出站数据只需要你提供回调或利用future操作返回的。
    这使得链操作简单，高效，促进编写可重用的代码。一个Netty的设计主要目标是促进：“关注分离点：”你的业务逻辑从网络基础设施应用中分离
    
### Selector，Event和Event Loop
    Netty通过触发事件从应用程序中抽象出Selector，从而避免手写调度代码，EventLoop分配给每个Channel来处理所有的事件 包括：
        1.注册事件
        2.调度事件到ChannelHandler
        3.安排进一步活动
    该EventLoop本身只有一个线程驱动，它给一个Channel处理所有的IO事件。并且在EventLoop生命周期内不会改变
    这个简单而强大的线程模型消除可能对你的ChannelHandler同步的关注。这样可以专注提供正确的回调逻辑来执行。
        


    
    
    
   
    
    
   

    


    
    
    
    
    
    
    
    
    