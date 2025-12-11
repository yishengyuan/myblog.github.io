下面我把 **Redisson 分布式锁（RLock）常用参数**和**实际 CEX 项目中的推荐设置**全部一次讲清楚，带上最佳实践和坑点。

---

# ✅ **Redisson 分布式锁核心参数（最重要的 3 个）**

## **1) leaseTime（锁自动过期时间）**

`lock.lock(10, TimeUnit.SECONDS);`

- **含义**：锁在 10 秒后自动释放（无论线程是否正常执行完）。
    
- **适用于**：执行时间可预估、业务不复杂的场景。
    
- **风险**：如果业务执行时间超过 10 秒 → **锁提前释放** → 导致多个线程同时执行（数据脏写风险）。
    

---

## **2) waitTime（等待获取锁的最大时间）**

`boolean locked = lock.tryLock(5, 10, TimeUnit.SECONDS);`

含义：

- 等待锁最多 5 秒
    
- 获得锁后自动在 10 秒后释放（leaseTime）
    

**适用于：**

- 对延迟敏感，不能无限等待锁的业务。
    

---

## **3) watchdog 自动续期（默认是 30 秒）**

当你使用：

`lock.lock();`

没有指定 leaseTime 时：

- Redisson 会启用 **看门狗 Watchdog**
    
- 每 10 秒自动给锁续期 30 秒
    
- 只要业务没执行结束，锁就不会掉
    

**适用场景：**

- 执行时间不可预估
    
- CEX 下单、撮合、资金变动这种必须确保强一致性的逻辑
    

---

# 🔥 **总结：三种常用加锁方式**

|加锁方式|有无续期|是否安全|场景|
|---|---|---|---|
|`lock.lock()`|✅自动续期|安全|业务耗时不确定|
|`lock.lock(10, SECONDS)`|❌无续期|风险大|业务必须死在 10s 内|
|`tryLock(wait, lease, unit)`|❌部分安全|良好|拿不到锁就放弃的操作|

---

# 🧩 **完整示例代码**

### **1. 推荐方式：lock()（带 watchdog 自动续期）**

`RLock lock = redissonClient.getLock("order_lock:" + orderId); try {     lock.lock();  // 自动续期     // 业务逻辑... } finally {     if (lock.isHeldByCurrentThread()) lock.unlock(); }`

⭐ **CEX 项目里默认推荐这个**

---

### **2. 限时等待 + 死线锁 leaseTime**

`boolean success = lock.tryLock(3, 10, TimeUnit.SECONDS); if (!success) {     throw new RuntimeException("系统繁忙，请稍后再试"); }`

---

# 📌 **最常见的参数踩坑与建议**

## ❌ **坑 1：自定义 leaseTime 太短**

`lock.lock(5, SECONDS)`

如果业务 >5 秒 → 锁提前释放，导致两个线程冲突写入。

**建议：**

- CEX 类系统 **绝不要**随意 lock(5)
    
- 复杂逻辑统一使用 `lock()` 自动续期
    

---

## ❌ **坑 2：waitTime 设置太长导致系统阻塞**

如果：

`tryLock(60, 30, SECONDS)`

高并发时线程阻塞 60 秒 → 堆满 → 服务雪崩。

**推荐：**

`waitTime = 1~5 秒 leaseTime = 根据业务设（10~30 秒）`

---

## ❌ **坑 3：finally 中没有判断是否当前线程持有锁**

`lock.unlock(); // 错误：可能抛异常`

正确写法：

`if (lock.isHeldByCurrentThread()) {     lock.unlock(); }`

---

# 🚀 **CEX 交易系统中参数推荐（实战级）**

| 业务场景        | waitTime | leaseTime | 建议方式    |
| ----------- | -------- | --------- | ------- |
| 下单冻结资金      | 1s       | 自动续期      | lock()  |
| 提币上链 Deduct | 3s       | 自动续期      | lock()  |
| 账户资金变动      | 0–2s     | 自动续期      | lock()  |
| 冲正、对账       | 5s       | 60s       | tryLock |
| 活动发奖励       | 不等待      | 30s       | tryLock |
