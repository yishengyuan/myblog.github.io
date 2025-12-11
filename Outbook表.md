# 🏦 OutBox 表是什么？

OutBox 表就是 **用于存放业务事件消息的缓冲表**  
它与订单表、余额表在**同一个数据库、同一个事务中写入**，保证业务数据和事件消息要么**一起成功**，要么**一起失败**。

---

## 📍 一张典型的 OutBox 表结构

```
CREATE TABLE outbox_event (
    id               BIGINT PRIMARY KEY AUTO_INCREMENT,
    event_type       VARCHAR(64) NOT NULL,   -- 订单创建/取消/成交/划转等
    aggregate_id     BIGINT NOT NULL,        -- 关联的业务主键，如 order_id
    payload          JSON NOT NULL,          -- 事件内容（订单详情、变动金额等）
    status           TINYINT DEFAULT 0,      -- 0=未投递 1=投递中 2=成功 3=失败
    retry_count      INT DEFAULT 0,          -- 重试次数
    created_at       DATETIME NOT NULL,
    updated_at       DATETIME NOT NULL
);

-- 业务写库（订单 or 余额） + 写一行Outbox事件 —— 同事务提交

```

📌 事件被投递到 Kafka 前，都会先躺在这个表里。

---

# 🔥 原理（面试能直接讲）

> **单库本地事务解决跨系统一致性问题**  
> 把消息当成业务的一部分写入 OutBox 表，然后异步投递 Kafka。

---

### 🔁 处理流程

```
【事务内】
订单落 MySQL
余额变动写 MySQL
写 Outbox_event 表一行
---- 事务一起提交 (成功/失败一体)

【事务外】
异步Outbox Worker拉取未送出的消息
   ↓ 发送Kafka（支持重试+补偿）
Kafka成功 → 标记status=成功 or 删除
下游（撮合/清算）幂等消费 → 最终一致

```

---

### ⭐ 为什么可靠？

|机制|保证了什么|
|---|---|
|业务写库 + OutBox 同事务|消息不会少（不丢）|
|异步 Worker + 重试 + 重放|失败可重新投递|
|Kafka事务/幂等消费设计|消息不会乱、可重复但不产生重复效果|
|MySQL持久化存储|即使宕机恢复后仍可继续投递|

> **不丢、不漏、可重试、可回溯、最终一致。**

---

# 🧠 你可以在面试这样说（可直接背）

> Outbox 表就是存放业务事件消息的本地表，用来保证业务落库和消息投递的原子性。  
> 订单写入 MySQL 时，会同步写一行 outbox_event，属于同一个本地事务，因此业务成功消息一定存在。  
> 后台 Worker 异步扫描 Outbox 表投递 Kafka，失败可重试、可补偿、可重放，下游做幂等即保证最终一致性。  
> 这个方案避免分布式事务，性能远高于 XA，是目前交易所（Binance / OKX / Bybit）最通用架构。