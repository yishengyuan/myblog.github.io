# ✅ **一、JDK7 vs JDK8 实现差异（关键）**

### **JDK7：Segment 分段锁**

- 整个 Map 被拆成 16 个 segments（默认）
    
- 每个 segment 是一个小型的 HashMap + ReentrantLock
    
- 并发度 = segment 数量
    
- **缺点：Segment 架构太重，扩容复杂**
### **DK8：取消 Segment**

使用更细粒度的锁 + CAS，性能大幅提升：

| 操作    | JDK8 并发手段                      |
| ----- | ------------------------------ |
| 插入/替换 | **CAS + synchronized（锁桶头节点）**  |
| 查询    | **完全无锁（volatile 读 + 数组不可变）**   |
| 扩容    | **多线程协助扩容 + forwardNode 迁移标记** |
# ✅ **二、核心：JDK8 的线程安全依靠 3 大支柱**

---

# **1️⃣ CAS（保证无锁竞争）**

### put() 的第一步就是 CAS：

当 bucket（某个索引）是 **null** 时，ConcurrentHashMap 会尝试：

`if (tabAt(tab, index) == null) {     if (casTabAt(tab, index, null, new Node(...)))          return;   // 插入成功，无锁 }`

这里的 tabAt 和 casTabAt 直接操作数组内存偏移（Unsafe），不经过锁。

**目的：**

- 空桶插入不需要加锁
    
- 减少锁的使用，性能极高

# **2️⃣ synchronized（只锁链表/红黑树头节点）**

CAS 失败后（桶不为空），会走 synchronized：

`synchronized (firstNode) {     // 在桶内处理链表或红黑树节点 }`

### 这是关键点：

⚠️ **锁住的是桶（bin）的头节点，不是整个 Map，也不是整个数组。**

因此锁非常细粒度，多个线程只要落在不同的桶，就不会互相阻塞。

相比 JDK7 的 Segment 锁粒度更小、性能更高。

# **3️⃣ 多线程协助扩容（transfer 机制）**

当达到扩容阈值，线程不仅会触发扩容，还会：

- **多个线程一起搬迁数据**
    
- 每个线程负责部分 bucket 的迁移
    
- bucket 在迁移后写入一个 **ForwardNode**（扩容标记）
    

插入或查询时遇到 ForwardNode，会：

→ 主动帮助扩容  
→ 或跳转到新 table 继续处理

这就是所谓的 **协助扩容机制（Helping Transfer）**。

# ✅ **三、ConcurrentHashMap 读取是完全无锁的**

### get() 完全无锁：

- 数组是 volatile 修饰
    
- Node 的 key/value/final 域对读线程可见
    
- 单次读取永远不会阻塞
    

查询流程：

`Node<K,V> e = tabAt(tab, index); if (e == null) return null;  // 链表遍历（只读，不加锁）`

读取永远不加锁，因为：

- Node 的 next 是 final（不可变）
    
- 数组引用是 volatile（可见性）
    
- 扩容用 ForwardNode 保证读操作能顺利跳转
    

---

# ✅ **四、线程安全核心总结（面试可直接背）**

> **JDK8 ConcurrentHashMap 线程安全靠 CAS + synchronized（锁桶头）+ 多线程协助扩容。**
> 
> - 空桶插入用 CAS，无锁
>     
> - 非空桶操作只锁链表/树头节点（极小粒度）
>     
> - 查询无锁（volatile + final）
>     
> - 扩容由多个线程协作完成，避免单线程卡顿
>     

这是目前最优的高并发 Map 设计方案。

# 🔥 面试官常问的扩展问题（我给你简版答案）

---

### **1. 为什么不用 ReentrantLock，而用 synchronized？**

JDK8 之后锁优化非常强，synchronized：

- 轻量级锁、偏向锁等优化比 ReentrantLock 更快
    
- 实现代码更简单
    
- 更适合局部锁场景
    

---

### **2. put() 时为什么需要 CAS + synchronized 混合使用？**

因为：

- **CAS 快**，能避免大量锁竞争
    
- 但在链表或红黑树操作中必须有互斥
    

使用两者结合，效率最高。

---

### **3. JDK8 为什么取消 Segment？**

Segment 结构扩容时太复杂，而且锁粒度较粗；JDK8 的桶锁和协助扩容更加高效。