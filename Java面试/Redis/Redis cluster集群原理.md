# 1️⃣ 为什么需要 Redis Cluster？

Redis 单机最大问题：

- **容量有限**（单节点内存上限）
    
- **QPS瓶颈**
    
- **单点故障**
    

Redis Cluster 通过 **分布式存储 + 多 master 多 replica + 自动故障转移** 来解决上述问题。

# 2️⃣ Redis Cluster 核心设计：16384 个槽位（hash slot）

**Redis 没有用一致性哈希，而是把整个 key 空间分成固定的 `16384` 个槽**。

流程：

`slot = CRC16(key) % 16384`

然后每个 Master 负责一部分 slot，比如：

|节点|槽位范围|
|---|---|
|Master A|0–5500|
|Master B|5501–11000|
|Master C|11001–16383|

🔍 **槽位是 Redis Cluster 的最核心设计**  
它用于：

- key 分布
    
- 数据迁移
    
- 扩缩容
    
- 故障转移
    

你可以把槽理解为数据库的 “表分区”，节点负责一段区间。

# 客户端直连 Cluster：Smart Client 模式

Redis cluster **不需要 proxy（不像 Codis）**，客户端自己负责路由。

执行命令流程：

1. 客户端先向任何节点询问槽位分布（cluster slots）
    
2. 发现 key 属于哪个 slot
    
3. 直接访问负责该槽的主节点
    

如果访问了错误的节点，会得到：

`MOVED slot node_ip:port`

客户端自动重试。

---

# 4️⃣ 节点之间的通信：Gossip 协议

节点间使用 **Gossip 协议** 定期交换集群信息：

- who 是 master / slave
    
- 槽位分布
    
- 节点是否宕机
    
- 配置版本号
    

特点：

- 类似以太坊 P2P：消息逐渐扩散
    
- 无中心节点
    
- 传播慢但可靠
    

通信端口 = redis_port + 10000  
比如 redis 6379 → cluster bus 16379。
# 5️⃣ 高可用：自动故障转移（failover）

每个 master 有 1～N 个 slave。

### 故障检测流程：

1. 一个节点对另一个节点发起 `PFAIL`（疑似下线）
    
2. 多个节点达成共识 → `FAIL`（确认下线）
    
3. 对应 slave 开始选举
    

### Slave 选举流程（类似RAFT 的投票）

- 优先级：复制偏移量越大（数据越新）优先当 master。
    
- 选中后将自己升级成 master
    
- 重新接管这个节点负责的槽位
    

---

# 6️⃣ 扩容 / 缩容：slot 的迁移

⚡ Redis Cluster 扩容不是自动的，需要人工迁移槽位：

例如增加一个新节点 D：

1. 把 A/B/C 的部分槽迁移到 D
    
2. 迁移过程支持在线进行
    
3. Redis 使用 `ASK` 重定向来保证一致性
    

迁移时客户端可能收到：

`ASK slot node_ip:port`

意思：  
“该槽正在迁移，你去新的节点，但只这一次”。

迁移完成后，该槽永久属于新的节点。

---

# 7️⃣ Redis Cluster 不能执行的命令？

由于数据分布在不同主节点，有些命令无法跨 slot：

❌ 多 key 操作必须在同一个 slot，否则会报错：

`CROSSSLOT Keys in request don't hash to the same slot`

解决方案：

### ⭐ Hash Tag 技术

Key 中加 `{}`：

`user:{1001}:name user:{1001}:balance`

大括号内的内容参与 hash，因此保证落在同一个 slot，可支持事务、Lua、流水线。

---

# 8️⃣ Redis Cluster 节点数量要求

至少：

- **3 个 Master**
    
- **3 个 Slave**（每个 Master 一个）
    

原因：

- 只有一个 slave 声明 fail 才能故障转移（避免误判）
    
- 多数投票机制需要至少 3 个 master
    

---

# 9️⃣ 整体架构总结（图）
```
                ┌──────────────┐
                │   Client     │
                └──────┬───────┘
                       │
      MOVED/ASK + slots metadata
                       │
       ┌───────────────┴───────────────┐
       │               │                │
┌──────────┐    ┌──────────┐    ┌──────────┐
│ Master A │    │ Master B │    │ Master C │
└────┬─────┘    └────┬─────┘    └────┬─────┘
     │               │               │
┌────┴────┐    ┌─────┴────┐    ┌─────┴────┐
│ Slave A │    │ Slave B  │    │ Slave C  │
└─────────┘    └──────────┘    └──────────┘

Gossip 通信 + 槽位分布 + 故障转移

```

---

# 🔥 面试高频问答

### Q1：Redis Cluster 为什么是 16384 槽？

因为：

- 保证哈希均匀
    
- 不需要太多槽（影响节点内部 bitmap）
    
- 节点间元数据同步效率更高
    

比一致性哈希更简单可靠。

---

### Q2：Redis Cluster 如何保证数据一致性？

Redis 集群是 **AP** 模型：

- 可用性优先
    
- 最终一致性，不保证强一致
    
- 写入是先写 master，再异步复制给 slave
    

---

### Q3：节点挂了，整个集群会崩吗？

只要：

- master 掉了
    
- 有 slave
    
- 主节点能达成多数投票
    

就能自动 failover，不影响可用性。

---

# 🏁 总结（面试记住这 4 句话）

1. **Redis Cluster 使用 16384 个槽位分片 key。**
    
2. **客户端 smart 路由 + MOVED/ASK 重定向。**
    
3. **节点使用 Gossip 协议同步 cluster 状态。**
    
4. **master-slave 自动故障转移，多数投票机制。**