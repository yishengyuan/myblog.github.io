# 🔥 Redis 并发竞争问题有哪些？

常见的 7 大类：

1. **缓存击穿（Cache Miss + 高并发）**
    
2. **缓存穿透（访问不存在的数据）**
    
3. **缓存雪崩（大量 key 同时失效）**
    
4. **Cache Aside 读写并发导致缓存脏读**
    
5. **多线程竞争写缓存 → 数据覆盖**
    
6. **Redis 分布式锁竞争问题**
    
7. **Redis 自增、扣减并发竞争（库存问题）**
    

---

下面我按 **问题 → 场景 → 竞态风险 → 解决方案** 完整整理。

---

# 🟥 1. 缓存击穿（热点 key 失效瞬间被大量请求打爆数据库）

## ❗ 场景

一个超热点 key（如商品详情、排行榜）突然过期 → 上万请求同时读 → 都打到 DB

## ❗ 风险

- 数据库扛不住
    
- 产生短暂雪崩
    

## ✔ 解决方案

### **方案 A：互斥锁（Mutex）防击穿**

伪代码：

`if (cache miss) {     // 只允许一个线程去查询数据库     if (SETNX lock) {          value = db.query()          redis.set(key, value, ttl)          release lock          return value     } else {          sleep 50ms          retry     } }`

Redis 实现：

`SET lock_key 1 NX EX 5`

### **方案 B：逻辑过期（不设置真正 TTL）**

用字段记录 expireTime

读缓存后发现逻辑过期 → 异步后台线程更新缓存 → 前台仍然提供旧数据（保证可用性）

---

# 🟩 2. 缓存穿透（频繁访问不存在的数据）

## ❗ 场景

攻击者或业务 bug 请求极大范围内不存在的 ID，例如：

`GET /user?id=999999999`

## ❗ 风险

- 缓存永远 miss
    
- 所有请求都打 DB
    

## ✔ 解决方案

### **方案 A：缓存 Null 值**

不存在的数据：

`redis.set(key, null, ttl=60s)`

### **方案 B：布隆过滤器（Bloom Filter）**

大规模鉴别 key 是否存在，大厂常用。

---

# 🟦 3. 缓存雪崩（大量 key 同时过期）

## ❗ 场景

同一时间大量 key 设置了相同 TTL，突然同时过期。

## ❗ 风险

- 数据库瞬间被压垮
    
- 缓存层宕机
    

## ✔ 解决方案

### **方案 A：TTL 随机化**

给 TTL 加随机时间，例如：

`3600 + rand(0~600)`

### **方案 B：热点数据永不过期 + 逻辑过期**

避免真实过期造成雪崩。

### **方案 C：多级缓存（L1 + L2）**

---

# 🟧 4. Cache Aside 读写并发导致脏数据

你前面提到的问题：

`读线程写回旧值 → 覆盖最新值`

## ✔ 解决方案

- 延迟双删（删缓存 → 更新 DB → sleep → 再删除）
    
- 用队列对更新操作排队
    
- TTL 自动修复
    
- Redis 分布式锁保证读写互斥
    

---

# 🟪 5. 高并发下，多个线程同时写缓存 → 数据覆盖

## ❗ 场景

两个线程执行：

`A: redis.set(key, v1) B: redis.set(key, v2)`

不知道最终是谁覆盖谁（特别是在写热点 key 时）。

## ✔ 解决方案

### **方案 A：加分布式锁（Redisson RLock）**

### **方案 B：使用版本号（CAS乐观锁）**

`watch key if version match:    set new value else:    retry`

### **方案 C：写操作入队，单线程消费（消息队列）**

---

# 🟫 6. Redis 分布式锁竞争问题

常见问题：

1. 锁续期失败导致被提前释放
    
2. 锁被其它线程误释放（非原子）
    
3. Redis 主从切换导致锁丢失（Redis 单点不安全）
    

## ✔ 解决方案

### **方案 A：使用 Redisson（推荐）**

自动续期 + Lua 原子释放锁 + 多节点 RedLock 方案。

### **方案 B：Lua 脚本保证原子性**

`if redis.call("get", key) == value then     return redis.call("del", key) end return 0`

### **方案 C：锁加随机值 + 自动续期线程**

---

# 🟨 7. Redis 原子操作竞争（比如扣库存）

## ❗ 典型错误做法：

`if (stock > 0) {     stock = stock - 1     redis.set(stock) }`

并发下肯定超卖！

## ✔ 正确做法

### **方案 A：Redis 原子命令**

`DECR stock_key`

### **方案 B：Lua 脚本保证“检查 + 扣减”原子性**

`if stock > 0 then     stock = stock - 1     return stock else     return -1 end`

### **方案 C：使用 Redis Stream 做限流/排队**

### **方案 D：使用 Redisson Semaphore/RateLimiter**

**Redis 的并发竞争问题主要体现在 7 个方面：**

1. 缓存击穿 → 加互斥锁 / 逻辑过期
    
2. 缓存穿透 → 缓存空值 / 布隆过滤器
    
3. 缓存雪崩 → TTL 随机化 / 多级缓存
    
4. 读写并发不一致 → 延迟双删 / 锁 / MQ
    
5. 并发写缓存覆盖 → 分布式锁 / CAS / 消息队列
    
6. 分布式锁竞争问题 → Redisson + Lua 原子释放
    
7. 扣库存并发竞争 → Lua 脚本 / 原子 DECR / Stream