# ✅ **Solidity / EVM 里常见的几种 Call**

EVM 中的外部调用主要有 **5 类 call**：

---

# 1️⃣ **call**

### **作用：普通外部调用（最底层 call）**

`(bool success, bytes memory data) = target.call{value: x}(payload);`

**特点：**

- 能发送 ETH
    
- 能传任意 calldata
    
- **不会自动 revert**（需要手动检查 success）
    
- Gas 不限制（默认把剩余 gas 全给对方）
    
- 可能引发 **重入风险（Reentrancy）**
    

👉 低级、高风险，常见于 proxy、fallback、跨合约接口兼容。

# 2️⃣ **delegatecall**

### **作用：目标合约用“调用者合约的存储与上下文”执行代码**

`(bool success, ) = lib.delegatecall(payload);`

**特点：**

- 使用 **当前合约的 storage**
    
- 使用 **当前合约的 msg.sender、msg.value**
    
- 常用于 **可升级合约（Proxy Pattern）**
    
- 极高风险（对 storage layout 要严格一致）
    

👉 Proxy 可升级核心，火币、币安几乎所有合约都靠它。

# 3️⃣ **staticcall**

### **作用：只读调用（不能修改状态）**

`(bool success, bytes memory result) = target.staticcall(payload);`

**特点：**

- 不能写入状态
    
- 常用于 view/pure 查询
    
- EVM 强制禁止 SSTORE/LOG/MINT 等写入操作
    

👉 常用于价格预言机查询、链上 view 查询、构造签名数据等。

# 4️⃣ **callcode（已废弃）**

老式调用方式：

- 类似 delegatecall
    
- 但使用 **调用者的 storage** 与 **被调用合约的代码**
    
- 因安全性不如 delegatecall，已废弃，不再使用
    

👉 面试回答里提一下就够。

---

# 5️⃣ **transfer / send（其实也是基于 call）**

### **transfer**

`payable(target).transfer(amount);`

- 固定只给 2300 gas（防止重入）
    
- 如果失败自动 revert
    
- 但由于 EIP-1884 增加 gas 消耗，transfer 已不可靠
    

### **send**

`bool ok = payable(target).send(amount);`

- 只给 2300 gas
    
- 返回 bool，不会 revert
    

👉 transfer/send 本质都是 **call 的封装**。

---

# ⭐ 补充：Solidity 高级特性里的 call

### **① try/catch（对 call 包装）**

`try target.foo() returns (uint v) {} catch {}`

### **② fallback + receive（用于处理未知 call）**

`fallback() external {} receive() external payable {}`

---

# 🔥 **终极高分总结（你可以直接背）**

> Solidity 里一共有几种 Call？
> 
> - **call**：最底层外部调用，可带 ETH，可能导致重入，需检查 success。
>     
> - **delegatecall**：目标合约以当前合约的 storage/msg.sender 运行代码，是 Proxy 可升级模式核心。
>     
> - **staticcall**：只读调用，禁止状态修改，用于 view 查询。
>     
> - **transfer/send**：都是基于 call，实现 ETH 转账但限制 2300 gas。
>     
> - **callcode**：老的 delegatecall 版本，已废弃。
>     
> 
> 这几种 Call 构成了 EVM 的全部外部调用模型。