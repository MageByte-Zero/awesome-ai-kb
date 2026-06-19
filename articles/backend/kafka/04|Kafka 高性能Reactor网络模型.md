在这篇文章中，我将深入解析 Kafka 背后的 7 大技术优势中的 Kafka 网络 I/O 模型，Kafka 的高性能部分归功于其网络 IO 模型的精心设计。

本文将深入解析 Kafka 的网络 IO 模型，从源码和架构的角度揭示其高效非阻塞通信的奥秘。

## Java NIO

大多数 Java 程序员提到网络框架，首先想到的就是 Netty。Dubbo、Avro-RPC 等等优秀的框架都使用 Netty 作为底层的网络通信框架。

Kafka 并没有采用 Netty，而是自己基于 Java NIO 实现了一个网络模型，采用和 Netty 一样的 Reactor 线程模型。

Java NIO (New Input/Output) 是 Java 1.4 中引入的，用于替代传统的阻塞式 IO。NIO 提供了多种功能，其中最关键的包括：

- **Channels**：数据的载体，类似于传统 IO 中的流。
- **Buffers**：存储数据的容器，读写操作都在缓冲区上进行。
- **Selectors**：用于监听多个通道的事件（如连接、读、写），实现非阻塞 IO。

### Channels

Java NIO 中的通道是一个双向的通信通道，可以用于读、写、或者同时进行读写操作。

常见的通道包括 FileChannel、SocketChannel、ServerSocketChannel 和 DatagramChannel。

Kafka 主要使用 SocketChannel 和 ServerSocketChannel 来处理网络通信。

### Buffers

缓冲区是 Java NIO 的核心部分，所有的数据都要通过缓冲区进行读写操作。常用的缓冲区类型包括 ByteBuffer、CharBuffer、IntBuffer 等。Kafka 主要使用 ByteBuffer 来处理网络数据。

### Selectors

选择器是 Java NIO 提供的一个机制，用于监听多个通道的事件（如连接、读、写）。选择器允许一个单独的线程管理多个通道，从而实现高效的非阻塞 IO 操作。

## Reactor 网络 I/O 模型

Kafka 使用 Java NIO 实现了高效的网络通信。其网络 IO 模型基于 Reactor 模式，该模式的核心思想是单线程处理多个 IO 事件，避免了多线程切换的开销。Reactor 线程模型如图 1 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/kafka_list/reactor.png)

图 1

Reacotr 模型主要分为三个角色。

- Reactor：把 I/O 事件根据类型分配给分配给对应的 Handler 处理。
- Acceptor：处理客户端连接事件。
- Handler：处理读写等任务。

从图 1 可以知道，多个客户端发送请求给 Reactor，Acceptor 会处理客户端的连接事件，它会通过 Dispatch 线程分发这些请求给对应的 handler。

Acceptor 线程只用来进行请求分发，所以是轻量级的，具有非常高的吞吐量。

在传统阻塞 I/O 模型中，每个连接都需要独立线程处理，当并发数大时，创建线程数多，占用资源；采用阻塞 I/O 模型，连接建立后，若当前线程没有数据可读，线程会阻塞在读操作上，造成资源浪费。

针对传统阻塞 I/O 模型的两个问题，Reactor 模型基于池化思想，避免为每个连接创建线程，连接完成后将业务处理交给线程池处理；基于 IO 复用模型，多个连接共用同一个阻塞对象，不用等待所有的连接。遍历到有新数据可以处理时，操作系统会通知程序，线程跳出阻塞状态，进行业务逻辑处理。

## kafka 网络 I/O 模型核心组件

Kafka 使用 Java NIO 实现了高效的网络通信。其网络 IO 模型基于 Reactor 模式，该模式的核心思想是单线程处理多个 IO 事件，避免了多线程切换的开销。

Kafka 的网络模块由以下几个关键组件组成：

- **SocketServer**：Kafka 网络通信核心类，持有 Acceptor 和 Processor 两个组件。
  - **Acceptor**：负责监听新的客户端连接线程，并将其注册分发给 `Processor` 进行处理。
  - **Processor**：每个 Processor 对应一个独立的线程，处理 Acceptor 分发过来的可读事件，处理客户端的读请求，并把处理后的响应写回给客户端，主要职责有：
    - 接受新的客户端连接。
    - 读取客户端请求数据。
    - 处理请求并生成响应。
    - 将响应数据写回客户端。
- **Selector**：`Selector` 是 Kafka 实现非阻塞 IO 的关键组件，它通过监听多个通道（`Channel`）的事件，实现高效的网络通信。`Selector` 类的核心是 Java NIO 提供的 `java.nio.channels.Selector`，Kafka 在其基础上进行了封装和扩展。
- **KafkaChannel**：对 Java `SocketChannel` 的封装，封装是实际的读写 I/O 操作。
- **RequestChannel**：封装了和 API 层通信的 Request、Response 以及相应的 requestQueue 和 responseQueue 通信队列。

Kafka 基于 Reactor 模型架构如图 2 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/acceptor.png)

图 2

## Kafka 网络 I/O 架构原理

Kafka 网络层核心类是 `SocketServer` ，包含一个或者多个`Acceptor`（如果 listeners 配置了多个就会创建多个 `Acceptor`）用于监听客户端的新连接请求。

多线程多 `Reactor` 模型：每个`Acceptor` 独占一个 `Selector`，每个 `Acceptor` 拥有多个`Processor` 线程；

`Acceptor` 会使用 `round-robin` 负载均衡算法将成功的连接 放入 Processor 的队列里，每个`Processor` 有自己的 Selector，监听网络 I/O 读写事件的发生，也就是说 `Processor` 主要处理网络 I/O 的可读/可写事件。

`Processor` 线程与 `RequestHandler` 线程之间通过 `RequestChannel` 组件进行通信。

当**网络 I/O 读事件**发生时，所有 `Processor`会将请求包装成 Request 对象放入 RequestChannel 组件中默认大小 500 的全局 `requestQueue`队列中。

然后一堆 `RequestHandler` 线程不停的从 `requestQueue` 队列中获取请求信息，然后进行相应的业务逻辑处理，比如说发送消息的请求，它就会找到对应的 `parittion`，写到对应 `partition` 所在磁盘的文件中。

`RequestHandler` 处理 Request，完成业务逻辑后，需要把处理结果告诉客户端，这个时候 `RequestHandler` 线程会将处理结果放到 处理该 Request 对应的 Processor 线程内的 `ResponseQueue` 响应队列中。

I/O 写事件发生，`Processor` 将相应数据写回客户端。架构如图 3 所示。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202406032202284.png)

图 3

**博主简介**

码哥，9 年互联网后端工作经验，目前担任后端架构师主责。InfoQ 签约作者、51CTO Top 红人，阿里云开发者社区专家博主，擅长 Redis、Spring、Kafka、MySQL 技术和云原生微服务。