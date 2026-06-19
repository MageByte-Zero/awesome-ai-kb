# Redis 6.X Sentinel 搭建

码哥带大家完成在 CentOS 7 中安装 Redis 6.x 教程。在学习 Redis 之前，我们需要先搭建一套哨兵环境。机器有限，实现目标是一台机器上搭建 6 个节点，构成**一主两从三哨兵**集群模式。

## 下载解压

可直接到 Redis 官网[下载](https://redis.io/download)最新稳定包，地址：https://redis.io/download。或者使用 命令：`sudo wget http://download.redis.io/releases/redis-6.0.9.tar.gz ` 下载安装包.。

1. 码哥统一把软件包放在 `/opt/soft` 目录下，并创建目录 `mkdir redisSentinel`。在 redisSentinel 目录下执行 `mkdir 6479 6480 6481 26379 26380 26381` 6479 6480 6481分别对应 Redis 主从节点 redis.conf 配置模板。
2. `tar -zxf redis-6.0.9.tar.gz -C redisSentinel`解压到 redisSentinel 目录中。

## make 编译

在编译之前我们需要确认 gcc 版本，自 redis 6.0.0 之后，编译 redis 需要支持 C11 特性，C11 特性在 4.9 中被引入。Centos 7 默认 gcc 版本为 4.8.5，所以需要升级gcc版本。

否则在编译过程中会报错。

**解决方式**

```shell
yum -y install gcc gcc-c++ make tcl
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
```

升级之后便可解决 make 报错问题。

**注意：scl命令启用只是临时的，退出xshell或者重启就会恢复到原来的gcc版本。如果要长期生效的话，执行如下 sudo echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile**。

`cd /opt/soft/redisSentinel/redis-6.0.9` 切换到目录执行 `make` 。

编译完成使用 make install 对 redis 进行安装 ，命令：`sudo make install`。

##  主从复制

将 redis.conf 复制三份到 6479 6480 6481 目录下，并修改配置：

**Master**

```shell
# master 端口
port 6479
# 让 Redis 可以跨网访问
bind 172.16.90.152
# 后台执行
daemonize yes
pidfile /var/run/redis_6479.pid
```

**slave**

主要在于端口号不同，分别是 6480、6481，并且在末尾添加 `replicaof 172.16.90.152 6479`

```shell
# master 端口
port 6480
# 让 Redis 可以跨网访问
bind 172.16.90.152
# 后台执行
daemonize yes
# 指定 masterip master port
replicaof 172.16.90.152 6479
```

**分别启动 Redis**

通过 redis-server 启动主从节点。

```shell
./redis-6.0.9/src/redis-server redis-6479/redis.conf
./redis-6.0.9/src/redis-server redis-6480/redis.conf
./redis-6.0.9/src/redis-server redis-6481/redis.conf
```

**检查集群状态**

```shell
./redis-6.0.9/src/redis-cli -p 6479 info Replication
```

**配置哨兵集群**

将哨兵配置文件分别复制到 `sentinel26380 sentinel26381 sentinel26382`，需要注意的是每个文件的端口配置以及 `sentinel monitor mymaster 172.16.90.152 6479 2` 中最后的数字 2，哨兵集群汇总每个节点必须一致。

分别修改这三个配置文件：

```shell
# 绑定IP
bind 0.0.0.0
# 后台运行
daemonize yes
# 默认yes，没指定密码或者指定IP的情况下，外网无法访问
protected-mode no
# 哨兵的端口，客户端通过这个端口来发现redis
port 26380
# 这个文件会自动生成（如果同一台服务器上启动，注意要修改为不同的端口）
pidfile /var/run/redis-sentinel-26380.pid
# sentinel监控的master的名字叫做mymaster,初始地址为 127.0.0.1 6380,2代表两个及以上哨兵认定为死亡，才认为是真的死亡
sentinel monitor mymaster 172.16.90.152 6479 2
```

**启动哨兵集群**

```sh
./redis-6.0.9/src/redis-sentinel sentinel26380/sentinel.conf
./redis-6.0.9/src/redis-sentinel sentinel26381/sentinel.conf
./redis-6.0.9/src/redis-sentinel sentinel26382/sentinel.conf
```

查看 sentinel 监控的 master-slave 信息:

```shell
redis-cli -h 192.168.31.220 -p 26380
sentinel master mymaster
SENTINEL replicas mymaster
SENTINEL sentinels mymaster
```

**测试故障自动转移**

```shell
redis-cli -p 6480 DEBUG sleep 30
```

再次检查当前 master 地址，这次将得到不同的响应：

```shell
SENTINEL get-master-addr-by-name mymaster
```


