# ✅ **EVM 链监听（Deposit Listener）总体目标**

CEX 监听以太坊等 EVM 链，主要目的是：

1. **监听用户充值**
    
2. **监听提现是否上链成功**
    
3. **监听风险相关（大额、可疑地址、黑名单）**
    
4. **维护账务一致性（链上与数据库）**
    

可以理解为：**链→监听→入账→对账→风控** 的流水线。

# 🔥 **一、监听交易所用户充值（核心）**

## **1. 扫描区块（Scan Block）**

监听器会执行：

### ✔ 获取最新区块高度

- 读取链上最新高度
    
- 和本地已处理高度对比
    
- 如果落后则启动补区块任务
    

### ✔ 拉取区块数据：transactions + receipts

包括：

- from
    
- to
    
- input data
    
- logs
    
- value
    
- token transfers
    

### ✔ 防止回滚（Reorg）

一般需要 **6~12 confirmations** 才入账。

> 主链回滚时要自动撤销未确认的入账。

# 🔥 **二、识别交易所的充值**

交易所内部维护一份：

`用户充值地址表（address → userId）`

监听器做：

## ✔ **识别是否充值到本交易所地址（to == our address）**

包括：

- 原生币（ETH）
    
- ERC20 Token
    
- ERC721 / ERC1155 NFT（大多是不用的）
    

对于 tokens 我们要解析：

- Transfer(address from, address to, uint value)
    
- topics[1] → from
    
- topics[2] → to
    
- data → value
    

## ✔ **识别合约调用是否是充值（特别是代理合约）**

USDT、USDC 使用 **proxy 合约**  
监听要处理：

- Proxy 合约 address
    
- Implementation 变化
    

每次升级需要同步 ABI。

---

# 🔥 **三、解析充值信息并入账**

监听到交易地址是交易所地址后：

## ✔ 抽取充值细节

- txHash
    
- blockNumber
    
- fromAddress
    
- toAddress（交易所地址）
    
- tokenAddress
    
- amount
    
- decimals
    
- gas info（一般不用于入账）
    

## ✔ 入账逻辑

典型 CEX 流程：

`1. 状态：pending（等待确认） 2. 等待足够 confirm 3. confirm 达到 → credited（入账成功） 4. 通知用户余额变更`

如果确认数未达到，用户余额不增加，只显示 “正在确认中”。

---

# 🔥 **四、监听提现**

提现分两部分：

### **1）发送链上交易**

由交易所冷/热钱包签名发送。

### **2）监听回执判断成功 or 失败**

- status = 1 → success
    
- status = 0 → reverted
    

如果失败：

`更改提现订单状态为失败 给用户退回余额 记录失败原因 进入异常队列（运营处理）`

如果成功：

`确认提现成功 更新账务流水`

---

# 🔥 **五、风险控制（风控监听）**

### ✔ 监听大额或可疑充值

链监听服务会接入：

- 链上 AML 风控系统（如 Chainalysis）
    
- 黑名单地址（OFAC）
    
- mixer 资金（Tornado Cash / Railgun）
    

发现可疑交易：

`冻结用户充值 进入二次审核`

---

# 🔥 **六、对账（Ledger Reconciliation）**

每天或每小时执行：

### ✔ 链上余额 vs 数据库余额

每个币种执行：

`链上余额（address balance） - 提现金额 + 充值金额 = 系统账务余额`

如不一致：

`报警 暂停充值/提现 自动进入排查`

---

# 🔥 **七、处理链上回滚（Reorg）**

如果某个区块被链回滚：

`1. 找出被影响的充值 2. 将充值状态从 credited → rollback 3. 如果用户已可用：扣回 4. 写审计日志`

需要复杂的补偿机制。

---

# 🔥 **八、扫描补偿（Rescan）**

为了防止漏单：

- 每小时补扫过去 N 个区块（比如 2000）
    
- 避免因为故障漏掉充值
    
- 如果发现漏帐 → 自动补记账
    

---

# 🔥 **九、内部事件推送**

监听系统通常会在识别到充值后向内部服务推消息：

- Kafka（topic: deposit-event）
    
- 内部 wallet-service 处理入账
    
- 再推到 user-balance-service
    
- 再推 websocket 给前端
    

---

# 🔥 **十、监控与报警**

必须监控：

- 区块高度落后（严重）
    
- RPC 不可用
    
- 交易解析失败
    
- ABI 解析报错
    
- 单地址资金异常大量变动（私钥泄漏风险）
    

报警等级分级（S1/S2/S3）。

---

# ⭐ 总结（面试可复述版）

> 监听 EVM 链主要做十件事：  
> **扫描区块 → 识别交易所地址 → 解析充值 → 等待确认 → 入账 → 监听提现回执 → 风控检测 → 处理回滚 → 对账 → 补扫 + 监控报警。**
> 
> 核心挑战是：**确认数、合约解析、proxy 解析、回滚处理、漏单补偿、风控审查。**