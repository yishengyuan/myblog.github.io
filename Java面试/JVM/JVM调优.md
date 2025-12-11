# 🚀 一、JVM 调优的总原则（非常关键）

> **没有问题不要调优，有问题要先定位根因，再调优。**  
> JVM 调优不是调参数，而是：  
> **定位瓶颈 → 评估对象模型 → 调整内存结构 → 观察 GC 效果 → 压测验证**

调优一定要有输入指标，比如：

- Full GC 频繁
    
- 延迟高
    
- 吞吐量低
    
- OOM
    
- 内部服务间响应超时
    
- CPU 持续居高不下
# 🚦 二、JVM 常见问题分类（对应调优方向）

### **1. GC 频繁（Young GC / Full GC）**

一般原因：

- 新生代太小导致频繁 YGC
    
- 老年代太小导致频繁 FGC
    
- 对象晋升太快
    
- 大量短命对象 or 大对象
    
- 老年代碎片严重（CMS）
    

对应调优：

- 增大年轻代、调 Eden:Survivor 比例
    
- 增大老年代
    
- 使用 G1 替代 CMS
    
- Large Object 直接晋升老年代的问题避免
### **2. 延迟抖动（RT 高峰）**

原因：

- 停顿时间过长（STW）
    
- CMS remark 或 Full GC
    
- 大对象分配失败
    
- G1 Mixed GC 阶段设置不合理
    

调优方向：

- 使用 G1（JDK8+ 推荐）
    
- 限制单次 Mixed GC 时间
    
- 增大 TLAB
    
- 调整触发 Mixed GC 的阈值
### **3. OOM**

常见 OOM 类型：

- Java heap space
    
- GC overhead limit exceeded
    
- Direct buffer memory
    
- Metaspace
    
- unable to create new native thread
    

每种对应不同策略。
### **4. CPU 高、卡顿、STW 时间过长**

- YGC/FGC 耗时长
    
- GC 线程太少
    
- G1 的 humongous object 太多
    
- CMS 线程不足
    
- User code 死循环
# 🔥 三、JVM 调优的黄金 7 步流程（企业生产实际用的）

---

# **① 收集数据（必须）**

用常见工具：

- **jstat -gcutil pid 1s**
    
- **jmap -histo pid**
    
- **jmap -heap pid**
    
- **jstack pid**
    
- **arthas**
    
- **Java Flight Recorder（JFR） → 最强**
    

要获取指标：

- YGC 频率
    
- FGC 次数
    
- 老年代占用
    
- 新生代晋升速率
    
- 碎片率
    
- 吞吐量
    
- STW 时间
    
- 线程数
    

---

# **② 判断问题类型**

例子：

- “YGC 每秒 3 次” → 新生代太小
    
- “FGC 每 10 分钟一次” → 老年代太小或 promotion failure
    
- “FGC 一次 2s” → G1 参数不合适
    
- “出 OOM:MetaSpace” → 类加载过多，反射代理太多
    
- “unable to create new native thread” → Linux ulimit / 容器配置问题
    

---

# **③ 调整堆结构（最重要）**

## 📌 常用配置（推荐生产基础配置）

### **1. 设置堆大小**

`-Xms4g -Xmx4g`

（线上一般 Xms = Xmx，避免动态扩容）

### **2. 设置 GC 线程**

`-XX:ParallelGCThreads=8 -XX:ConcGCThreads=4`

### **3. 设置年轻代大小**

- 保证 YGC < 0.5 秒
    
- 新生代占大概总堆的 1/3 ~ 1/2 较好
    

`-XX:NewRatio=2         # 老年代:新生代 = 2:1`

或手动：

`-Xmn1g`

### **4. Survivor 区调优**

`-XX:SurvivorRatio=8      # Eden:Sur = 8:1:1 -XX:MaxTenuringThreshold=15`

---

# **④ 选择 GC 垃圾回收器（核心）**

### 🧩 **生产一般推荐 G1（JDK8+）**

`-XX:+UseG1GC`

并设置停顿目标：

`-XX:MaxGCPauseMillis=200`

也可以设置：

- 混合回收比例
    
- Region 大小
    

---

### 🚀 特殊场景选择

#### 场景 1：延迟敏感（RT < 20ms）→ **ZGC / Shenandoah（JDK11+）**

`-XX:+UseZGC`

#### 场景 2：CPU 很少、吞吐量优先 → **Parallel GC**

`-XX:+UseParallelGC`

---

# **⑤ 调整 TLAB（减少锁竞争）**

对于大量小对象分配场景（RPC 框架、JSON 序列化），非常有效：

`-XX:+UseTLAB -XX:TLABSize=512k`

---

# **⑥ 针对问题专项优化**

## 💥 Promotion failed（晋升失败）

- 增大 Survivor
    
- 增大老年代
    
- 降低动态年龄判断
    

## 💥 老年代碎片（CMS）

→ 换 G1，彻底解决

## 💥 DirectBuffer OOM（Netty 常见）

`-XX:MaxDirectMemorySize=2g`

## 💥 Metaspace OOM

`-XX:MaxMetaspaceSize=512m`

---

# **⑦ 压测验证（关键但常被忽略）**

- 使用 JMeter、wrk、locust
    
- 压 30 分钟以上
    
- 必须观察 GC 日志（一般用 GCViewer / GCEasy）
    

---

# 🔥 四、最常用且有效的 JVM 调优模板（直接可用）

## **G1 调优模板（99% 都用这个）**

`-server -Xms4g -Xmx4g -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:InitiatingHeapOccupancyPercent=45 -XX:+ParallelRefProcEnabled -XX:ConcGCThreads=4 -XX:ParallelGCThreads=8 -XX:+UseStringDeduplication -XX:+DisableExplicitGC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/data/dump.hprof`

---

# 🧨 五、总结（经验萃取）

> **JVM 调优是分析问题 + 修改策略 + 验证，不是堆参数。**

调优关键点：

1. **先分析，再调整**
    
2. 观察：GC 次数、STW、晋升、老年代占用
    
3. 新生代大小和 Region 设置决定 60% 性能
    
4. GC 类型决定 30% 性能
    
5. 线程数、TLAB、StringDedup 再决定剩下的 10%


![[Pasted image 20251207134539.png]]

![[Pasted image 20251207134914.png]]
![[Pasted image 20251207134925.png]]

