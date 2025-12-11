4# 1️⃣ OOM 的本质

**本质是内存耗尽或系统无法分配所需内存。**

引发 OOM 的原因主要有三类：

1. **程序产生了无限增长的对象（逻辑错误）**
    
2. **内存分配不足（堆太小、Metaspace太小等）**
    
3. **Native 层发生内存泄漏（直接内存、线程栈）**
# 2️⃣ 常见的 OOM 类型（必须记住）

### （1）`java.lang.OutOfMemoryError: Java heap space`

含义：堆空间不足，无法创建新的对象  
常见场景：

- 大量 List/Map 无界增长
    
- 数据一次性加载过大
    
- 死循环创建对象
    
- 对象太大（大数组、大字节数组）
### （2）`java.lang.OutOfMemoryError: GC overhead limit exceeded`

含义：**GC 99% 的时间都在回收，但回收不到 1% 的内存**  
说明：已经进入 **内存溃败（memory collapse）** 状态。

根因：

- 堆太小
    
- 内存泄漏
    
- 大对象频繁创建

### （3）`java.lang.OutOfMemoryError: Metaspace`

含义：Metaspace用满  
触发因素：

- 大量动态生成的类（CGLib、JDK Proxy）
    
- 热部署导致类加载器泄漏
    
- Web 容器（Tomcat）频繁发布应用
### （4）`java.lang.OutOfMemoryError: Direct buffer memory`

含义：**直接内存（NIO / Netty）耗尽**  
常见于：

- Netty大型 buffer 没有释放
    
- ByteBuffer.allocateDirect 大量创建
    
- 文件/网络 IO 引发的 Native 内存泄漏
### （5）`java.lang.OutOfMemoryError: unable to create new native thread`

含义：系统无法创建更多线程  
原因：

- 线程数过多（线程池配置错误，例如 CachedThreadPool）
    
- OS 对单进程线程数的限制（ulimit）
    
- 每个线程的栈空间过大（-Xss 配置过大）
### （6）`java.lang.OutOfMemoryError: Out of swap space`

含义：系统 swap 耗尽 / Native内存耗尽  
一般是 **JVM 外的内存泄漏**（JNI、Unsafe）

# 3️⃣ 如何快速排查 OOM（面试官最看重）

标准排查三板斧：

## （1）查看 GC 日志

判断是否是：

- 淘汰不掉（内存泄漏）
    
- 堆太小
    
- Full GC 是否频繁
    
- Promotion 是否异常
    

## （2）抓取堆内存快照（heap dump）

使用工具：

- MAT（Eclipse Memory Analyzer）
    
- VisualVM
    
- JProfiler
    

重点分析：

- Dominator Tree（谁占用了最大内存）
    
- 静态集合（HashMap、ConcurrentHashMap）
    
- ThreadLocal 泄漏
    
- Listener 泄漏
    

## （3）排查线程（如果是 unable to create new native thread）

- `jstack` 查看线程数
    
- 看是否线程池创建过多线程
    
- 检查死循环导致线程不退出
# 4️⃣ 常见 OOM 真实案例（面试官爱问）

### ✔ 1. HashMap 作为缓存，没有过期策略 → 内存永远增长

解决：使用 Caffeine/Guava 缓存

### ✔ 2. ThreadLocal 使用不当

解决：使用后 remove()

### ✔ 3. Netty 直接内存泄漏

解决：使用 leakDetector、确认 ByteBuf release()

### ✔ 4. CGLib 大量代理类生成

解决：开启代理类回收、减少动态代理数量

---

# 5️⃣ 如何预防 OOM（亮点总结）

1. **合理配置堆大小：**  
    `-Xms -Xmx` 固定，避免堆动态缩放
    
2. **大对象优化：**  
    拆分、复用、避免一次性加载大数据集
    
3. **使用合适的 GC：**  
    G1、ZGC 减少 Full GC 引发的内存溃败
    
4. **严格控制缓存大小**  
    使用 LRU、LFU、TTL
    
5. **监控 + 告警**
    
    - 老年代使用率 >70%
        
    - Full GC 次数
        
    - 线程数增长
        
6. **写代码要有生命周期意识**
    
    - 不随便用 static
        
    - 及时释放引用
        
    - 避免全局对象无限增长