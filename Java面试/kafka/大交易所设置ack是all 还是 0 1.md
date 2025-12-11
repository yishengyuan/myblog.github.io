在大交易所或者金融级系统里，**Kafka Producer 的 `acks` 设置几乎都是 `all`**（或 `-1`，含义一样），原因如下：

---

# ✅ Kafka `acks` 选项对比

|acks 值|含义|优缺点|交易所适用性|
|---|---|---|---|
|0|Producer 不等待 Broker 响应|最低延迟，但消息可能丢失|不适合交易系统|
|1|Leader 写入就返回|延迟低，副本可能未写入就 ack|风险较大，可能在 leader crash 时丢消息|
|all / -1|Leader + ISR（同步副本集）全部写入才返回|延迟稍高，但可靠性最高|**大交易所/金融级系统首选**|
# 🔑 原因分析

1. **金融交易系统要求极高可靠性**
    
    - 交易、撮合、风控消息丢失可能直接造成资金损失或风险控制失败。
        
2. **延迟可通过批处理、异步优化**
    
    - Producer 可以 batch、压缩、异步发送，减少 acks=all 带来的延迟。
        
3. **副本机制保证高可用**
    
    - Kafka ISR（in-sync replicas）机制保证只要 leader 和大部分副本写入才 ack，避免数据丢失。
# 🔧 实际配置参考（交易所/量化平台常见）

```
acks=all
retries=10
max.in.flight.requests.per.connection=1
enable.idempotence=true

```

说明：

- `acks=all` → 消息写入 ISR 才返回成功
    
- `retries` → 遇到临时网络或 leader crash 可重试
    
- `max.in.flight.requests.per.connection=1` → 保证顺序
    
- `enable.idempotence=true` → Producer 端幂等，保证 exactly-once 语义
    

---

💡 **总结面试答法**：

> 大交易所生产环境一般使用 **acks=all**，配合幂等 Producer 和副本机制，保证消息可靠性。  
> acks=0/1 虽然延迟低，但在金融级系统中几乎不用，因为可能造成消息丢失。