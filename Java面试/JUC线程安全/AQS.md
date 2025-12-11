# ✅ 一、面试官最爱听的 20 秒版（背诵）

**AQS 是 Java 并发包中用于构建锁和同步器的核心框架。  
它通过一个 `state`（共享资源）+ 一个 FIFO 等待队列（CLH队列）来管理线程争用，  
并提供“独占模式”和“共享模式”的模板方法，  
ReentrantLock、CountDownLatch、Semaphore、ReentrantReadWriteLock 都是基于 AQS 实现的。**

# ✅ 二、AQS 深度理解版（强烈推荐）

## ⭐ 1. AQS 的本质

AQS = 一个同步框架，让你可以非常轻松地实现各种 Lock、同步工具。

核心思想：

`如果获取资源失败，则将线程进入队列阻塞； 如果资源可用，则通过 CAS 修改 state 来获取； 释放资源后唤醒队列中的下一个`

➡️ **它是 Java 并发包的地基。**

---

# ⭐ 2. AQS 的核心组成部分

## ✔ (1) 一个 int 类型的 `state`

表示同步状态，例如：

- 可重入锁的加锁次数
    
- Semaphore 的剩余许可数
    
- CountDownLatch 的计数器
    

通过 **CAS 原子更新**：

`compareAndSetState()`


## ✔ (2) 一个 CLH 双向队列（FIFO）

用于管理等待线程。

结构大概是：

`head <-> node1 <-> node2 <-> node3 ... tail`

每个节点包含：

- 线程本身
    
- 是否需要被唤醒
    
- 前驱 / 后继节点
    
- 等待状态
    

➡️ **失败的线程被封装成节点 → 入队 → 阻塞。**

## ✔ (3) LockSupport.park() / unpark()

AQS 不直接调用 wait/notify，而是用 LockSupport（更底层、更可控）。

- 不满足条件：`park()` 阻塞
    
- 被唤醒：`unpark(nextThread)` 唤醒下一个
    

---

# ⭐ 3. AQS 的两种模式

## ✔ （1）独占模式（Exclusive）

一次只能让一个线程获得资源。

典型：

- ReentrantLock（独占锁）
    
- ReentrantReadWriteLock 的写锁
    

原理：

`tryAcquire() 失败：入队 + park() 成功：继续执行`

---

## ✔ （2）共享模式（Shared）

允许多个线程共享资源。

典型：

- Semaphore（限制最大并发量）
    
- CountDownLatch（计数器归零后同时唤醒所有）
    
- 读写锁的读锁
    

共享模式会唤醒多个节点：

`tryAcquireShared() tryReleaseShared()`

---

# ⭐ 4. AQS 为什么强大？

因为它是一个**模板方法模式（Template Method）**框架。

你继承 AQS，只需要实现：

`tryAcquire() tryRelease() tryAcquireShared() tryReleaseShared()`

剩下：

- 排队
    
- 阻塞/唤醒
    
- CAS State
    
- 锁重入处理
    
- 竞争策略
    

全部由 AQS 帮你处理。

➡️ 这是 AQS 的设计精髓："业务逻辑你写，底层线程调度我来干"。

---

# ⭐ 5. AQS 在实际类中的体现（必考）

|同步器|模式|state 含义|
|---|---|---|
|ReentrantLock|独占|持有锁次数|
|Semaphore|共享|剩余许可数量|
|CountDownLatch|共享|计数器|
|ReentrantReadWriteLock|独占 + 共享|高 16 位写锁，低 16 位读锁|
|FutureTask|独占|任务状态|

➡️ **几乎整个 java.util.concurrent 都是靠 AQS 撑起来的。**

---

# ⭐ 6. AQS 的执行流程（面试官喜欢你画图的说法）

以独占模式为例：

`线程 A 尝试 CAS 获取 state 成功 → 直接执行   线程 B 获取失败 → 进入 CLH 队列 → park 阻塞   线程 A 释放锁 → 修改 state → unpark(B)   线程 B 醒来再尝试抢锁`

清晰、严谨。

---

# ⭐ 7. 典型追问（面试官常问，看你是否理解 AQS）

### ❓1. 为什么 AQS 使用 CLH 队列？

答：

- FIFO 公平排队
    
- 不需要大量锁保护
    
- 减少线程切换
    
- 有前驱指针即可判断是否需要挂起
    

---

### ❓2. AQS 是自旋还是阻塞？

答：

- 初次尝试：CAS 自旋
    
- 多次失败：加入队列，park 阻塞
    
- 唤醒后：继续尝试自旋
    

➡️ **自旋 + 阻塞混合策略，性能更佳。**

---

### ❓3. 为什么 AQS 不直接使用 synchronized + wait/notify？

答：

- wait/notify 只能通知全部或一个，不可控制目标线程
    
- notify 可能丢失唤醒
    
- LockSupport.unpark 可以精准唤醒指定线程
    
- park/unpark 更底层、更高性能
    

---

### ❓4. 可重入锁如何支持重入？

答：

CAS 成功 → state++  
释放时 state--  
当 state 回到 0 → 真正释放锁

---

# 🎯 总结一句适合面试的高阶回答

**AQS 通过 CAS 维护 state，通过 CLH 队列管理等待线程，通过 LockSupport 实现阻塞/唤醒。  
	它是 Java 锁和同步器的通用模板，分独占和共享模式，使得像 ReentrantLock、Semaphore、CountDownLatch 等同步组件得以高性能实现。**
![[Pasted image 20251128185005.png]]