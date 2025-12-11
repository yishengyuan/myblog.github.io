# ✅ **1. 静态集合导致的内存泄漏（最经典）**

```
public class MemoryLeak {
    private static List<Object> list = new ArrayList<>();

    public static void main(String[] args) {
        while (true) {
            list.add(new byte[1024 * 1024]); // 不断往静态列表加数据
        }
    }
}

```

### ❗ 解释

静态集合生命周期 = 整个 JVM。  
只要 list 不被清空，它引用的对象永远不会被 GC → 泄漏。

# ✅ **2. 未关闭资源（IO、JDBC、Socket）**

```
public void read() throws Exception {
    FileInputStream fis = new FileInputStream("a.txt");
    byte[] data = fis.readAllBytes();
    // 忘记 fis.close();
}

```

### ❗ 解释

没有 close() → FileDescriptor 无法释放 → 堆外内存泄漏。

# ✅ **3. ThreadLocal 内存泄漏（面试高频）**

```
private static ThreadLocal<byte[]> tl = new ThreadLocal<>();

public static void main(String[] args) {
    tl.set(new byte[1024 * 1024]); // 大对象
}

```

### ❗ 为什么泄漏？

ThreadLocalMap 的 key 是弱引用，但 value 是强引用。  
如果 ThreadLocal 被回收但线程还活着：

`Entry(key=null, value=大对象)`

value 永远不会被 GC。

# ✅ **4. HashMap key 没有正确实现 hashCode/equals**

```
Map<Person, String> map = new HashMap<>();
Person p = new Person("Tom");
map.put(p, "xxx");

// 修改 p 的字段导致 hashCode 改变
p.name = "Jerry";

// 现在拿不到这个 key，也删除不了 → 泄漏

```

### ❗ 解释

hashCode 改变后 key 永远找不到，GC 也不能清理，形成隐藏引用。

✅ **5. 自定义监听器/回调未注销**

```
public class EventSource {
    private static List<EventListener> listeners = new ArrayList<>();

    public void register(EventListener l) {
        listeners.add(l);
    }
}

```

### ❗ 解释

Listener 持有对业务对象的引用，如果不 remove，就会导致对象无法被回收。

✅ **6. 内部类隐式持有外部类引用（常见于多线程）**
```
public class Outer {
    private byte[] big = new byte[10 * 1024 * 1024];

    class Inner implements Runnable {
        public void run() {
            // ...
        }
    }
}

```
### ❗ 解释

Inner 实例隐式持有外部类 Outer 的引用。  
如果线程长期存活，大对象 big 也无法回收。

解决：使用 static inner class。
✅ **7. 长生命周期对象持有短生命周期对象**


```
Map<String, Object> cache = new HashMap<>();

public void putTempData(String key) {
    cache.put(key, new Object());  // 短期数据放到长期缓存
}

```
### ❗ 解释

长生命周期 = 静态对象 / 单例  
短生命周期对象加入它 → 永远不能释放。
✅ **8. TimerTask 异常导致的泄漏**
```
Timer timer = new Timer();

timer.schedule(new TimerTask() {
    @Override
    public void run() {
        throw new RuntimeException("error"); // Timer线程死了
    }
}, 0, 1000);

```

### ❗ 解释

Timer 线程挂掉后，已加入的 TimerTask 永远不会执行或释放。

---

# ✅ **9. 使用堆外内存但没有释放（Netty / DirectBuffer）**
```
ByteBuffer buffer = ByteBuffer.allocateDirect(1024 * 1024);
// 忘记调用 Cleaner.clean()

```

### ❗ 解释

DirectByteBuffer 在堆外，GC 不管理 → 会导致 off-heap 内存泄漏。

# 🔥 最适合面试总结一句话（强烈建议背）

> Java 的内存泄漏本质是 **对象依然被引用但不再使用**。  
> 常见原因包括：
> 
> - 静态集合持有对象
>     
> - 未关闭资源（IO/JDBC/Socket）
>     
> - ThreadLocal 使用不当
>     
> - 长生命周期对象引用短生命周期对象
>     
> - 回调 / 监听器未注销
>     
> - 内部类隐式引用外部类
>     
> - DirectBuffer 堆外内存未释放
>