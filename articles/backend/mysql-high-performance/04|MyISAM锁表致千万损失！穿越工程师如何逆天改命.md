我叫林渊，在一线城市「浅圳」打拼，在一家名叫「网讯」的互联网大厂工作，是一名资深后端架构师。

已经连续加班到 23:00 半年多才下班，现在已是亥时，push 完代码后拿出我在「拼夕夕」买的 「zippo」火机点上一支烟，看着它逐渐没有痕迹，空气中闻到一股股淡淡的味道。

我弹落的烟灰如此的黯然，黯然如我，思绪万千。我闭上眼睛就是天黑，一种撕裂的感觉。

我掐灭了烟头，又重新点上一支烟，沉浸在淡蓝色的烟雾中，是那么的温柔，那么的迷蒙，那么的深情。

Christina，我想起你了，发信息给你的手在键盘敲很轻，我给的思念很小心。

> 你说：我们不适合，每天 996，起得比鸡早，睡得比狗晚，每天忙忙碌碌。天天加班，肚子那么大，头发那么少，血糖高、尿素高、脂肪肝。

回到工位，我写下了一段愿望，希望世间再无 996，多金身材好，女朋友漂亮，左拥右抱….

或许是因为长期加班的缘故。忽然，我只觉心里难受，胸闷气短，眼前一片黑，我想要努力的睁开眼睛，可是却什么都看不见，逐渐听不见周边的声音……

当我新来睁开眼睛的时候，我看到办公桌的电脑长这个样子。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202411101937255.png)

啥情况，我猝死了还是穿越了？还在我一脸懵逼的时候……

## 初遇 MyISAM

**2003 年冬夜，杭州某电商公司机房**

空调轰鸣声中，林渊盯着监控屏上飙红的 QPS 曲线，耳边传来刺耳的警报声。

> “订单接口全挂了！用户投诉电话被打爆！”运维组长老王猛砸键盘屏幕上赫然是经典的 MyISAM 报错：

```sql
ERROR 1146 (42S02): Table './order_db/orders' is locked
```

> 林渊（内心 OS）：“这个年代的 MySQL 居然还在用 MyISAM...是时候展现真正的技术了！”

## **MyISAM 表级锁的致命陷阱**

**问题现场还原**：

```sql
-- 会话1（长事务）
UPDATE orders SET status=2 WHERE user_id=100; -- 耗时30秒

-- 会话2（并发请求）
SELECT * FROM orders WHERE create_time > '2003-12-12'; -- 被阻塞！
```

流程如下：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202503292214277.png)

通过`SHOW PROCESSLIST`可见：

```sql
+-----+------+-----------+------+---------+------+-----------------+----------------------------------+
| Id  | User | Host      | db   | Command | Time | State           | Info                             |
+-----+------+-----------+------+---------+------+-----------------+----------------------------------+
| 101 | root | localhost | test | Query   | 29   | Updating        | UPDATE orders SET status=2 ...   |
| 102 | root | localhost | test | Query   | 15   | Waiting for lock| SELECT * FROM orders WHERE ...    |
+-----+------+-----------+------+---------+------+-----------------+----------------------------------+
```

## **数据丢失**

服务器突然断电重启后，订单表出现诡异现象：

```bash
$ ls -lh /var/lib/mysql/order_db/
-rw-rw---- 1 mysql mysql 2.0G Dec 12 03:14 orders.MYD  # 数据文件
-rw-rw---- 1 mysql mysql  128K Dec 12 03:14 orders.MYI  # 索引文件
-rw-rw---- 1 mysql mysql  8.5K Dec 12 03:14 orders.frm  # 表结构
```

**恢复过程**：

```bash
# 林渊的紧急操作
$ myisamchk --safe-recover /var/lib/mysql/order_db/orders.MYI
- recovering (with sort) MyISAM-table '/var/lib/mysql/order_db/orders.MYI'
Data records: 834592   # 部分数据永久丢失！
```

#### **MyISAM 的七宗罪（技术深挖）**

|      痛点      |                      原理性分析                      |      InnoDB 对比方案      |
| :------------: | :--------------------------------------------------: | :-----------------------: |
|     表级锁     |    通过`thr_lock.c`中的`lock_table()`函数全局锁定    | 行级锁（`lock_rec_lock`） |
|    崩溃易损    |          依赖操作系统刷盘，无 Redo 日志保护          | WAL 机制（先日志后数据）  |
| 索引与数据分离 | `.MYI`索引文件与`.MYD`数据文件独立存储，增加 IO 次数 |  聚簇索引（数据即索引）   |
|   无事务支持   |              缺乏 Undo 日志和 MVCC 机制              | ACID 事务（Undo 日志链）  |
|   修复成本高   |        `myisamchk`需停机维护，且可能丢失数据         | 自动崩溃恢复（Redo 回放） |

---

#### **技术彩蛋：MyISAM 的隐藏技能**

**内存映射加速（原理揭秘）**：

```c
// MyISAM通过mmap优化IO
void mi_extra(MI_INFO *info, enum ha_extra_function operation) {
    if (operation == HA_EXTRA_MMAP) {
        info->s->file_map = mmap(0, (size_t)size, PROT_READ,
                               MAP_SHARED, info->s->kfile, 0);
    }
}
```

**适用场景**：

- 只读查询（如数据仓库）
- GIS 空间数据（R 树索引优势）
- 全文检索（2003 年的最佳选择）

---

**下节预告**：

"当我试图用 ALTER TABLE 转换引擎时，整个数据库突然卡死..." —— 林渊如何用插件化存储引擎设计化解危机？

且看 1.2 节《解剖 MySQL 五脏六腑》！

### 本章技术要点总结：

1. **MyISAM 锁机制**：表级锁导致并发灾难，`SHOW STATUS LIKE 'Table_locks%'`监控锁争用
2. **崩溃恢复缺陷**：断电可能导致索引/数据不一致，需定期执行`REPAIR TABLE`
3. **性能优化方向**：通过`myisamchk --analyze`优化索引统计信息
4. **历史价值**：MyISAM 在 GIS 和全文索引领域仍具参考价值

（注：所有技术细节均基于 MySQL 4.0.26 源码及 2003 年硬件环境验证）