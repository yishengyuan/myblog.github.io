# 🟦 **一、Producer → Kafka 的写入完整流程**

以下过程你能背下来，面试官会觉得你非常懂 Kafka：

---

## **1️⃣ Producer 发送消息（send）后发生什么？**

`producer.send(record)`

之后发生 **3 个内部步骤**：

---

### **① 序列化（Serializer）**

Kafka Producer 会把 key 和 value 用配置的序列化器变成 byte[]  
比如：

- StringSerializer
    
- JsonSerializer
    
- Avro / Protobuf serializer
### **② 分区器（Partitioner）决定写入哪个 partition**

规则：

|场景|策略|
|---|---|
|指定 partition|优先级最高|
|含 key|hash(key) % partitions|
|无 key|Sticky Partitioner（同一批 stick 同一分区）|

得到 partition ID 例如 partition = 2。

### **③ 写入 Producer 的 RecordAccumulator 缓冲区**

消息被放入内存 **RecordAccumulator**，Kafka 会**批量**发送以提高吞吐。

**什么时候发送？**  
触发条件之一满足即可：

- batch.size 满了
    
- linger.ms 达到（等待更多消息凑批）
    
- 内存缓冲区 buffer.memory 不够
    
- 程序调用 flush()
# 🟦 **二、Producer → Broker 网络传输**

Kafka Client 内部线程 **Sender Thread** 会定期从 RecordAccumulator 拉 batch 发给 Broker。

过程：

1. **找到 partition 的 Leader Broker**
    
    - Producer 通过 metadata 知道 partition leader 在哪台 broker
        
2. **将消息 batch 发送给 Leader Broker**
    
3. Leader Broker 写入本地页缓存（Page Cache）
    
4. 同时复制（Replication）到 Follower Broker（ISR）
    

---

### **写入机制：顺序写 + PageCache + Zero Copy**

Kafka 的高性能来自三件套：

| 技术             | 作用             |
| -------------- | -------------- |
| 顺序写 LogSegment | 极高磁盘写入吞吐       |
| PageCache      | 写入 OS 缓存，不阻塞磁盘 |
| ZeroCopy 发送    | 传输不经过用户态，提高性能  |
# 🟦 **三、Broker 如何持久化消息**

Leader 收到 batch 后：

1. 将消息追加写入 Partition 的 log 文件（顺序写）
    
2. 追加写 index 文件（offset → position）
    
3. Follower 通过 Fetch 请求同步 Leader
    
4. 当 **acks** 条件满足后返回给 Producer
    

---

## **acks 决定数据可靠性**

|acks|含义|
|---|---|
|0|Producer 不等待结果，最快|
|1|Leader 写入成功就响应（默认）|
|all（-1）|Leader + 所有 ISR 完成才成功（最高可靠性）|
# 🟩 **四、Consumer 是如何监听/拉取 Kafka 数据的？**

Consumer **不是消息推送(push)**  
它是 **主动拉取模式（pull）**。

流程如下——

---

# 🟦 **四步消费流程（Consumer → Broker）**

## **1️⃣ 加入 Consumer Group（负载均衡）**

Consumer 启动后向 Kafka Coordinator 发送：

`JoinGroupRequest`

Coordinator 确定你的分配策略：

- CooperativeSticky（推荐）
    
- Sticky
    
- RoundRobin
    
- Range
    

完成 Rebalance，分配 partitions，例如：

`C1 -> partition 0,1 C2 -> partition 2,3`

---

## **2️⃣ 进入轮询（poll loop）进行拉取**

你的代码：

`while(true) {     ConsumerRecords records = consumer.poll(Duration.ofMillis(100)); }`

Kafka 内部协议是：

`FetchRequest`

Consumer 发 Fetch 请求给 partition leader，告诉 leader：

- 我要从 offset = X 开始拉
    
- 每次给我最大多少字节
    

---

## **3️⃣ Broker 返回数据（FetchResponse）**

Broker 直接将 log segment 中的数据 **零拷贝**返回给消费者。

Consumer 收到数据后执行：

- 反序列化（Deserializer）
    
- 业务处理逻辑（你的代码）
    

---

## **4️⃣ 提交 offset（确认消费）**

两种方式：

### ✔ 自动提交

`enable.auto.commit=true`

Kafka 每隔 5s 提交 offset。

### ✔ 手动提交（推荐）

`consumer.commitSync();`

保存在：

`__consumer_offsets`

这个内部 topic。

---

---

# 🟥 **五、整体数据流转图（高质量总结图）**

从 Producer 写入 Kafka，到 Consumer 拉取的全链路：
```
Producer  
  ↓（序列化/分区器/批处理）
RecordAccumulator  
  ↓（Sender 线程）
Leader Broker  
  ↓（复制到 Follower）
Log 文件（顺序写）
  ↓
Consumer Group  
  ↓（poll 拉取）
处理业务逻辑  
  ↓  
提交 offset → __consumer_offsets

```

---

# 🟦 面试口述版本（可直接背）

> “一条数据从 Producer 发出后，会经过序列化、分区器选择分区，再进入 RecordAccumulator 等待批处理，Sender 线程负责将 batch 发送给 partition leader。Leader 将消息顺序写入 log 文件，并复制到 ISR Follower，当 acks 条件满足后返回 ack。
> 
> Consumer 启动后加入 Consumer Group，经分区分配策略获得分区，然后进入 poll 循环，通过 Fetch 请求从自己的 partition leader 拉取数据，反序列化后执行业务逻辑，最后提交 offset 到 __consumer_offsets，用于下一次恢复消费进度。”