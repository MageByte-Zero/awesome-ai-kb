> 上帝视角拆解 Tomcat 架构设计，在了解整个组件设计思路之后。我们需要下凡深入了解每个组件的细节实现。从远到近，架构给人以宏观思维，细节展现饱满的美。关注「码哥字节」获取更多硬核，你，准备好了么？

上回「码哥字节」站在上帝视角给大家拆解了 Tomcat 架构设计，分析 Tomcat 如何实现启动、停止，通过设计连接池与容器两大组件完成了一个请求的接受与响应。**连接器负责对外交流，处理 socket 连接，容器对内负责，加载 Servlet 以及处理具体 Request 请求与响应**。[详情点我进入传输门：Tomcat 架构解析到工作借鉴](https://mp.weixin.qq.com/s/fU5Jj9tQvNTjRiT9grm6RA)。

## 高并发拆解核心准备

这回，再次拆解，**专注 Tomcat 高并发设计之道与性能调优**，让大家对整个架构有更高层次的了解与感悟。其中设计的每个组件思路都是将 Java 面向对象、面向接口、如何封装变与不变，如何根据实际需求抽象不同组件分工合作，如何设计类实现单一职责，怎么做到将相似功能高内聚低耦合，设计模式运用到极致的学习借鉴。

这次主要涉及到的是 I/O 模型，以及线程池的基础内容。

在学习之前，希望大家积累以下一些技术内容，很多内容「码哥字节」也在历史文章中分享过。大家可爬楼回顾……。**希望大家重视如下几个知识点，在掌握以下知识点再来拆解 Tomcat，就会事半功倍，否则很容易迷失方向不得其法**。

一起来看 Tomcat 如何实现并发连接处理以及任务处理，性能的优化是每一个组件都起到对应的作用，如何使用最少的内存，最快的速度执行是我们的目标。

### 设计模式

[模板方法模式](https://mp.weixin.qq.com/s/AVfEy9fMRHhESKnfRERynw)： 抽象算法流程在抽象类中，封装流程中的变化与不变点。将变化点延迟到子类实现，达到代码复用，开闭原则。

[观察者模式](https://mp.weixin.qq.com/s/7FL-m75fOctUVAMI_YgIww)：针对事件不同组件有不同响应机制的需求场景，达到解耦灵活通知下游。

[责任链模式](https://mp.weixin.qq.com/s/Ar_FZ6ck_WEjr9ia9bjmcQ)：将对象连接成一条链，将沿着这条链传递请求。在 Tomcat 中的 Valve 就是该设计模式的运用。

更多设计模式可查看「码哥字节」之前的设计模式专辑，这里是[传送门](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU3NDkwMjAyOQ==&action=getalbum&album_id=1351452628844363777&subscene=159&subscene=&scenenote=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FAr_FZ6ck_WEjr9ia9bjmcQ#wechat_redirect)。

### I/O 模型

Tomcat 实现高并发接收连接，必然涉及到 I/O 模型的运用，了解同步阻塞、异步阻塞、I/O 多路复用，异步非阻塞相关概念以及 Java NIO 包的运用很有必要。本文也会带大家着重说明 I/O 是如何在 Tomcat 运用实现高并发连接。大家通过本文我相信对 I/O 模型也会有一个深刻认识。

### Java 并发编程

实现高并发，除了整体每个组件的优雅设计、设计模式的合理、I/O 的运用，还需要线程模型，如何高效的并发编程技巧。在高并发过程中，不可避免的会出现多个线程对共享变量的访问，需要加锁实现，如何高效的降低锁冲突。因此作为程序员，要有意识的尽量避免锁的使用，比如可以使用原子类 CAS 或者并发集合来代替。如果万不得已需要用到锁，也要尽量缩小锁的范围和锁的强度。

对于并发相关的基础知识，如果读者感兴趣「码哥字节」后面也给大家安排上，目前也写了部分并发专辑，大家可移步到历史文章或者专辑翻阅，这里是[传送门](http://mp.weixin.qq.com/mp/homepage?__biz=MzU3NDkwMjAyOQ==&hid=7&sn=660e8728f9eb24248cc7953796dbe773&scene=18#wechat_redirect)，主要讲解了并发实现的原理、什么是内存可见性，JMM 内存模模型、读写锁等并发知识点。

## Tomcat 总体架构

再次回顾下 Tomcat 整体架构设计，主要设计了 **connector 连接器**处理 TCP/IP 连接，**container 容器**作为 Servlet 容器，处理具体的业务请求。对外对内分别抽象两个组件实现拓展。

- 一个 Tomcat 实例默认会有一个 Service，而一个 Service 可以包含多个连接器。连接器主要有 ProtocalHandler 和 Adapter 两个组件共同完成连接器核心功能。
- `ProtocolHandler` 主要由 `Acceptor` 以及 `SocketProcessor` 构成，实现了 TCP/IP 层 的 Socket 读取并转换成 `TomcatRequest` 和 `TomcatResponse`，最后根据 http 或者 ajp 协议获取合适的 `Processor` 解析为应用层协议，并通过 Adapter 将 TomcatRequest、TomcatResponse 转化成 标准的 ServletRequest、ServletResponse。通过 `getAdapter().service(request, response);`将请求传递到 Container 容器。
- adapter.service()实现将请求转发到容器 `org.apache.catalina.connector.CoyoteAdapter`

```java
// Calling the container
connector.getService().getContainer().getPipeline().getFirst().invoke(
                        request, response);
```

这个调用会触发 getPipeline 构成的责任链模式将请求一步步走入容器内部，每个容器都有一条 Pipeline，通过 First 开始到 Basic 结束并进入容器内部持有的子类容器，最后到 Servlet，这里就是责任链模式的经典运用。具体的源码组件是 Pipeline 构成一条请求链，每一个链点由 Valve 组成。「码哥字节」在上一篇[Tomcat 架构解析到工作借鉴](https://mp.weixin.qq.com/s/fU5Jj9tQvNTjRiT9grm6RA) 已经详细讲解。如下图所示，整个 Tomcat 的架构设计重要组件清晰可见，希望大家将这个全局架构图深深印在脑海里，掌握全局思路才能更好地分析细节之美。

![Tomcat 架构](../../../../tomcat高性能拆解/20200725210507.png)

### 启动流程：startup.sh 脚本到底发生了什么

![Tomcat 启动流程](../../../../tomcat高性能拆解/20200725214643.png)

- Tomcat 本生就是一个 Java 程序，所以 startup.sh 脚本就是启动一个 JVM 来运行 Tomcat 的启动类 Bootstrap。
- Bootstrap 主要就是实例化 Catalina 和初始化 Tomcat 自定义的类加载器。热加载与热部署就是靠他实现。
- Catalina: 解析 server.xml 创建 Server 组件，并且调用 Server.start() 方法。
- Server：管理 Service 组件，调用 Server 的 start() 方法。
- Service：主要职责就是管理简介器的顶层容器 Engine，分别调用 `Connector` 和 `Engine` 的 `start` 方法。

Engine 容器主要就是组合模式将各个容器根据父子关系关联，并且 Container 容器继承了 Lifecycle 实现各个容器的初始化与启动。Lifecycle 定义了 `init()、start()、stop()` 控制整个容器组件的生命周期实现一键启停。

**这里就是一个面向接口、单一职责的设计思想** ，Container 利用组合模式管理容器，LifecycleBase 抽象类继承 Lifecycle 将各大容器生命周期统一管理这里便是，而实现初始化与启动的过程又 LifecycleBase 运用了[模板方法](https://mp.weixin.qq.com/s/AVfEy9fMRHhESKnfRERynw)设计模式抽象出组件变化与不变的点，将不同组件的初始化延迟到具体子类实现。并且利用观察者模式发布启动事件解耦。

具体的 init 与 start 流程如下泳道图所示：这是我在阅读源码 debug 所做的笔记，读者朋友们**不要怕笔记花费时间长，自己跟着 debug 慢慢记录，相信会有更深的感悟**。

**init 流程**

![Tomcat Init](../../../../tomcat高性能拆解/20200725220526.png)

**start 流程**

![Tomcat start](../../../../tomcat高性能拆解/image-20200725220713598.png)

读者朋友根据我的两篇内容，抓住主线组件去 debug，然后跟着该泳道图阅读源码，我相信都会有所收获，并且事半功倍。在读源码的过程中，切勿进入某个细节，一定要先把各个组件抽象出来，了解每个组件的职责即可。最后在了解每个组件的职责与设计哲学之后再深入理解每个组件的实现细节，千万不要一开始就想着深入理解具体一篇叶子。

每个核心类我在架构设计图以及泳道图都标识出来了，「码哥字节」给大家分享下如何高效阅读源码，以及保持学习兴趣的心得体会。

### 如何正确阅读源码

切勿陷入细节，不看全局：**我还没弄清楚森林长啥样，就盯着叶子看** ，看不到全貌和整体设计思路。所以阅读源码学习的时候不要一开始就进入细节，而是宏观看待整体架构设计思想，模块之间的关系。

**1.阅读源码之前，需要有一定的技术储备**

比如常用的设计模式，这个必须掌握，尤其是：**模板方法、策略模式、单例、工厂、观察者、动态代理、适配器、责任链、装饰器。**大家可以看 「码哥字节」关于设计模式的历史文章，打造好的基础。

**2.必须会使用这个框架/类库，精通各种变通用法**

魔鬼都在细节中，如果有些用法根本不知道，可能你能看明白代码是什么意思，但是不知道它为什么这些写。

**3.先去找书，找资料，了解这个软件的整体设计。**

从全局的视角去看待，上帝视角理出主要核心架构设计，先森林后树叶。都有哪些模块？ 模块之间是怎么关联的？怎么关联的？

可能一下子理解不了，但是要建立一个整体的概念，就像一个地图，防止你迷航。

在读源码的时候可以时不时看看自己在什么地方。就像「码哥字节」给大家梳理好了 Tomcat 相关架构设计，然后自己再尝试跟着 debug，这样的效率如虎添翼。

**4. 搭建系统，把源代码跑起来！**

Debug 是非常非常重要的手段， 你想通过只看而不运行就把系统搞清楚，那是根本不可能的！合理运用调用栈（观察调用过程上下文）。

**5.笔记**

一个非常重要的工作就是记笔记（又是写作！），画出系统的类图（不要依靠 IDE 给你生成的）， 记录下主要的函数调用， 方便后续查看。

**文档工作极为重要**，因为代码太复杂，人的大脑容量也有限，记不住所有的细节。 文档可以帮助你记住关键点， 到时候可以回想起来，迅速地接着往下看。

要不然，你今天看的，可能到明天就忘个差不多了。所以朋友们记得收藏后多翻来看看，尝试把源码下载下来反复调试。

#### 错误方式

- 陷入细节，不看全局：**我还没弄清楚森林长啥样，就盯着叶子看** ，看不到全貌和整体设计思路。所以阅读源码学习的时候不要一开始就进入细节，而是宏观看待整体架构设计思想，模块之间的关系。
- 还没学会用就研究如何设计：首先基本上框架都运用了设计模式，我们最起码也要了解常用的设计模式，即使是“背”，也得了然于胸。在学习一门技术，我推荐先看官方文档，看看有哪些模块、整体设计思想。然后下载示例跑一遍，最后才是看源码。
- 看源码深究细节：到了看具体某个模块源码的时候也要下意识的不要去深入细节，重要的是学习设计思路，而不是具体一个方法实现逻辑。除非自己要基于源码做二次开发，而且二次开发也是基于在了解扎鞥个架构的情况下才能深入细节。

## 组件设计-落实单一职责、面向接口思想

当我们接到一个功能需求的时候，最重要的就是抽象设计，将功能拆解主要核心组件，然后找到需求的变化与不变点，将相似功能内聚，功能之间若耦合，同时对外支持可拓展，对内关闭修改。努力做到一个需求下来的时候我们需要合理的抽象能力抽象出不同组件，而不是一锅端将所有功能糅合在一个类甚至一个方法之中，这样的代码牵一发而动全身，无法拓展，难以维护和阅读。

带着问题我们来分析 Tomcat 如何设计组件完成连接与容器管理。

看看 Tomcat 如何实现将 Tomcat 启动，并且又是如何接受请求，将请求转发到我们的 Servlet 中。

### Catalina

主要任务就是创建 Server，并不是简单创建，而是解析 server.xml 文件把文件配置的各个组件意义创建出来，接着调用 Server 的 init() 和 start() 方法，启动之旅从这里开始…，同时还要兼顾异常，比如关闭 Tomcat 还需要做到优雅关闭启动过程创建的资源需要释放，Tomcat 则是在 JVM 注册一个「关闭钩子」，源码我都加了注释，省略了部分无关代码。同时通过 `await()` 监听停止指令关闭 Tomcat。

```java
    /**
     * Start a new server instance.
     */
    public void start() {
				// 若 server 为空，则解析 server.xml 创建
        if (getServer() == null) {
            load();
        }
				// 创建失败则报错并退出启动
        if (getServer() == null) {
            log.fatal("Cannot start server. Server instance is not configured.");
            return;
        }

        // 开始启动 server
        try {
            getServer().start();
        } catch (LifecycleException e) {
            log.fatal(sm.getString("catalina.serverStartFail"), e);
            try {
                // 异常则执行 destroy 销毁资源
                getServer().destroy();
            } catch (LifecycleException e1) {
                log.debug("destroy() failed for failed Server ", e1);
            }
            return;
        }

        // 创建并注册 JVM 关闭钩子
        if (useShutdownHook) {
            if (shutdownHook == null) {
                shutdownHook = new CatalinaShutdownHook();
            }
            Runtime.getRuntime().addShutdownHook(shutdownHook);
        }
				// 通过 await 方法监听停止请求
        if (await) {
            await();
            stop();
        }
    }
```

通过「关闭钩子」，就是当 JVM 关闭的时候做一些清理工作，比如说释放线程池，清理一些零时文件，刷新内存数据到磁盘中…...

「关闭钩子」本质就是一个线程，JVM 在停止之前会尝试执行这个线程。我们来看下 CatalinaShutdownHook 这个钩子到底做了什么。

```java
    /**
     * Shutdown hook which will perform a clean shutdown of Catalina if needed.
     */
    protected class CatalinaShutdownHook extends Thread {

        @Override
        public void run() {
            try {
                if (getServer() != null) {
                    Catalina.this.stop();
                }
            } catch (Throwable ex) {
               ...
        }
    }

    /**
     * 关闭已经创建的 Server 实例
     */
    public void stop() {

        try {
            // Remove the ShutdownHook first so that server.stop()
            // doesn't get invoked twice
            if (useShutdownHook) {
                Runtime.getRuntime().removeShutdownHook(shutdownHook);
            }
        } catch (Throwable t) {
            ......
        }

        // 关闭 Server
        try {
            Server s = getServer();
            LifecycleState state = s.getState();
           // 判断是否已经关闭，若是在关闭中，则不执行任何操作
            if (LifecycleState.STOPPING_PREP.compareTo(state) <= 0
                    && LifecycleState.DESTROYED.compareTo(state) >= 0) {
                // Nothing to do. stop() was already called
            } else {
                s.stop();
                s.destroy();
            }
        } catch (LifecycleException e) {
            log.error("Catalina.stop", e);
        }

    }

```

实际上就是执行了 Server 的 stop 方法，Server 的 stop 方法会释放和清理所有的资源。

### Server 组件

来体会下面向接口设计美，看 Tomcat 如何设计组件与接口，抽象 Server 组件，Server 组件需要生命周期管理，所以继承 Lifecycle 实现一键启停。

它的具体实现类是 StandardServer，如下图所示，我们知道 Lifecycle 主要的方法是组件的 初始化、启动、停止、销毁，和 监听器的管理维护，其实就是观察者模式的设计，当触发不同事件的时候发布事件给监听器执行不同业务处理，这里就是如何解耦的设计哲学体现。

而 Server 自生则是负责管理 Service 组件。

![](../../../../tomcat高性能拆解/20200726120449.png)

接着，我们再看 Server 组件的具体实现类是 StandardServer 有哪些功能，又跟哪些类关联？

![StandardServer](../../../../tomcat高性能拆解/StandardServer.png)

在阅读源码的过程中，我们一定要多关注接口与抽象类，接口是组件全局设计的抽象；而抽象类基本上是模板方法模式的运用，主要目的就是抽象整个算法流程，将变化点交给子类，将不变点实现代码复用。

StandardServer 继承了 LifeCycleBase，它的生命周期被统一管理，并且它的子组件是 Service，因此它还需要管理 Service 的生命周期，也就是说在启动时调用 Service 组件的启动方法，在停止时调用它们的停止方法。Server 在内部维护了若干 Service 组件，它是以数组来保存的，那 Server 是如何添加一个 Service 到数组中的呢？

```java
    /**
     * 添加 Service 到定义的数组中
     *
     * @param service The Service to be added
     */
    @Override
    public void addService(Service service) {

        service.setServer(this);

        synchronized (servicesLock) {
           // 创建一个 services.length + 1 长度的 results 数组
            Service results[] = new Service[services.length + 1];
           // 将老的数据复制到 results 数组
            System.arraycopy(services, 0, results, 0, services.length);
            results[services.length] = service;
            services = results;
						// 启动 Service 组件
            if (getState().isAvailable()) {
                try {
                    service.start();
                } catch (LifecycleException e) {
                    // Ignore
                }
            }

            // 观察者模式运用，触发监听事件
            support.firePropertyChange("service", null, service);
        }

    }
```

从上面的代码可以知道，并不是一开始就分配一个很长的数组，而是在新增过程中动态拓展长度，这里就是为了节省空间，对于我们平时开发是不是也要主要空间复杂度带来的内存损耗，追求的就是极致的美。

除此之外，还有一个重要功能，上面 Caralina 的启动方法的最后一行代码就是调用了 Server 的 await 方法。

这个方法主要就是监听停止端口，在 **await** 方法里会创建一个 Socket 监听 8005 端口，并在一个死循环里接收 Socket 上的连接请求，如果有新的连接到来就建立连接，然后从 Socket 中读取数据；如果读到的数据是停止命令“SHUTDOWN”，就退出循环，进入 stop 流程。

### Service

同样是面向接口设计，Service 组件的具体实现类是 StandardService，Service 组件依然是继承 Lifecycle 管理生命周期，这里不再累赘展示图片关系图。我们先来看看 Service 接口主要定义的方法以及成员变量。通过接口我们才能知道核心功能，在阅读源码的时候一定要多关注每个接口之间的关系，不要急着进入实现类。

```java
public interface Service extends Lifecycle {

  // ----------主要成员变量

    //Service 组件包含的顶层容器 Engine
    public Engine getContainer();

    // 设置 Service 的 Engine 容器
    public void setContainer(Engine engine);

    // 该 Service 所属的 Server 组件
    public Server getServer();

    // --------------------------------------------------------- Public Methods

   // 添加 Service 关联的连接器
    public void addConnector(Connector connector);

    public Connector[] findConnectors();

   // 自定义线程池
    public void addExecutor(Executor ex);

   // 主要作用就是根据 url 定位到 Service，Mapper 的主要作用就是用于定位一个请求所在的组件处理
    Mapper getMapper();
}

```

接着再来细看 Service 的实现类：

```java
public class StandardService extends LifecycleBase implements Service {
    // 名字
    private String name = null;

    //Server 实例
    private Server server = null;

    // 连接器数组
    protected Connector connectors[] = new Connector[0];
    private final Object connectorsLock = new Object();

    // 对应的 Engine 容器
    private Engine engine = null;

    // 映射器及其监听器，又是观察者模式的运用
    protected final Mapper mapper = new Mapper();
    protected final MapperListener mapperListener = new MapperListener(this);
}
```

StandardService 继承了 LifecycleBase 抽象类，抽象类定义了 三个 final 模板方法定义生命周期，每个方法将变化点定义抽象方法让不同组件时间自己的流程。这里也是我们学习的地方，利用模板方法抽象变与不变。

此外 StandardService 中还有一些我们熟悉的组件，比如 Server、Connector、Engine 和 Mapper。

那为什么还有一个 MapperListener？这是因为 Tomcat 支持热部署，当 Web 应用的部署发生变化时，Mapper 中的映射信息也要跟着变化，MapperListener 就是一个监听器，它监听容器的变化，并把信息更新到 Mapper 中，这是典型的观察者模式。下游服务根据多上游服务的动作做出不同处理，这就是[观察者模式](https://mp.weixin.qq.com/s/7FL-m75fOctUVAMI_YgIww)的运用场景，实现一个事件多个监听器触发，事件发布者不用调用所有下游，而是通过观察者模式触发达到解耦。

Service 管理了 连接器以及 Engine 顶层容器，所以继续进入它的 startInternal 方法，其实就是 LifecycleBase 模板定义的 抽象方法。看看他是怎么启动每个组件顺序。

```java
protected void startInternal() throws LifecycleException {

    //1. 触发启动监听器
    setState(LifecycleState.STARTING);

    //2. 先启动 Engine，Engine 会启动它子容器，因为运用了组合模式，所以每一层容器在会先启动自己的子容器。
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }

    //3. 再启动 Mapper 监听器
    mapperListener.start();

    //4. 最后启动连接器，连接器会启动它子组件，比如 Endpoint
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            if (connector.getState() != LifecycleState.FAILED) {
                connector.start();
            }
        }
    }
}
```

Service 先启动了 Engine 组件，再启动 Mapper 监听器，最后才是启动连接器。这很好理解，因为内层组件启动好了才能对外提供服务，才能启动外层的连接器组件。而 Mapper 也依赖容器组件，容器组件启动好了才能监听它们的变化，因此 Mapper 和 MapperListener 在容器组件之后启动。组件停止的顺序跟启动顺序正好相反的，也是基于它们的依赖关系。

### Engine

作为 Container 的顶层组件，所以 Engine 本质就是一个容器，继承了 ContainerBase ，看到抽象类再次运用了模板方法设计模式。ContainerBase 使用一个 `HashMap<String, Container> children = new HashMap<>();` 成员变量保存每个组件的子容器。同时使用 `protected final Pipeline pipeline = new StandardPipeline(this);` Pipeline 组成一个管道用于处理连接器传过来的请求，责任链模式构建管道。

```java
 public class StandardEngine extends ContainerBase implements Engine {
 }
```

Engine 的子容器是 Host，所以 children 保存的就是 Host。

我们来看看 ContainerBase 做了什么...

- initInternal 定义了容器初始化，同时创建了专门用于启动停止容器的线程池。
- startInternal：容器启动默认实现，通过组合模式构建容器父子关系，首先获取自己的子容器，使用 startStopExecutor 启动子容器。

```java
public abstract class ContainerBase extends LifecycleMBeanBase
        implements Container {

   // 提供了默认初始化逻辑
    @Override
    protected void initInternal() throws LifecycleException {
        BlockingQueue<Runnable> startStopQueue = new LinkedBlockingQueue<>();
       // 创建线程池用于启动或者停止容器
        startStopExecutor = new ThreadPoolExecutor(
                getStartStopThreadsInternal(),
                getStartStopThreadsInternal(), 10, TimeUnit.SECONDS,
                startStopQueue,
                new StartStopThreadFactory(getName() + "-startStop-"));
        startStopExecutor.allowCoreThreadTimeOut(true);
        super.initInternal();
    }

  // 容器启动
    @Override
    protected synchronized void startInternal() throws LifecycleException {

        // 获取子容器并提交到线程池启动
        Container children[] = findChildren();
        List<Future<Void>> results = new ArrayList<>();
        for (Container child : children) {
            results.add(startStopExecutor.submit(new StartChild(child)));
        }
        MultiThrowable multiThrowable = null;
        // 获取启动结果
        for (Future<Void> result : results) {
            try {
                result.get();
            } catch (Throwable e) {
                log.error(sm.getString("containerBase.threadedStartFailed"), e);
                if (multiThrowable == null) {
                    multiThrowable = new MultiThrowable();
                }
                multiThrowable.add(e);
            }

        }
       ......

        // 启动 pipeline 管道，用于处理连接器传递过来的请求
        if (pipeline instanceof Lifecycle) {
            ((Lifecycle) pipeline).start();
        }
				 // 发布启动事件
        setState(LifecycleState.STARTING);
        // Start our thread
        threadStart();
    }


}
```

继承了 LifecycleMBeanBase 也就是还实现了生命周期的管理，提供了子容器默认的启动方式，同时提供了对子容器的 CRUD 功能。

Engine 在启动 Host 容器就是 使用了 ContainerBase 的 startInternal 方法。Engine 自己还做了什么呢？

我们看下 构造方法，pipeline 设置了 setBasic，创建了 StandardEngineValve。

```java
/**
     * Create a new StandardEngine component with the default basic Valve.
     */
    public StandardEngine() {

        super();
        pipeline.setBasic(new StandardEngineValve());
        .....

    }
```

容器主要的功能就是处理请求，把请求转发给某一个 Host 子容器来处理，具体是通过 Valve 来实现的。每个容器组件都有一个 Pipeline 用于组成一个责任链传递请求。而 Pipeline 中有一个基础阀（Basic Valve），而 Engine 容器的基础阀定义如下：

```java
final class StandardEngineValve extends ValveBase {
    @Override
    public final void invoke(Request request, Response response)
        throws IOException, ServletException {

        // 选择一个合适的 Host 处理请求，通过 Mapper 组件获取到合适的 Host
        Host host = request.getHost();
        if (host == null) {
            response.sendError
                (HttpServletResponse.SC_BAD_REQUEST,
                 sm.getString("standardEngine.noHost",
                              request.getServerName()));
            return;
        }
        if (request.isAsyncSupported()) {
            request.setAsyncSupported(host.getPipeline().isAsyncSupported());
        }

        // 获取 Host 容器的 Pipeline first Valve ,将请求转发到 Host
        host.getPipeline().getFirst().invoke(request, response);
}
```

这个基础阀实现非常简单，就是把请求转发到 Host 容器。处理请求的 Host 容器对象是从请求中拿到的，请求对象中怎么会有 Host 容器呢？这是因为请求到达 Engine 容器中之前，Mapper 组件已经对请求进行了路由处理，Mapper 组件通过请求的 URL 定位了相应的容器，并且把容器对象保存到了请求对象中。

### 组件设计总结

大家有没有发现，Tomcat 的设计几乎都是面向接口设计，也就是通过接口隔离功能设计其实就是单一职责的体现，每个接口抽象对象不同的组件，通过抽象类定义组件的共同执行流程。单一职责四个字的含义其实就是在这里体现出来了。在分析过程中，我们看到了观察者模式、模板方法模式、组合模式、责任链模式以及如何抽象组件面向接口设计的设计哲学。

## 连接器之 I/O 模型与线程池设计

连接器主要功能就是接受 TCP/IP 连接，限制连接数然后读取数据，最后将请求转发到 `Container` 容器。所以这里必然涉及到 I/O 编程，今天带大家一起分析 Tomcat 如何运用 I/O 模型实现高并发的，一起进入 I/O 的世界。

I/O 模型主要有 5 种：**同步阻塞、同步非阻塞、I/O 多路复用、信号驱动、异步 I/O**。是不是很熟悉但是又傻傻分不清他们有何区别？

所谓的**I/O 就是计算机内存与外部设备之间拷贝数据的过程**。

CPU 是先把外部设备的数据读到内存里，然后再进行处理。请考虑一下这个场景，当程序通过 CPU 向外部设备发出一个读指令时，数据从外部设备拷贝到内存往往需要一段时间，这个时候 CPU 没事干了，程序是主动把 CPU 让给别人？还是让 CPU 不停地查：数据到了吗，数据到了吗……

这就是 I/O 模型要解决的问题。今天我会先说说各种 I/O 模型的区别，然后重点分析 Tomcat 的 NioEndpoint 组件是如何实现非阻塞 I/O 模型的。

### I/O 模型

一个网络 I/O 通信过程，比如网络数据读取，会涉及到两个对象，分别是调用这个 I/O 操作的用户线程和操作系统内核。一个进程的地址空间分为用户空间和内核空间，用户线程不能直接访问内核空间。

网络读取主要有两个步骤：

- 用户线程等待内核将数据从网卡复制到内核空间。
- 内核将数据从内核空间复制到用户空间。

同理，将数据发送到网络也是一样的流程，将数据从用户线程复制到内核空间，内核空间将数据复制到网卡发送。

不同 I/O 模型的区别：实现这两个步骤的方式不一样。

- 对于同步，则指的应用程序调用一个方法是否立马返回，而不需要等待。
- 对于阻塞与非阻塞：主要就是数据从内核复制到用户空间的读写操作是否是阻塞等待的。

#### 同步阻塞 I/O

用户线程发起`read`调用的时候，线程就阻塞了，只能让出 CPU，而内核则等待网卡数据到来，并把数据从网卡拷贝到内核空间，当内核把数据拷贝到用户空间，再把刚刚阻塞的读取用户线程唤醒，两个步骤的线程都是阻塞的。

![同步阻塞 I/O](../../../../tomcat高性能拆解/20200810162926.png)

#### 同步非阻塞

用户线程一直不停的调用`read`方法，如果数据还没有复制到内核空间则返回失败，直到数据到达内核空间。用户线程在等待数据从内核空间复制到用户空间的时间里一直是阻塞的，等数据到达用户空间才被唤醒。循环调用`read`方法的时候不阻塞。

![同步非阻塞](../../../../tomcat高性能拆解/20200810165207.png)

#### I/O 多路复用

用户线程的读取操作被划分为两步：

1. 用户线程先发起 `select` 调用，主要就是询问内核数据转备好了没？当内核把数据准备好了就执行第二步。
2. 用户线程再发起 `read` 调用，在等待内核把数据从内核空间复制到用户空间的时间里，发起 read 线程是阻塞的。

为何叫 I/O 多路复用，核心主要就是：一次 `select` 调用可以向内核查询多个**数据通道(Channel)**的状态，因此叫多路复用。

![I/O 多路复用](../../../../tomcat高性能拆解/20200810165847.png)

#### 异步 I/O

用户线程执行 read 调用的时候会注册一个回调函数， read 调用立即返回，不会阻塞线程，在等待内核将数据准备好以后，再调用刚刚注册的回调函数处理数据，在整个过程中用户线程一直没有阻塞。

![异步 I/O](../../../../tomcat高性能拆解/20200810170158.png)

### Tomcat NioEndpoint

Tomcat 的 NioEndpoit 组件实际上就是实现了 I/O 多路复用模型，正式因为这个并发能力才足够优秀。让我们一起窥探下 Tomcat NioEndpoint 的设计原理。

对于 Java 的多路复用器的使用，无非是两步：

1. 创建一个 Seletor，在它身上注册各种感兴趣的事件，然后调用 select 方法，等待感兴趣的事情发生。

2. 感兴趣的事情发生了，比如可以读了，这时便创建一个新的线程从 Channel 中读数据。

Tomcat 的 NioEndpoint 组件虽然实现比较复杂，但基本原理就是上面两步。我们先来看看它有哪些组件，它一共包含 LimitLatch、Acceptor、Poller、SocketProcessor 和 Executor 共 5 个组件，它们的工作过程如下图所示：

![NioEndPoint](../../../../tomcat高性能拆解/NioEndPoint.jpg)

正是由于使用了 I/O 多路复用，Poller 内部本质就是持有 Java Selector 检测 channel 的 I/O 时间，当数据可读写的时候创建 SocketProcessor 任务丢到线程池执行，也就是少量线程监听读写事件，接着专属的线程池执行读写，提高性能。

### 自定义线程池模型

为了提高处理能力和并发度， Web 容器通常会把处理请求的工作放在线程池来处理， Tomcat 拓展了 Java 原生的线程池来提升并发需求，在进入 Tomcat 线程池原理之前，我们先回顾下 Java 线程池原理。

#### Java 线程池

简单的说，Java 线程池里内部维护一个线程数组和一个任务队列，当任务处理不过来的时，就把任务放到队列里慢慢处理。

##### ThreadPoolExecutor

来窥探线程池核心类的构造函数，我们需要理解每一个参数的作用，才能理解线程池的工作原理。

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        ......
    }
```

- corePoolSize：保留在池中的线程数，即使它们空闲，除非设置了 allowCoreThreadTimeOut，不然不会关闭。
- maximumPoolSize：队列满后池中允许的最大线程数。
- keepAliveTime、TimeUnit：如果线程数大于核心数，多余的空闲线程的保持的最长时间会被销毁。unit 是 keepAliveTime 参数的时间单位。当设置 `allowCoreThreadTimeOut(true)` 时，线程池中 corePoolSize 范围内的线程空闲时间达到 keepAliveTime 也将回收。
- workQueue：当线程数达到 **corePoolSize** 后，新增的任务就放到工作队列 **workQueue** 里，而线程池中的线程则努力地从 **workQueue** 里拉活来干，也就是调用 poll 方法来获取任务。
- ThreadFactory：创建线程的工厂，比如设置是否是后台线程、线程名等。
- RejectedExecutionHandler：拒绝策略，处理程序因为达到了线程界限和队列容量执行拒绝策略。也可以自定义拒绝策略，只要实现 `RejectedExecutionHandler` 即可。默认的拒绝策略：`AbortPolicy` 拒绝任务并抛出 `RejectedExecutionException` 异常；`CallerRunsPolicy` 提交该任务的线程执行；``

来分析下每个参数之间的关系：

提交新任务的时候，如果线程池数 < corePoolSize，则创建新的线程池执行任务，当线程数 = corePoolSize 时，新的任务就会被放到工作队列 workQueue 中，线程池中的线程尽量从队列里取任务来执行。

如果任务很多，workQueue 满了，且 当前线程数 < maximumPoolSize 时则临时创建线程执行任务，如果总线程数量超过 maximumPoolSize，则不再创建线程，而是执行拒绝策略。`DiscardPolicy` 什么都不做直接丢弃任务；`DiscardOldestPolicy` 丢弃最旧的未处理程序;

具体执行流程如下图所示：

![线程池执行流程](../../../../tomcat高性能拆解/20200810105556.png)

#### Tomcat 线程池

定制版的 ThreadPoolExecutor，继承了 java.util.concurrent.ThreadPoolExecutor。 对于线程池有两个很关键的参数：

- 线程个数。
- 队列长度。

Tomcat 必然需要限定想着两个参数不然在高并发场景下可能导致 CPU 和内存有资源耗尽的风险。继承了 与 java.util.concurrent.ThreadPoolExecutor 相同，但实现的效率更高。

其构造方法如下，跟 Java 官方的如出一辙

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, handler);
        prestartAllCoreThreads();
    }
```

在 Tomcat 中控制线程池的组件是 `StandardThreadExecutor` , 也是实现了生命周期接口，下面是启动线程池的代码

```java
    @Override
    protected void startInternal() throws LifecycleException {
        // 自定义任务队列
        taskqueue = new TaskQueue(maxQueueSize);
        // 自定义线程工厂
        TaskThreadFactory tf = new TaskThreadFactory(namePrefix,daemon,getThreadPriority());
       // 创建定制版线程池
        executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), maxIdleTime, TimeUnit.MILLISECONDS,taskqueue, tf);
        executor.setThreadRenewalDelay(threadRenewalDelay);
        if (prestartminSpareThreads) {
            executor.prestartAllCoreThreads();
        }
        taskqueue.setParent(executor);
        // 观察者模式，发布启动事件
        setState(LifecycleState.STARTING);
    }
```

其中的关键点在于：

1. Tomcat 有自己的定制版任务队列和线程工厂，并且可以限制任务队列的长度，它的最大长度是 maxQueueSize。
2. Tomcat 对线程数也有限制，设置了核心线程数（minSpareThreads）和最大线程池数（maxThreads）。

除此之外， Tomcat 在官方原有基础上重新定义了自己的线程池处理流程，原生的处理流程上文已经说过。

- 前 corePoolSize 个任务时，来一个任务就创建一个新线程。
- 还有任务提交，直接放到队列，队列满了，但是没有达到最大线程池数则创建临时线程救火。
- 线程总线数达到 maximumPoolSize ，直接执行拒绝策略。

Tomcat 线程池扩展了原生的 ThreadPoolExecutor，通过重写 execute 方法实现了自己的任务处理逻辑：

- 前 corePoolSize 个任务时，来一个任务就创建一个新线程。
- 还有任务提交，直接放到队列，队列满了，但是没有达到最大线程池数则创建临时线程救火。
- **线程总线数达到 maximumPoolSize ，继续尝试把任务放到队列中。如果队列也满了，插入任务失败，才执行拒绝策略。**

**最大的差别在于 Tomcat 在线程总数达到最大数时，不是立即执行拒绝策略，而是再尝试向任务队列添加任务，添加失败后再执行拒绝策略。**

代码如下所示:

```java
    public void execute(Runnable command, long timeout, TimeUnit unit) {
       // 记录提交任务数 +1
        submittedCount.incrementAndGet();
        try {
            // 调用 java 原生线程池来执行任务，当原生抛出拒绝策略
            super.execute(command);
        } catch (RejectedExecutionException rx) {
          //总线程数达到 maximumPoolSize，Java 原生会执行拒绝策略
            if (super.getQueue() instanceof TaskQueue) {
                final TaskQueue queue = (TaskQueue)super.getQueue();
                try {
                    // 尝试把任务放入队列中
                    if (!queue.force(command, timeout, unit)) {
                        submittedCount.decrementAndGet();
                      // 队列还是满的，插入失败则执行拒绝策略
                        throw new RejectedExecutionException("Queue capacity is full.");
                    }
                } catch (InterruptedException x) {
                    submittedCount.decrementAndGet();
                    throw new RejectedExecutionException(x);
                }
            } else {
              // 提交任务书 -1
                submittedCount.decrementAndGet();
                throw rx;
            }

        }
    }
```

Tomcat 线程池是用 submittedCount 来维护已经提交到了线程池，这跟 Tomcat 的定制版的任务队列有关。Tomcat 的任务队列 TaskQueue 扩展了 Java 中的 LinkedBlockingQueue，我们知道 LinkedBlockingQueue 默认情况下长度是没有限制的，除非给它一个 capacity。因此 Tomcat 给了它一个 capacity，TaskQueue 的构造函数中有个整型的参数 capacity，TaskQueue 将 capacity 传给父类 LinkedBlockingQueue 的构造函数，防止无限添加任务导致内存溢出。而且默认是无限制，就会导致当前线程数达到核心线程数之后，再来任务的话线程池会把任务添加到任务队列，并且总是会成功，这样永远不会有机会创建新线程了。

为了解决这个问题，TaskQueue 重写了 LinkedBlockingQueue 的 offer 方法，在合适的时机返回 false，返回 false 表示任务添加失败，这时线程池会创建新的线程。

```java
public class TaskQueue extends LinkedBlockingQueue<Runnable> {

  ...
   @Override
  // 线程池调用任务队列的方法时，当前线程数肯定已经大于核心线程数了
  public boolean offer(Runnable o) {

      // 如果线程数已经到了最大值，不能创建新线程了，只能把任务添加到任务队列。
      if (parent.getPoolSize() == parent.getMaximumPoolSize())
          return super.offer(o);

      // 执行到这里，表明当前线程数大于核心线程数，并且小于最大线程数。
      // 表明是可以创建新线程的，那到底要不要创建呢？分两种情况：

      //1. 如果已提交的任务数小于当前线程数，表示还有空闲线程，无需创建新线程
      if (parent.getSubmittedCount()<=(parent.getPoolSize()))
          return super.offer(o);

      //2. 如果已提交的任务数大于当前线程数，线程不够用了，返回 false 去创建新线程
      if (parent.getPoolSize()<parent.getMaximumPoolSize())
          return false;

      // 默认情况下总是把任务添加到任务队列
      return super.offer(o);
  }

}
```

只有当前线程数大于核心线程数、小于最大线程数，并且已提交的任务个数大于当前线程数时，也就是说线程不够用了，但是线程数又没达到极限，才会去创建新的线程。这就是为什么 Tomcat 需要维护已提交任务数这个变量，它的目的就是**在任务队列的长度无限制的情况下，让线程池有机会创建新的线程**。可以通过设置 maxQueueSize 参数来限制任务队列的长度。

## 性能优化

### 线程池调优

跟 I/O 模型紧密相关的是线程池，线程池的调优就是设置合理的线程池参数。我们先来看看 Tomcat 线程池中有哪些关键参数：

| 参数                   | 详情                                                         |
| ---------------------- | ------------------------------------------------------------ |
| threadPriority         | 线程优先级，默认是 5                                         |
| daemon                 | 是否是 后台线程，默认 true                                   |
| namePrefix             | 线程名前缀                                                   |
| maxThreads             | 最大线程数，默认 200                                         |
| minSpareThreads        | 最小线程数（空闲超过一定时间会被回收），默认 25              |
| maxIdleTime            | 线程最大空闲时间，超过该时间的会被回收，直到只有 minSpareThreads 个。默认是 1 分钟 |
| maxQueueSize           | 任务队列最大长度                                             |
| prestartAllCoreThreads | 是否在线程池启动的时候就创建 minSpareThreads 个线程，默认是 fasle |

这里面最核心的就是如何确定 maxThreads 的值，如果这个参数设置小了，Tomcat 会发生线程饥饿，并且请求的处理会在队列中排队等待，导致响应时间变长；如果 maxThreads 参数值过大，同样也会有问题，因为服务器的 CPU 的核数有限，线程数太多会导致线程在 CPU 上来回切换，耗费大量的切换开销。

**线程 I/O 时间与 CPU 时间**

至此我们又得到一个线程池个数的计算公式，假设服务器是单核的：

**线程池大小 = （线程 I/O 阻塞时间 + 线程 CPU 时间 ）/ 线程 CPU 时间**

其中：线程 I/O 阻塞时间 + 线程 CPU 时间 = 平均请求处理时间。

### Tomcat 内存溢出的原因分析及调优

JVM 在抛出 java.lang.OutOfMemoryError 时，除了会打印出一行描述信息，还会打印堆栈跟踪，因此我们可以通过这些信息来找到导致异常的原因。在寻找原因前，我们先来看看有哪些因素会导致 OutOfMemoryError，其中内存泄漏是导致 OutOfMemoryError 的一个比较常见的原因。

其实调优很多时候都是在找系统瓶颈，假如有个状况：系统响应比较慢，但 CPU 的用率不高，内存有所增加，通过分析 Heap Dump 发现大量请求堆积在线程池的队列中，请问这种情况下应该怎么办呢？可能是请求处理时间太长，去排查是不是访问数据库或者外部应用遇到了延迟。

#### java.lang.OutOfMemoryError: Java heap space

当 JVM 无法在堆中分配对象的会抛出此异常，一般有以下原因：

1. 内存泄漏：本该回收的对象呗程序一直持有引用导致对象无法被回收，比如在线程池中使用 ThreadLocal、对象池、内存池。为了找到内存泄漏点，我们通过 jmap 工具生成 Heap Dump，再利用 MAT 分析找到内存泄漏点。`jmap -dump:live,format=b,file=filename.bin pid`

2. 内存不足：我们设置的堆大小对于应用程序来说不够，修改 JVM 参数调整堆大小，比如 -Xms256m -Xmx2048m。
3. finalize 方法的过度使用。如果我们想在 Java 类实例被 GC 之前执行一些逻辑，比如清理对象持有的资源，可以在 Java 类中定义 finalize 方法，这样 JVM GC 不会立即回收这些对象实例，而是将对象实例添加到一个叫“java.lang.ref.Finalizer.ReferenceQueue”的队列中，执行对象的 finalize 方法，之后才会回收这些对象。Finalizer 线程会和主线程竞争 CPU 资源，但由于优先级低，所以处理速度跟不上主线程创建对象的速度，因此 ReferenceQueue 队列中的对象就越来越多，最终会抛出 OutOfMemoryError。解决办法是尽量不要给 Java 类定义 finalize 方法。

#### java.lang.OutOfMemoryError: GC overhead limit exceeded

垃圾收集器持续运行，但是效率很低几乎没有回收内存。比如 Java 进程花费超过 96％的 CPU 时间来进行一次 GC，但是回收的内存少于 3％的 JVM 堆，并且连续 5 次 GC 都是这种情况，就会抛出 OutOfMemoryError。

这个问题 IDE 解决方法就是查看 GC 日志或者生成 Heap Dump，先确认是否是内存溢出，不是的话可以尝试增加堆大小。可以通过如下 JVM 启动参数打印 GC 日志:

```shell
-verbose:gc //在控制台输出GC情况
-XX:+PrintGCDetails  //在控制台输出详细的GC情况
-Xloggc: filepath  //将GC日志输出到指定文件中
```

比如 可以使用 `java -verbose:gc -Xloggc:gc.log -XX:+PrintGCDetails -jar xxx.jar` 记录 GC 日志，通过 GCViewer 工具查看 GC 日志，用 GCViewer 打开产生的 gc.log 分析垃圾回收情况。

#### java.lang.OutOfMemoryError: Requested array size exceeds VM limit

抛出这种异常的原因是“请求的数组大小超过 JVM 限制”，应用程序尝试分配一个超大的数组。比如程序尝试分配 128M 的数组，但是堆最大 100M，一般这个也是配置问题，有可能 JVM 堆设置太小，也有可能是程序的 bug，是不是创建了超大数组。

#### java.lang.OutOfMemoryError: MetaSpace

JVM 元空间的内存在本地内存中分配，但是它的大小受参数 MaxMetaSpaceSize 的限制。当元空间大小超过 MaxMetaSpaceSize 时，JVM 将抛出带有 MetaSpace 字样的 OutOfMemoryError。解决办法是加大 MaxMetaSpaceSize 参数的值。

#### java.lang.OutOfMemoryError: Request size bytes for reason. Out of swap space

当本地堆内存分配失败或者本地内存快要耗尽时，Java HotSpot VM 代码会抛出这个异常，VM 会触发“致命错误处理机制”，它会生成“致命错误”日志文件，其中包含崩溃时线程、进程和操作系统的有用信息。如果碰到此类型的 OutOfMemoryError，你需要根据 JVM 抛出的错误信息来进行诊断；或者使用操作系统提供的 DTrace 工具来跟踪系统调用，看看是什么样的程序代码在不断地分配本地内存。

#### java.lang.OutOfMemoryError: Unable to create native threads

1. Java 程序向 JVM 请求创建一个新的 Java 线程。
2. JVM 本地代码（Native Code）代理该请求，通过调用操作系统 API 去创建一个操作系统级别的线程 Native Thread。
3. 操作系统尝试创建一个新的 Native Thread，需要同时分配一些内存给该线程，每一个 Native Thread 都有一个线程栈，线程栈的大小由 JVM 参数`-Xss`决定。
4. 由于各种原因，操作系统创建新的线程可能会失败，下面会详细谈到。
5. JVM 抛出“java.lang.OutOfMemoryError: Unable to create new native thread”错误。

这里只是概述场景，对于生产在线排查后续会陆续推出，受限于篇幅不再展开。**关注「码哥字节」给你硬货来啃！**

## 总结

回顾 Tomcat 总结架构设计，详细拆解 Tomcat 如何处理高并发连接设计。并且分享了如何高效阅读开源框架源码思路，设计模式、并发编程基础是重中之重，读者朋友可以翻阅历史「码哥字节」的历史文章学习。

### 推荐阅读

[Tomcat 架构解析到工作借鉴](https://mp.weixin.qq.com/s/fU5Jj9tQvNTjRiT9grm6RA)

[设计模式专辑](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU3NDkwMjAyOQ==&action=getalbum&album_id=1351452628844363777&subscene=159&subscene=&scenenote=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FAr_FZ6ck_WEjr9ia9bjmcQ#wechat_redirect)

[并发编程实战](http://mp.weixin.qq.com/mp/homepage?__biz=MzU3NDkwMjAyOQ==&hid=7&sn=660e8728f9eb24248cc7953796dbe773&scene=18#wechat_redirect)

拆解 Tomcat 核心组件，去体会 Tomcat 如何面向接口设计、落实单一职责的设计哲学思想。接着概括了 连接器涉及到的 I/O 模型，并对不同的 I/O 模型进行了详解，接着看 Tomcat 如何实现 NIO，如何自定义线程池以及队列实现高并发设计，最后简单分享常见的 OOM 场景以及解决思路，限于篇幅不再详细展开，关注「码哥字节」后续会分享各种线上故障排查调优思路，敬请期待…...

有任何疑问或者计数探讨可以**加个人微信：MageByte1024**，一起学习进步。

也可以通过公众号菜单加入技术群，里面有**阿里、腾讯**的大佬。

![码哥字节](https://magebyte.oss-cn-shenzhen.aliyuncs.com/wechat/%E7%A0%81%E5%93%A5%E5%AD%97%E8%8A%82%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

**编写文章不易，如果阅读后觉得有用，希望关注「码哥字节」公众号，点击「分享」、「点赞」、「在看」是最大的鼓励。**