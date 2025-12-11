The network Network   super node master node seed node各个节点通过tcp沟通
application layer :  BitComin Block chain 
network layer :overlay network 

比特币对块的是1M区块大小

**H(block header) ≤ target**  
意思是：

> **把区块头做一次 SHA-256 哈希，结果必须小于某个难度目标值（target）。**

如果满足这个条件，矿工才算“挖到了一个有效区块”。

# 为什么会有这个条件？

这就是 **比特币的挖矿规则**。

SHA-256 是不可预测的，所以矿工只能**疯狂试 nonce**（随机数）让哈希结果小到某个范围之内。

# **区块头（block header）包含什么？**

区块头是 80 字节，包含：

| 字段                  | 含义            |
| ------------------- | ------------- |
| version             | 协议版本          |
| previous block hash | 上一个区块的哈希      |
| merkle root         | 当前区块所有交易的哈希树根 |
| timestamp           | 时间戳           |
| nBits               | 难度目标的压缩格式     |
| nonce               | 挖矿时不断变化的随机数   |

矿工主要就是不断改变 **nonce** 和 **extra nonce**，看能不能算出一个足够小的哈希。

# **H(block header) ≤ target 的数学含义**

举个例子：

`哈希结果：0x000000A45B92F... target:   0x000000F000000...`

因为哈希结果 < target，

→ **满足挖矿难度**  
→ 这个区块有效  
→ 可以广播给全网

如果：

`哈希结果 = 0x00000FFF... target    = 0x0000000FF...`

哈希结果 > target → 不符合难度 → 无效区块
# 为什么是 “≤ target” 而不是“= target”？

因为 SHA-256 输出随机，想算出刚好等于一个值几乎不可能。

所以用 ≤，让矿工拿到满足“足够小”的哈希即可。

通俗理解：

- target 是一个**门槛**
    
- 哈希是一个**随机点数**
    
- 小于门槛 → 通过
    
- 大于门槛 → 不通过
    

---

# 🎚 target 与难度（Difficulty）的关系

target 越小，满足 H ≤ target 的概率越低，难度越高。

比特币规定：

`difficulty = max_target / current_target`

max_target 是最容易的难度（也叫 difficulty 1）。

当网络总算力上升，target 会自动变小 → 挖矿更难。

---

# 🎰 为什么矿工需要不断试 nonce？

SHA-256 的结果：

- 不可预测
    
- 均匀分布
    
- 每次计算相当于“摇号”
    

成功概率（举例）：

如果 target 要求前 **32 位为 0**，则成功概率是：

`1 / 2^32 ≈ 1 / 40亿`

矿工必须疯狂试：

`nonce 0 nonce 1 nonce 2 nonce 3 ...`

直到某一次：

`sha256(sha256(block_header)) ≤ target`

那一刻，就是“挖到块”。

---

# 💡 用一句最直白的话总结

> **比特币挖矿就是一直算哈希，看谁先算出一个特别小的哈希。  
> target 就是‘小到什么程度算合格’的门槛。  
> H(block header) ≤ target 满足就能出块。**

比特币的出块速度为10分钟
以太坊为15秒 

每两个星期调整一次 

2016 区块* 10/（60 * 24）= 14天 
target阈值迭代更新时间 
挖矿难度太高 出块块慢 需要14天调整
target = target * (autualTime /expected time)

## 如果不调整时间？
代码是开源的 如果某一个节点运行的不调
nbits 在header里面有四个字节 
区块链合法性通不过

很多参数设计是比较保守的，比特币之前也有很多数字货币 但是之前都失败了。 

![[Pasted image 20251119105041.png]]![[Pasted image 20251119105107.png]]

挖矿难度
