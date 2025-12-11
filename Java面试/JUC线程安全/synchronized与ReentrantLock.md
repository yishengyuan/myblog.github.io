synchronized(OtherObject.class){  -> 类的class对象 
   synchronized(myObject){
}
}
 
加锁，一般来说都是必须得一个对象进行加锁 
如果一个线程第一次到synchronized那里，获取到了myObject对象的monitor的锁，计数器就加1，
然后第二次synchronized那里，会再次获取myObject对象的monitor的锁，这个就是重入锁了， 
然后计数器加1变成2。
在底层编译的jvm指令中，会有monitorenter和monitorexit两个命令。  
cpu之类的硬件级别的原理、原子性、可见性、有序性、指令重排、JDK对他的一些优化，偏向锁。 
加锁会执行monitortenter 释放锁会执行monitorexit 


 
1：原始构成
* synchronized 是关键字属于jvm层面的。  monitorenter 底层是通过monitor对象完成的，
* 其实wait/notify 等方法也依赖于monitor对象只有在同步块或者方法中才能调用wait/notify等方法。
* Lock是具体类（java.util.concurrent.locks.Lock） 是api层面的锁
* 2：使用方法
* syncronized 不需要用户手动释放锁，而syncronized 代码执行完系统会自动让宪曾释放对锁的占用 ，
* ReentrantLock则需要用户去手动释放锁，若没有自动释放锁，则可能导致出现死锁现象。
* 需要lock和unlock方法配合try{}finally 语句块来完成
* 3:等待是否可以中断
* synchronized不可终端，除非抛出异常或正常运行完成。
* ReentrantLock可中断 ，1：设置超时方法tryLock{ long timeout,TimeUnit unit}
*                      2:lockInterruptibly()放代码块中，调用interrupt 就可以中断
* 4：加锁是否公平
* synchronized 非公平锁
* ReentrantLock两者都可以 ，默认非公平锁，构造方法可以传入boolean值 ，true为公平锁，false为非公平锁
* 5：锁可以绑定多个条件
* synchronize没有，
* ReentrantLock用来实现分组唤醒需要唤醒的线程们，可以实现精准唤醒，而不是像synchronized要么随机唤醒一个
* 要么唤醒全部线程。
![[Pasted image 20251128180014.png]]

![[Pasted image 20251128180020.png]]
 
 


