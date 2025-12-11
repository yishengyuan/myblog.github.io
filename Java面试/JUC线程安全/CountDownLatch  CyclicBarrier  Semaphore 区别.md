# ✅ **1. CountDownLatch —— “一次性的门闩”**

**适用场景：等待一组任务执行完毕，再继续往下走。**

特点：

- 一次性用完（count 到 0 后就不能重置）
    
- 常用于：**主线程等待多个子任务完成**
    

### **💡 场景例子：系统启动时需要加载三个配置模块，全部加载完才能启动服务**

`CountDownLatch latch = new CountDownLatch(3);  new Thread(() -> {     loadConfigA();     latch.countDown(); }).start();  new Thread(() -> {     loadConfigB();     latch.countDown(); }).start();  new Thread(() -> {     loadConfigC();     latch.countDown(); }).start();  // 等全部加载完成 latch.await(); System.out.println("系统启动完成");`

**一句话总结**：  
👉 _多个任务干完了，主线程才继续。一次性的。_



# ✅ **2. CyclicBarrier —— “可重复使用的栅栏”**

**适用场景：一组线程互相等待，等都到齐了再一起继续执行。**

特点：

- 可重复使用（循环）
    
- 强调：**线程之间互相等待，像赛跑选手在起跑线等齐**
    

### **💡 场景例子：多玩家组队游戏，必须 5 人到齐才能开始匹配**

`CyclicBarrier barrier = new CyclicBarrier(5, () -> {     System.out.println("5人到齐，开始匹配！"); });  for (int i = 0; i < 5; i++) {     new Thread(() -> {         System.out.println(Thread.currentThread().getName() + " 到达房间");         barrier.await();         System.out.println(Thread.currentThread().getName() + " 开始游戏");     }).start(); }`

**一句话总结：**  
👉 _大家都到齐后一起继续，可重复使用。像赛跑等待同伴。_


# ✅ **3. Semaphore —— “信号量，限流/允许多少线程进入”**

**适用场景：控制并发量，限制同时访问某资源的线程数。**

特点：

- 强调 “最多允许 N 个线程同时执行临界区”
    
- 用于限流、连接池、抢车位
    

### **💡 场景例子：停车场只有 3 个车位，同时只能停 3 辆车**

`Semaphore semaphore = new Semaphore(3);  for (int i = 0; i < 10; i++) {     new Thread(() -> {         try {             semaphore.acquire(); // 抢车位             System.out.println(Thread.currentThread().getName() + " 停车中");             Thread.sleep(2000);         } finally {             semaphore.release(); // 离开         }     }).start(); }`

**一句话总结**：  
👉 _限制并发量，有多少许可证就能有多少线程执行。_


# 📌 三者核心区别一句话总结

|工具|用途|特点|
|---|---|---|
|**CountDownLatch**|让“主线程等其他线程”|一次性|
|**CyclicBarrier**|让“线程相互等待”|可重复使用|
|**Semaphore**|限制允许多少线程同时执行|强调限流|

# 📌 最通俗的类比（你面试可以直接说）

### ✔ **CountDownLatch = 发射火箭倒计时**

等所有准备完成（countdown 0），火箭发射。

### ✔ **CyclicBarrier = 大家一起等满人再出发的旅游车**

人齐了车就开，可以周而复始运行。

### ✔ **Semaphore = 停车场的车位数量**

最多只能有 N 个线程同时运行。