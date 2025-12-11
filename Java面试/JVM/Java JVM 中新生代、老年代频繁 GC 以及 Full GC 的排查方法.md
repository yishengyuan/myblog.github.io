## **一、新生代（Young Generation）频繁 GC**

**表现**：

- GC 日志显示 `Minor GC` 很频繁
    
- CPU 被 GC 消耗高
    
- 应用响应延迟增加
    

**可能原因**：

1. **对象创建过快**：短生命周期对象过多
    
2. **新生代容量过小**：Eden 区太小
    
3. **Promotion 阻塞**：对象频繁晋升到老年代
    

**排查方法**：

1. **查看 GC 日志**（开启 `-Xlog:gc*` 或 `-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps`）
    
    - 看 Minor GC 的频率、耗时、晋升数量
        
2. **分析堆分布**
    
    - 使用 `jstat -gc` 或 `jcmd <pid> GC.heap_info`
        
    - 如果 Eden 很快满，Minor GC 就频繁
        
3. **工具分析**
    
    - `VisualVM / JMC / YourKit` 看对象分配情况
        
    - 找出频繁分配的大对象或临时对象
        
4. **优化策略**
    
    - 增大新生代 `-Xmn`
        
    - 使用对象池减少重复创建
        
    - 调整 SurvivorRatio、TenuringThreshold
---

## **老年代（Old Generation）频繁 GC**

**表现**：

- 日志显示 `Major GC` 或老年代 GC 很频繁
    
- Minor GC 可能因为晋升失败触发 Full GC
    

**可能原因**：

1. **老年代内存不足**
    
    - Survivor 区晋升过多对象导致老年代满
        
2. **长生命周期对象过多**
    
    - 缓存、静态集合、List/Map 无限制增长
        
3. **Promotion 失败**
    
    - Minor GC 发现对象晋升到老年代失败触发 Full GC
        

**排查方法**：

1. **分析老年代使用情况**
    
    - `jstat -gc <pid>` 查看 OldGen 使用率
        
    - `jmap -heap <pid>` 获取堆快照
        
2. **排查内存泄漏**
    
    - `MAT / Eclipse Memory Analyzer` 分析 dump 文件
        
    - 查找持续增长对象或无法回收对象
        
3. **优化策略**
    
    - 调整老年代大小 `-Xmx`
        
    - 减少缓存滞留，使用弱引用 / 定期清理
        
    - 优化数据结构，避免全局静态集合膨胀
        

---

## **Full GC**

**表现**：

- 日志显示 `Full GC`
    
- 所有线程停顿，响应明显变慢
    

**可能原因**：

1. 老年代空间不足，Minor GC 触发失败
    
2. 元空间或永久代满
    
3. System.gc() 被显式调用
    
4. 类加载 / 静态缓存导致的内存压力
    

**排查方法**：

1. **分析 GC 日志**
    
    - 查看 Full GC 发生时间、频率、耗时
        
    - 检查是老年代不足还是 Metaspace 不够
        
2. **排查 System.gc() 调用**
    
    - `-XX:+PrintGCApplicationStoppedTime` 或 JVM 参数 `-XX:+DisableExplicitGC`
        
3. **堆快照分析**
    
    - 使用 MAT 查找占用大内存的对象
        
4. **优化策略**
    
    - 增大老年代、元空间
        
    - 避免频繁调用 System.gc()
        
    - 优化缓存和长生命周期对象
        

---

## **排查流程总结**

1. **开启 GC 日志**
    
    - `-Xlog:gc*:file=gc.log:time,uptime,level,tags`
        
2. **初步定位**
    
    - Minor GC 频繁 → 新生代问题
        
    - Major GC/Full GC 频繁 → 老年代或元空间问题
        
3. **堆快照分析**
    
    - 找出泄漏或占用对象
        
4. **代码优化 & JVM 参数调整**
    
    - Eden/OldGen 调整、对象复用、缓存清理
        
5. **监控与验证**
    
    - Prometheus、Grafana、JMC、YourKit 等


![[Pasted image 20251207155609.png]]


Arthas
![[Pasted image 20251207155728.png]]

![[Pasted image 20251207155752.png]]

执行dashboard 仪表盘
![[Pasted image 20251207155839.png]]


![[Pasted image 20251207155909.png]]

![[Pasted image 20251207160523.png]]