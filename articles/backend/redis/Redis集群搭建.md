# Redis 6.X Cluster 集群搭建

码哥带大家完成在 CentOS 7 中安装 Redis 6.x 教程。在学习 Redis 之前，我们需要先搭建一套集群环境。机器有限，实现目标是一台机器上搭建 6 个节点，构成一个三主三从集群模式。

## 下载解压

可直接到 Redis 官网[下载](https://redis.io/download)最新稳定包，地址：https://redis.io/download。或者使用 命令：`sudo wget http://download.redis.io/releases/redis-6.0.9.tar.gz ` 下载安装包.。

1. 码哥统一把软件包放在 `/opt/soft` 目录下，并创建目录 `mkdir redisCluster`用于放置集群配置文件。在 redisCluster 目录下执行 `mkdir 7000 7001 7002 7003 7004 7005` 创建 6 个目录分别对应每个节点 redis.conf 配置模板。
2. `tar -zxf redis-6.0.9.tar.gz -C redisCluster` 解压到 redisCluster 目录中。

## make 编译

在编译之前我们需要确认 gcc 版本，自 redis 6.0.0 之后，编译 redis 需要支持 C11 特性，C11 特性在 4.9 中被引入。Centos 7 默认 gcc 版本为 4.8.5，所以需要升级gcc版本。

**编译错误**

否则在编译过程中会遇到如下错误日志：

```
In file included from server.c:31:0:
server.c:4999:59: error: ‘struct redisServer’ has no member named ‘cluster’
             (server.cluster_enabled && nodeIsMaster(server.cluster->myself)));
                                                           ^
cluster.h:58:27: note: in definition of macro ‘nodeIsMaster’
 #define nodeIsMaster(n) ((n)->flags & CLUSTER_NODE_MASTER)
                           ^
server.c: In function ‘main’:
server.c:5047:11: error: ‘struct redisServer’ has no member named ‘sentinel_mode’
     server.sentinel_mode = checkForSentinelMode(argc,argv);
           ^
server.c:5064:15: error: ‘struct redisServer’ has no member named ‘sentinel_mode’
     if (server.sentinel_mode) {
               ^
server.c:5131:19: error: ‘struct redisServer’ has no member named ‘sentinel_mode’
         if (server.sentinel_mode && configfile && *configfile == '-') {
                   ^
server.c:5153:168: error: ‘struct redisServer’ has no member named ‘sentinel_mode’
         serverLog(LL_WARNING, "Warning: no config file specified, using the default config. In order to specify a config file use %s /path/to/%s.conf", argv[0], server.sentinel_mode ? "sentinel" : "redis");
                                                                                                                                                                        ^
server.c:5158:11: error: ‘struct redisServer’ has no member named ‘supervised’
     server.supervised = redisIsSupervised(server.supervised_mode);
           ^
server.c:5158:49: error: ‘struct redisServer’ has no member named ‘supervised_mode’
     server.supervised = redisIsSupervised(server.supervised_mode);
                                                 ^
server.c:5159:28: error: ‘struct redisServer’ has no member named ‘daemonize’
     int background = server.daemonize && !server.supervised;
                            ^
server.c:5159:49: error: ‘struct redisServer’ has no member named ‘supervised’
     int background = server.daemonize && !server.supervised;
                                                 ^
server.c:5163:29: error: ‘struct redisServer’ has no member named ‘pidfile’
     if (background || server.pidfile) createPidFile();
                             ^
server.c:5168:16: error: ‘struct redisServer’ has no member named ‘sentinel_mode’
     if (!server.sentinel_mode) {
                ^
server.c:5178:19: error: ‘struct redisServer’ has no member named ‘cluster_enabled’
         if (server.cluster_enabled) {
                   ^
server.c:5186:19: error: ‘struct redisServer’ has no member named ‘ipfd_count’
         if (server.ipfd_count > 0 || server.tlsfd_count > 0)
                   ^
server.c:5186:44: error: ‘struct redisServer’ has no member named ‘tlsfd_count’
         if (server.ipfd_count > 0 || server.tlsfd_count > 0)
                                            ^
server.c:5188:19: error: ‘struct redisServer’ has no member named ‘sofd’
         if (server.sofd > 0)
                   ^
server.c:5189:94: error: ‘struct redisServer’ has no member named ‘unixsocket’
             serverLog(LL_NOTICE,"The server is now ready to accept connections at %s", server.unixsocket);
                                                                                              ^
server.c:5190:19: error: ‘struct redisServer’ has no member named ‘supervised_mode’
         if (server.supervised_mode == SUPERVISED_SYSTEMD) {
                   ^
server.c:5191:24: error: ‘struct redisServer’ has no member named ‘masterhost’
             if (!server.masterhost) {
                        ^
server.c:5201:19: error: ‘struct redisServer’ has no member named ‘supervised_mode’
         if (server.supervised_mode == SUPERVISED_SYSTEMD) {
                   ^
server.c:5208:15: error: ‘struct redisServer’ has no member named ‘maxmemory’
     if (server.maxmemory > 0 && server.maxmemory < 1024*1024) {
               ^
server.c:5208:39: error: ‘struct redisServer’ has no member named ‘maxmemory’
     if (server.maxmemory > 0 && server.maxmemory < 1024*1024) {
                                       ^
server.c:5209:176: error: ‘struct redisServer’ has no member named ‘maxmemory’
         serverLog(LL_WARNING,"WARNING: You specified a maxmemory value that is less than 1MB (current value is %llu bytes). Are you sure this is what you really want?", server.maxmemory);
                                                                                                                                                                                ^
server.c:5212:31: error: ‘struct redisServer’ has no member named ‘server_cpulist’
     redisSetCpuAffinity(server.server_cpulist);
                               ^
server.c: In function ‘hasActiveChildProcess’:
server.c:1480:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
server.c: In function ‘allPersistenceDisabled’:
server.c:1486:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
server.c: In function ‘writeCommandsDeniedByDiskError’:
server.c:3826:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
server.c: In function ‘iAmMaster’:
server.c:5000:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
 ....

```

**解决方式**

```shell
yum -y install gcc gcc-c++ make tcl
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
```

升级之后便可解决 make 报错问题。

**注意：scl命令启用只是临时的，退出xshell或者重启就会恢复到原来的gcc版本。如果要长期生效的话，执行如下 sudo echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile**。

`cd /opt/soft/redisCluster/redis-6.0.9` 切换到目录执行 `make` 。

编译完成使用 make install 对 redis 进行安装 ，命令：`sudo make install`。

##  修改配置文件

`cd /opt/soft/redisCluster/redis-6.0.9` 将 redis.conf 分别复制到 7000 7001 7002 7003 7004 目录中。

**分别修改  6 个 redis.conf**

```shell
## 7000-7005端口
port 7000
## 后台启动
daemonize yes
## 如果是在单机模拟集群必须指定bind的IP，如果不修改ip的话使用程序连接集群会报错
bind 192.168.221.150
## 开启redis-cluster集群
cluster-enabled yes
## 每个实例还包含存储此节点配置的文件的路径，默认情况下为nodes.conf，自动创建
cluster-config-file nodes_7000.conf
## 超时
cluster-node-timeout 500
## 开启aof
appendonly yes
 
#注释cluster集群下不允许复制。
#replicaof 127.0.0.1 9000
#关闭保护模式,如果开启需要设置密码，比较繁琐，可根据自己的需求来
protected-mode no
```

每个配置文件只需要修改 **port 和 cluster-config-file** 就可以了。

## 启动节点并创建集群

**启动节点**

进入 redisCluster 目录，执行指令依次启动每个节点。`redis-6.0.9/src/redis-server 700x/redis.conf` 注意指定每个节点配置文件，**如果不指定配置文件会默认使用src下的配置**。

**创建集群**

进入任意一个节点，执行以下指令创建集群

指令如下：

```shell
redis-6.0.9/src/redis-cli --cluster create 172.16.90.152:7000 172.16.90.152:7001 172.16.90.152:7002 172.16.90.152:7003 172.16.90.152:7004 172.16.90.152:7005 --cluster-replicas 1
```

集群参数解释：

1. **cluster-replicas 1：表示希望为集群中的每个主节点创建一个从节点(一主一从)。**
2. **cluster-replicas 2：表示希望为集群中的每个主节点创建两个从节点(一主二从)。**

控制台响应：

```
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.16.90.152:7004 to 172.16.90.152:7000
Adding replica 172.16.90.152:7005 to 172.16.90.152:7001
Adding replica 172.16.90.152:7003 to 172.16.90.152:7002
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 06c56f5a6a4436108fae931be499465985141d39 172.16.90.152:7000
   slots:[0-5460] (5461 slots) master
M: 0ab7c9efd97319d94a8ea52452ec58f7708d812d 172.16.90.152:7001
   slots:[5461-10922] (5462 slots) master
M: 096f076d99363270c02785a2fb298e2ee65d3f07 172.16.90.152:7002
   slots:[10923-16383] (5461 slots) master
S: 69d621060295eb433af3e34e702142df0fd4d73d 172.16.90.152:7003
   replicates 06c56f5a6a4436108fae931be499465985141d39
S: 1d37df0aa0e2310aedb5a380f95cc818256003f8 172.16.90.152:7004
   replicates 0ab7c9efd97319d94a8ea52452ec58f7708d812d
S: d9204f6da875a4b2522c5fa25d9e6c1f95cf51ea 172.16.90.152:7005
   replicates 096f076d99363270c02785a2fb298e2ee65d3f07
Can I set the above configuration? (type 'yes' to accept):
```

Can I set the above configuration? (type 'yes' to accept): 询问是否确认节点 slots 分配方案， 我们输入 ‘yes’。

```
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 172.16.90.152:7000)
M: 06c56f5a6a4436108fae931be499465985141d39 172.16.90.152:7000
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 1d37df0aa0e2310aedb5a380f95cc818256003f8 172.16.90.152:7004
   slots: (0 slots) slave
   replicates 0ab7c9efd97319d94a8ea52452ec58f7708d812d
M: 0ab7c9efd97319d94a8ea52452ec58f7708d812d 172.16.90.152:7001
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: d9204f6da875a4b2522c5fa25d9e6c1f95cf51ea 172.16.90.152:7005
   slots: (0 slots) slave
   replicates 096f076d99363270c02785a2fb298e2ee65d3f07
S: 69d621060295eb433af3e34e702142df0fd4d73d 172.16.90.152:7003
   slots: (0 slots) slave
   replicates 06c56f5a6a4436108fae931be499465985141d39
M: 096f076d99363270c02785a2fb298e2ee65d3f07 172.16.90.152:7002
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

[OK] All 16384 slots covered.

到此完成！

## 查看集群状态

`redis-6.0.9/src/redis-cli --cluster check 172.16.90.152:7000`

## 注意事项

当使用 `redis-6.0.9/src/redis-cli --cluster create 172.16.90.152:7000 172.16.90.152:7001 172.16.90.152:7002 172.16.90.152:7003 172.16.90.152:7004 172.16.90.152:7005 --cluster-replicas 1` 创建集群以后，**一次创建，永久使用**。之后直接启动每个节点即可构建集群。

结束命令：`redis-6.0.9/src/redis-cli -c -h 192.168.124.23 -p 7004 shutdown`

**进入集群命令  redis-cli -c -h host -p prot 不带-c 参数进入的不是集群**