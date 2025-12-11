# ✅ **用 Disruptor 做了什么**

### 1️⃣ **为什么用 Disruptor**

- **高性能事件队列**：百万级 QPS、微秒级延迟
    
- **适合交易撮合/风控/清算**：对延迟敏感、高并发场景
    
- **特点**：
    
    - 内存环形队列（RingBuffer） + 无锁设计
        
    - 支持单生产者/多生产者，多消费者异步处理
        
    - **零 GC / 对象复用** → 降低延迟抖动
### 2️⃣ **我在项目中的典型应用**

#### **（1）撮合引擎事件传递**

- 将撮合事件（下单、撤单、成交）写入 Disruptor RingBuffer
    
- 异步推送到：
    
    - **清算服务**（保证金更新、爆仓）
        
    - **消息推送服务**（WebSocket / APP）
        
    - **数据库落库服务**（成交记录、流水）
        

#### **（2）清算服务**

- 内存清算模块消费 Disruptor 消息
    
- 保证 **顺序处理**，处理爆仓和保证金逻辑
    
- 异步将结果写回数据库和推送模块
    

#### **（3）高频风控**

- 对短时间高频下单、异常交易行为进行事件驱动检测
    
- Disruptor 队列保证 **低延迟事件处理**
    
- 触发风控报警或冻结账户操作
### 3️⃣ **实现方式**

1. **定义事件 Event**
```
    public class OrderEvent {
    private String orderId;
    private BigDecimal amount;
    private OrderType type;
    // getter/setter
}

```
    
2. **创建 RingBuffer + EventHandler**
    
    `RingBuffer<OrderEvent> ringBuffer = RingBuffer.createSingleProducer(OrderEvent::new, 1024*64);`
    
3. **生产者生产事件**
    
```
  long seq = ringBuffer.next();
try {
    OrderEvent event = ringBuffer.get(seq);
    event.setOrderId(orderId);
    event.setAmount(amount);
} finally {
    ringBuffer.publish(seq);
}

```
    
4. **消费者异步处理事件**
    
    - 可并行消费不同逻辑，如清算、落库、推送
        

---

### 4️⃣ **为什么选择 Disruptor 而不是普通队列**

|特性|普通队列|Disruptor|
|---|---|---|
|延迟|毫秒级|微秒级|
|GC|对象频繁创建|对象复用，零 GC|
|多线程|需锁|无锁 / CAS|
|场景|一般业务|高频撮合、金融风控、清算|

---

### 5️⃣ **高分总结回答模板**

> 我在项目中用 Disruptor 构建 **高性能异步事件队列**，将撮合事件、清算事件、风控事件异步解耦。  
> 它保证了 **低延迟（微秒级）、高吞吐（百万级 QPS）、顺序一致性**，同时支持异步落库、爆仓处理和推送。  
> 与普通队列相比，Disruptor 通过 **RingBuffer + 无锁 + 对象复用** 大幅降低了 GC 抖动，适合永续合约和现货高频交易场景。


单线程情况下，不加锁的性能 > CAS操作的性能 > 加锁的性能。

在多线程情况下，为了保证线程安全，必须使用CAS或锁，这种情况下，CAS的性能超过锁的性能，前者大约是后者的8倍。

综上可知，加锁的性能是最差的。

## Disruptor的设计方案

Disruptor通过以下设计来解决队列速度慢的问题：

- 环形数组结构

为了避免垃圾回收，采用数组而非链表。同时，数组对处理器的缓存机制更加友好。

- 元素位置定位

数组长度2^n，通过位运算，加快定位的速度。下标采取递增的形式。不用担心index溢出的问题。index是long类型，即使100万QPS的处理速度，也需要30万年才能用完。

- 无锁设计

每个生产者或者消费者线程，会先申请可以操作的元素在数组中的位置，申请到之后，直接在该位置写入或者读取数据。

下面忽略数组的环形结构，介绍一下如何实现无锁设计。整个过程通过原子变量CAS，保证操作的线程安全。