> 码老湿，上次你讲解了 [Redis 多线程模型](https://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&amp;mid=2247489917&amp;idx=1&amp;sn=6865c14ee10684c2766eba9bc0edd13a&amp;chksm=c27c5b4bf50bd25d6467e86df51fa7584009be05f2eca40eaa3aafdceb079e7b3855498774de&token=1844426616&lang=zh_CN#rd)，这次我想知道客户端缓存（Client side caching）技术，他的英文名叫： `Redis server-assisted client side caching` ，可以说说么？
> 我不是嫖客，看完我会点赞、再看、分享的。

别装逼了，还整英文，咋不上天，做人要说话算数哟，不然半夜尿裤子。在说这个之前，码哥先给读者送一段寄语作为开篇。

## 开篇寄语

不要吝啬你的赞美，如果别人做的很好，就给他正反馈，这也是一种利他。

另外，少关注用「赞美」投票的事物，而多关注用「交易」投票的事物。

判断一个人是否牛逼，不是看网上有多少人赞美他，而是看有多少人愿意跟他发生交易、赞赏、支付、下单。

**因为赞美太廉价，而愿意与他发生交易，才是真正的信任。**

# 为啥需要客户端缓存

`Redis` 的`Tracking Feature` 的实现代码在: `https://github.com/antirez/redis/blob/unstable/src/tracking.c`。

很多公司使用 Redis 做缓存系统，并且很好的提高了数据访问的性能，为了进一步应对热点数据，还是会在 Redis 的 Client 端缓存一部分热点数据，用来应对「吃瓜事件」。

比如，「[这该死的 996 福报](https://mp.weixin.qq.com/s/0A6tLuKnRLi29WA7A-daig)」、「吴亦凡之大方牢房」、「时间管理大师」、「[思聪舔我不得就锤我](https://mp.weixin.qq.com/s/vnFg3ICDBmAVy4Vwzmux9g)」、「吴秀波之谈恋爱么，能坐牢的那种」……

除了使用 Redis 缓存避免直接访问数据库以外，还会加更多的` cache` 层，比如采用 `Memcachced` 作为热点数据的本地缓存：

1. 先去 `Memcachced `中查询数据，命中直接返回。
2. `Memcachced` 未命中，则再从 Redis 查询，命中则返回数据，并在 `Memcachced` 保存这个数据。
3. Redis 未命中，则去 `MySQL`中查询，并依次设置到 Redis 和 `Memcachced`中。 

**访问本地内存的的性能必然比通过网络访问 Redis 快，所以这种模式可以极大地减少获取数据的延迟，并且可以减少 Redis 的负载，提高性能**。

1. 访问 Redis 获取数据，服务器响应。

![查询Redis](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/客户端缓存.png)

2. 使用客户端缓存，应用程序将获取的热门的数据存储在用用程序中，无需再次通过网络访问 Redis。

   ![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/Redis客户端缓存.png)

**应该缓存什么**

- 我们不应该缓存不断变化的键。
- 我们不该缓存很少请求的键。
- 我们希望缓存经常请求并以合理速率更改的键。对于没有稳定变化速度的例子，比如不断被`INCR`修改的全局计数器，就不应该缓存。

# 客户端缓存实现原理

> 码老湿， Redis 中的数据修改或者失效了，如何及时同步告知客户端失效了呢？自己实现也太复杂了。

Redis 实现的是一个服务端协助的客户端缓存，叫做`tracking`。客户端缓存的命令是:

```
CLIENT TRACKING ON|OFF [REDIRECT client-id] [PREFIX prefix] [BCAST] [OPTIN] [OPTOUT] [NOLOOP]
```

Redis 6.0 实现 `Tracking` 功能提供了两种模式解决这个问题，分别是使用RESP3 协议版本的**普通模式和广播模式**，以及使用 **RESP2 协议版本的转发**模式。

![来源于程序员厉小冰](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20210801205225.png)*



## 普通模式

当`tracking`开启时， `Redis `会「记住」每个客户端请求的 `key`，当 `key `的值发现变化时会发送失效信息给客户端 (`invalidation message`)。

失效信息可以通过 `RESP3 `协议发送给请求的客户端，或者转发给一个不同的连接 (支持 `RESP2 + Pub/Sub`) 的客户端。

- Server 端将 Client 访问的 key以及该 key 对应的客户端 ID 列表信息存储在全局唯一的表（TrackingTable），当表满了，回移除最老的记录，同时触发该记录已过期的通知给客户端。
- 每个 Redis 客户端又有一个唯一的数字 ID，TrackingTable 存储着每一个 Client ID，当连接断开后，清除该 ID 对应的记录。
- **TrackingTable 表**中记录的 Key 信息不考虑是哪个 database 的，虽然访问的是 db1 的 key，db2 同名 key 修改时会客户端收到过期提示，但这样做会减少系统的复杂性，以及表的存储数据量。

> 码老湿，可以说下这个 TrackingTable 原理么？

Redis 服务端使用 `TrackingTable `存储普通模式的客户端数据，它的数据类型是基数树 ( radix tree)。

基数树是针对稀疏的长整型数据查找的多叉搜索树，能快速且节省空间的完映射。

Redis 用它存储**键的指针**和**客户端 ID** 的映射关系。**因为键对象的指针就是内存地址，也就是长整型数据**。客户端缓存的相关操作就是对该数据的增删改查：

![图片来源-程序员厉小冰](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20210801224044.png)

**注意**

**服务端对于记录的 key 只会报告一次 invalidate 消息**，也就是说，服务端在给客户端发送过一次 invalidate 消息后，如果 key 再被修改，此时，服务端就不会再次给客户端发送 invalidate 消息。

**只有下次客户端再次执行只读命令被 track，才会进行下一次消息通知** 。

客户端默认不开启 track 模式，我们需要在获取执行指令之前执行开启命令：

```
CLIENT TRACKING ON|OFF
+OK
GET user:211
$3
公众号:码哥字节
```

## 广播模式(BCAST)

当广播模式 (broadcasting) 开启时，服务器不会记住给定客户端访问了哪些键，因此这种模式在服务器端根本不消耗任何内存。

在这个模式下，服务端会给客户端广播所有 key 的失效情况，如果 key 被频繁修改，服务端会发送大量的失效广播消息，这就会消耗大量的网络带宽资源。

所以，在实际应用中，我们设置让客户端注册只跟踪指定前缀的 key，当注册跟踪的 key 前缀匹配被修改，服务端就会把失效消息广播给所有关注这个 key前缀的客户端。

```
client tracking on bcast prefix user
```

这种监测带有前缀的 key 的广播模式，和我们对 key 的命名规范非常匹配。我们在实际应用时，会给同一业务下的 key 设置相同的业务名前缀，所以，我们就可以非常方便地使用广播模式。

![图片来源-程序员厉小冰](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20210801230434.png)

广播模式与普通模式类似，Redis 使用 `PrefixTable` 存储广播模式下的客户端数据，它存储**前缀字符串指针和(需要通知的 key 和客户端 ID)**的映射关系。

## 转发模式

普通模式与广播模式，需要客户端使用 RESP 3 协议，他是 Redis 6.0 新启用的协议。

对于使用 RESP 2 协议的客户端来说，实现客户端缓存则需要另一种模式：重定向模式（redirect）。

RESP 2 无法直接 PUSH 失效消息，所以 需要另一个支持 RESP 3 协议的客户端 告诉 Server 将失效消息通过 Pus/Sub 通知给 RESP 2 客户端。

在重定向模式下，想要获得失效消息通知的客户端，就需要执行订阅命令 SUBSCRIBE，专门订阅用于发送失效消息的频道 `_redis_:invalidate`。

同时，再使用另外一个客户端，执行 CLIENT TRACKING 命令，设置服务端将失效消息转发给使用 RESP 2 协议的客户端。

![图片来源-程序员厉小冰](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20210801232142.png)

假设客户端 B 想要获取失效消息，但是客户端 B 只支持 RESP 2 协议，客户端 A 支持 RESP 3 协议。我们可以分别在客户端 B 和 A 上执行 SUBSCRIBE 和 CLIENT TRACKING，如下所示：

```
//客户端B执行，客户端 B 的 ID 号是 606
SUBSCRIBE _redis_:invalidate

//客户端 A 执行
CLIENT TRACKING ON BCAST REDIRECT 606
```

B 客户端就可以通过 `_redis_:invalidate` 频道获取失效消息了。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/最终二维码.png)

**热门推荐**

[Redis 新特性篇：多线程模型解读](http://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247489917&idx=1&sn=6865c14ee10684c2766eba9bc0edd13a&chksm=c27c5b4bf50bd25d6467e86df51fa7584009be05f2eca40eaa3aafdceb079e7b3855498774de#rd)

[Redis 实战篇：通过 Geo 类型实现附近的人邂逅女神](http://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247489024&idx=1&sn=8ae92ff12bc8c1af1617dbfaf91cb1dd&chksm=c27c5436f50bdd20035974cc91a4b5c203f5c3224a83b80c55a3e795476ad2d872454120e6b2#rd)

[Redis 面霸篇：从高频问题透视核心原理](http://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247487999&idx=1&sn=c4c6b80d2d9592ae0a30d8b16b513bcd&chksm=c27c53c9f50bdadf61369a96841f8322bd27d8d2e2a31c5d78887928e155b974b6558c44e410#rd)

**参考文档**

[1]:https://redis.io/topics/client-side-caching
[2]:https://colobu.com/2020/05/02/redis-client-side-caching/
[3]:https://time.geekbang.org/column/article/310838
[4]:https://juejin.cn/post/6844904153827786760