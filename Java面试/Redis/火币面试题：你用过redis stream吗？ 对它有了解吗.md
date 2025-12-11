# ✅ **Redis Stream 用过吗？对它有什么了解？（高分回答）**

我实际项目中用过 Redis Stream 做 **订单异步消息、用户通知推送、日志流水处理**。  
Redis Stream 是 Redis5.0 引入的 **持久化消息队列**，主要特点是：

---

# 1️⃣ **核心概念（面试官最看重理解）**

### ✔ Stream（消息流）

- 类似 Kafka 的 topic
    
- 内部按 **消息 ID（时间戳-自增序列）** 保序存储
    
- append-only，写入很快。
    

### ✔ Consumer Group（消费者组）

- 多消费者可以组成一个 group。
    
- **每条消息只会被组内的一个消费者处理一次**（与 Kafka 一样）。
    

### ✔ Pending Entries List（PEL）

- 记录哪些消息已经被客户端读取但还没有 ack。
    
- 用于保证 **至少一次消费** 和消息追踪。
    

这部分是面试官最想听的三点。

---

# 2️⃣ **生产写入方式（简洁）**

`XADD mystream * user_id 123 order_id 456`

- `*` 表示 Redis 自动生成消息 ID。
    
- 支持 `MAXLEN` 限制长度，避免流无限增长。
    

`XADD mystream MAXLEN ~ 10000 * ...`

---

# 3️⃣ **消费方式（两种模式）**

## （1）普通消费（类似订阅）

`XREAD COUNT 10 BLOCK 5000 STREAMS mystream $`

## （2）消费者组消费（重点）

`XREADGROUP GROUP mygroup c1 COUNT 10 BLOCK 5000 STREAMS mystream >`

- `>` 表示只读 **未被分配过的新消息**。
    

### ack 消息

`XACK mystream mygroup 1607513491-0`

---

# 4️⃣ **死信处理 & 消费失败处理机制**

Redis Stream 自带未 ack 消息查询：

`XPENDING mystream mygroup`

如果某消费者挂掉，可使用：

`XCLAIM`

把未处理消息转移给另一个消费者处理。

➡ **这相当于 Kafka 的 rebalancing + 死信重分配逻辑。**  
交易所非常看重这类“消息可靠性”的细节。

---

# 5️⃣ **Redis Stream vs Kafka 区别（常被问）**

|项目|Redis Stream|Kafka|
|---|---|---|
|TPS|中等（单机10万+）|非常高（百万级）|
|持久化|RDB/AOF|磁盘日志|
|顺序性|Stream 内部按 ID 保序|Partition 内保序|
|重复消费|可能（至少一次）|可配置|
|典型场景|小型 MQ / 低延迟系统|大规模消息系统|

**总结一句话：** Stream 更适合作为“小型高可用 MQ”，延迟小；Kafka 更适合大规模数据流。

---

# 6️⃣ **项目实际使用经验（加分项）**

如果你用在交易场景，可以这样说：

> 在交易项目中，Redis Stream 用于处理订单事件流，结合 Consumer Group 保证每条订单事件只由一个消费者处理。
> 
> 我们会定期检查 PEL 防止消费者异常导致消息堆积，同时设置 MAXLEN 控制流长度。
> 
> 整个策略类似轻量级 Kafka，对延迟非常敏感的业务表现很好。

（这段话很加分，能体现你真的做过）

---

# 7️⃣ **可以强调的优点（面试官喜欢听）**

- **消息有序**
    
- **可持久化**
    
- **可自动分配消费者（group）**
    
- **阻塞式拉取，低延迟**
    
- **支持消息重试、消息恢复**
    

---

# 8️⃣ **一句总结**

Redis Stream 是 Redis 内置的队列系统，支持消费者组、消息追踪、未 ack 重试、持久化，适合高并发、低延迟的事件流处理场景。我在实际项目中用过，特别适用于订单、通知、日志等轻量 MQ 场景。