线程池为什么不自定义线程 ？ 撮合系统中适合用什么线程池配置？ 
## **一、线程池为什么不自定义线程**

1. **线程创建成本高**
    
    - Java 线程的创建和销毁涉及操作系统资源（内存、CPU、线程上下文切换），开销大
        
    - 如果每次任务都创建新线程，会导致频繁 GC 和系统资源浪费
        
2. **线程数量难以控制**
    
    - 自定义线程容易导致过多线程同时运行，CPU 竞争严重
        
    - 线程过多 → 上下文切换频繁 → 整体吞吐下降
        
3. **没有统一管理**
    
    - 线程池可以复用线程、统一调度、支持拒绝策略和任务队列
        
    - 自定义线程很难做到统一监控、管理和故障恢复
        
4. **缺乏高级功能**
    
    - 线程池提供：
        
        - **任务排队/队列管理**
            
        - **线程复用**
            
        - **拒绝策略**（当任务过多时的处理方式）
            
        - **定时调度和延迟任务**
            
    - 直接自定义线程无法轻松实现这些功能
        

✅ **总结**：线程池是 **线程复用 + 任务调度 + 流量控制** 的最佳实践，随意自定义线程在高并发场景下容易出问题。

## **二、撮合系统线程池配置**

撮合系统是典型的 **低延迟、高吞吐**场景，需要考虑以下几点：

### **1. 核心线程池大小**

- 核心线程数 ≈ **CPU 核心数**
    
- 因为撮合计算密集型（CPU-bound），线程数过多反而会抢 CPU
    
- 对 I/O 较少的撮合线程，通常用 `Runtime.getRuntime().availableProcessors()` 或略多 1~2 个
    

### **2. 队列类型**

- **SynchronousQueue / Disruptor**
    
    - 如果任务必须立即执行，避免队列堆积延迟
        
- **LinkedBlockingQueue**
    
    - 可用于非关键逻辑任务，例如日志、风控通知
        
- **撮合核心模块推荐使用 Disruptor**
    
    - 提供 **无锁、高吞吐、低延迟**
        
    - 事件顺序严格，避免线程切换开销
        

### **3. 拒绝策略**

- 核心撮合线程池任务满时：
    
    - **CallerRunsPolicy**：主线程自己执行，避免任务丢失
        
    - **AbortPolicy**：快速失败，触发告警
        
- 对非核心模块可选择丢弃策略
    

### **4. 线程命名和监控**

- 核心撮合线程池最好用 **ThreadFactory** 命名线程，方便监控和排查
    
- 定时打印线程池状态：`getActiveCount() / getQueue().size()`
    

---

### **示例：撮合线程池配置（Java）**

```
int cpu = Runtime.getRuntime().availableProcessors();

ThreadPoolExecutor matchExecutor = new ThreadPoolExecutor(
        cpu,                 // corePoolSize
        cpu + 1,             // maximumPoolSize
        60L, TimeUnit.SECONDS, 
        new SynchronousQueue<>(), 
        new ThreadFactory() { // 自定义线程名
            private final AtomicInteger count = new AtomicInteger(0);
            public Thread newThread(Runnable r) {
                return new Thread(r, "match-thread-" + count.getAndIncrement());
            }
        },
        new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略
);

```

- 线程数接近 CPU 核心数 → CPU 利用率高
    
- SynchronousQueue → 避免任务堆积
    
- CallerRunsPolicy → 避免任务丢失
