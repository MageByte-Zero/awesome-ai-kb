# Tomcat 帝国高层都在干嘛？

在[Tomcat 架构解析到设计思想借鉴](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247487707&idx=1&sn=7657d73cc944fae265ad6ce5f25cad1d&scene=19#wechat_redirect)中我们学到 Tomcat 的总体架构，学会从宏观上怎么去设计一个复杂系统，怎么设计顶层模块，以及模块之间的关系；

Tomcat 实现的 2 个核心功能：

- 处理 `Socket` 连接，负责网络字节流与 `Request` 和 `Response` 对象的转化。
- 加载并管理 `Servlet` ，以及处理具体的 `Request` 请求。

**所以 Tomcat 设计了两个核心组件连接器（Connector）和容器（Container），连接器负责对外交流，容器负责内部处理。**

![Tomcat整体架构](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tomcat/20200407171031.png)

本篇作为 [Tomcat](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzkzMDI1NjcyOQ==&action=getalbum&album_id=1920651737065930757&scene=173&from_msgid=2247487707&from_itemidx=1&count=3&nolastread=1#wechat_redirect) 系列的第三篇，带大家体会 Tomcat 帝国是如何构建的？每个组件如何分工协作？**连接器**和**容器**是如何被启动和管理的？

Tomcat 启动流程：`startup.sh -> catalina.sh start ->java -jar org.apache.catalina.startup.Bootstrap.main()`

![Tomcat 启动流程](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tomcat/20200725214643.png)

Bootstrap、Catalina、Server、Service 都承担了什么责任？

## Bootstrap

当执行 `startup.sh` 脚本的时候，就会启动一个 JVM 运行 Tomcat 的启动类 `Bootstrap`。

先看下他的成员变量