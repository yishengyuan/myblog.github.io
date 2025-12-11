

![[Pasted image 20251128175150.png]]

![[Pasted image 20251128175200.png]]
![[Pasted image 20251128175220.png]]

AtomicInteger原理是cas，cas的核心类是Unsafe类，使用volatile修饰

![[Pasted image 20251128175121.png]]

![[Pasted image 20251128175111.png]]

![[Pasted image 20251128175101.png]]
![[Pasted image 20251128175051.png]]


循环时间长，消耗很大
![[Pasted image 20251128175232.png]]

![[Pasted image 20251128175256.png]]

ABA 不介意中介过程：可以忽略ABA    比如借钱   


