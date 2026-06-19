上一篇[《MySQL：MyISAM锁表致千万损失！穿越工程师如何逆天改命》](https://mp.weixin.qq.com/s/G9t1fv5k_30v6Ns4W03qXg)，我发现自己穿越到了 过去，这个年代的 MySQL 居然还在用 MyISAM……

次日上午，技术部紧急会议

> "林工，你说要换引擎就换？" 

首席DBA老张拍案而起，"这系统跑了三年都没事，你才来三天就搞事情？"

林渊默然调出昨晚的监控数据：

```bash
# 昨夜事故报告
Lock_time_avg: 12.7s  # 表锁平均等待时间
Table_locks_immediate=2345  
Table_locks_waited=8765  # 锁等待率高达78%
```

> "各位请看，"林渊点击投影，"这不是故障，而是架构级癌症。"



## 连接池危机

**诡异现象**：

```sql
SHOW PROCESSLIST;
+-----+------+-----------+------+---------+------+-------+------------------+
| Id  | User | Host      | db   | Command | Time | State | Info             |
+-----+------+-----------+------+---------+------+-------+------------------+
| 101 | app  | 10.0.0.5  | prod | Sleep   | 632  |       | NULL             |
| 102 | app  | 10.0.0.5  | prod | Sleep   | 587  |       | NULL             |
| 103 | app  | 10.0.0.5  | prod | Sleep   | 524  |       | NULL             |
```

（超过300个僵尸连接，消耗1GB内存）

**技术解析**：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202503292238264.png)

- **线程泄漏原理**：
  MySQL 4.0采用"每连接每线程"模型，线程执行完不会销毁而是进入`thread_cache`。
- 但当`wait_timeout`设置过大时（默认8小时），大量空闲线程堆积。

**林渊的解法**：

```c
// 修改mysqld.cc的线程管理逻辑
void handle_one_connection(THD *thd) {
    while (!abort_loop) {
        if (thd->net.vio->read_packet() == 0) {  // 无数据时主动释放
            thread_scheduler.end_thread(thd, true);
            break;
        }
        do_command(thd);
    }
}
```

*操作结果：* 内存占用从3.2GB降至1.8GB，QPS提升40%。

## SQL执行过程

**惊魂时刻**：当林渊试图优化慢查询时，系统突然报错：

```sql
ERROR 1064 (42000): You have an error in your SQL syntax...
```

——用户输入的`SELECT * FORM orders`竟然未被拦截！

**解剖流程**：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202503292240360.png)



**关键发现**：

- **查询缓存陷阱**：`query_cache_type=ON`导致频繁缓存失效（命中率仅12%）
- **解析器漏洞**：未启用严格模式（`sql_mode`未设置）允许错误语法通过
- **优化器缺陷**：缺乏直方图统计，错误选择全表扫描

**林渊的急救包**：

```sql
SET GLOBAL query_cache_size=0;  -- 关闭毒药级查询缓存
SET GLOBAL sql_mode='STRICT_TRANS_TABLES';  -- 启用严格模式
ANALYZE TABLE orders;  -- 手动更新统计信息
```



## 变更存储引擎

**惊险时刻**：当林渊尝试在线更换存储引擎时

```sql
ALTER TABLE orders ENGINE=InnoDB;
```

系统突然僵死！`SHOW PROCESSLIST`显示：

```sql
| 145 | system user | NULL | NULL | alter table | 89  | copy to tmp table | 
```

**引擎切换原理**：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202503292244328.png)

**林渊的破局操作**：

1. 使用`pt-online-schema-change`工具在线变更（提前20年发明）
2. 分阶段迁移数据：

```bash
# 步骤1：创建影子表
CREATE TABLE _orders_new LIKE orders ENGINE=InnoDB;

# 步骤2：分批拷贝（每次10万条）
INSERT INTO _orders_new SELECT * FROM orders WHERE id BETWEEN ? AND ?;

# 步骤3：原子切换（0.01秒锁定）
RENAME TABLE orders TO _orders_old, _orders_new TO orders;
```



## 引擎插件的秘密

林渊在`ha_myisam.cc`中发现关键结构：

```c
struct st_mysql_storage_engine myisam_storage_engine = { 
    "MyISAM",
    "MySQL AB",
    "Default engine with fast read speed",
    { /* 函数指针表 */ 
        myisam_create_handler, 
        myisam_hton_commit,
        NULL  // 事务相关为空
    }
};
```

> "原来MyISAM的事务支持是先天残疾..."他若有所思。

**下节预告**：

> "当我启动InnoDB引擎时，服务器内存突然耗尽..." —— 林渊如何用Buffer Pool优化化解内存危机？

且看下一章节《InnoDB的降维打击》！



## 本章技术总结：

1. **连接池调优公式**： thread_cache_size = max_connections × 0.2 wait_timeout = 300 # 生产环境建议值

2. 查询优化三板斧：

   - 关闭查询缓存（`query_cache_type=OFF`）
   - 强制严格模式（`sql_mode=STRICT_ALL_TABLES`）
   - 定期ANALYZE TABLE更新统计信息

3. 引擎切换黄金法则：

   ```bash
   # 安全变更流程
   mysqldump --single-transaction db > backup.sql  # 事务级备份
   pt-online-schema-change --alter "ENGINE=InnoDB" D=db,t=orders
   ```