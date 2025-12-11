# ✅ 1. 先判断“堆积类型”

Kafka 消息堆积一般分三类：

### **① 消费速度 < 生产速度（最常见）**

—— Consumer 跟不上生产。

### **② 消费异常并重试/死循环**

—— 消费报错，导致 Consumer 一直重复消费某条消息。

### **③ 分区数太少，消费者加再多也没用**

Kafka 的扩展单位是**分区**。

# ✅ 2. 快速解决堆积的通用办法（面试和生产都适用）

## **方案 1：增加消费者并行能力（最有效）**

- 增加 Consumer 实例数
    
- 增加 Consumer 线程数（批量拉取批量处理）
    
- 确保消费者数量 ≤ 分区数，否则不会生效
    

**大厂统一做法：**

- 先把 topic 分区数扩到几百
    
- Consumer Group 扩容到 N 个实例，每实例 N 线程
    

### ⚠ 说重点：

> **想加快消费速度，只加消费者没用，你必须保证 topic 分区足够多。**

## **方案 2：使用批量消费 + 批量处理**

把单条消息处理 → 批次处理

### 拉取批量

`fetch.min.bytes=524288     # 512 KB fetch.max.wait.ms=50 max.poll.records=1000`

### 批量写入数据库（一次插入 1000 条）

- 减少 IO 次数
    
- 吞吐会提升十倍以上
    

---

## **方案 3：降低反压，提高 Consumer TPS**

### 调高 poll 间隔

`max.poll.interval.ms=300000`

### 调大 poll 数量

`max.poll.records=2000`

### 加快反序列化

- 自定义高性能序列化器（如 JSON 改为 ProtoBuf / Avro）
    

---

## **方案 4：Consumer 内部用多线程处理消息**

一种常见方式：

Consumer 主线程只 poll  
消息丢到 **Disruptor / 线程池**  
内部多线程并行处理

> 单 Consumer → 8 核 CPU → TPS 翻 5~10 倍

典型结构：

`KafkaConsumer.poll() ──> BlockingQueue / Disruptor ──> 线程池处理`

注意要自己控制 offset 提交（处理成功后再 commit）。

## **方案 5：开启异步 I/O 或 IO 批处理**

如果你消费后写：

- MySQL → 改为批插 batchInsert
    
- Redis → pipeline 批处理
    
- Elasticsearch → bulk API（1000 条一批）
    
- HTTP 调用 → 改为连接池 + 异步请求
    

---

## **方案 6：扩大分区数（最强提升方案）**

堆积严重时，大厂一般直接把 topic 分区扩容：

比如从 12 → 60 → 200

### 分区越多：

- 吞吐越大
    
- 可并行消费者越多
    

### 注意：

partition 扩容不会影响已写数据，但会改变路由分摊。

---

## **方案 7：临时加速通道（旁路快速消费）**

如果堆积百万级、千万级，常用应急方案：

### 做法：

写一个独立的“小工具消费端”，专门批量扫数据，带限流策略，比如：

- 一次性批量拉取 10 万条
    
- 多线程处理
    
- 手动 commit
    

业务逻辑也可以精简，只做必要字段处理。

生产上很常见。

---

## **方案 8：降低 broker 端 throttling（拉取速度受限）**

调大 Broker 参数：

`replica.fetch.max.bytes=10485760 message.max.bytes=5242880`

Consumer 端：

`fetch.max.bytes=52428800 max.partition.fetch.bytes=10485760`

可大幅提升拉取吞吐。

---

# 🔎 3. 关键排查点（排查堆积原因）

## 【原因 1】Consumer 处理逻辑太慢（最常见）

- 数据库写太慢
    
- HTTP 调用慢
    
- 序列化慢
    
- 单线程处理
    

👉 **解决方式：多线程 + 批处理 + 降低 IO**

---

## 【原因 2】Consumer Group 数量 < 分区数

例如：

- 2 个 Consumer
    
- 8 个分区
    

只能同时处理 2 个分区  
6 个分区在排队

👉 增加 consumer group 实例数

---

## 【原因 3】消费异常导致 repeated retry

消息报错 → 消费失败 → 不 commit → 重试 → 卡死

👉 建议加上死信队列（DLQ）或异常告警

---

## 【原因 4】消费端被重平衡（rebalance）卡住

比如：

- poll 太慢
    
- max.poll.interval.ms 太低
    

Consumer 又不断 rebalance → 停止消费 → 堆积

---

# 📌 面试官常问：你们线上堆积怎么处理？

你可以这样答（大厂标准回答）：

> 我们处理 Kafka 堆积，通常从三个维度：扩、并、快。
> 
> 1. **扩** —— 扩 partition 数量（比如从 12 扩到 60）
>     
> 2. **并** —— 扩 consumer group，并在 consumer 内部使用多线程 + 批处理
>     
> 3. **快** —— 减少 IO，使用批量写数据库 / redis pipeline / es bulk
>     
> 
> 同时优化 Kafka client 参数：
> 
> - max.poll.records=2000
>     
> - fetch.max.bytes 加大
>     
> 
> 在应急情况下，我们会临时拉一个“加速消费工具”，单独消费积压消息。
> 
> 最终能把 TPS 提升到原来的 5~20 倍，快速消除积压。