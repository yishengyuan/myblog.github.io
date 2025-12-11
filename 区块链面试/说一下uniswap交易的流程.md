下面我给你用 **面试官最喜欢的方式** + **真实链上执行细节** 来讲清楚 **Uniswap（以 V2 为例）完整交易流程**。  
（最后我也会简单补充 V3 的差异。）

# ✅ **Uniswap 交易流程（V2）——从用户点击 Swap 到链上执行完毕**

整个过程分成 **6 大步骤**：

`用户输入参数 → 前端计算路径与价格 → 用户授权（approve） → 交易合约进行 swap → AMM 价格更新 → LP 收到手续费`

# 🔥 **1. 用户输入 swap 参数**

用户操作：

- 输入 `swapTokenA → TokenB`
    
- 输入数量，比如 `1 ETH`
    
- 选择交易滑点 `slippage`
    
- 前端需要告诉用户预期可得到的 Token B 数
    

此时只是 UI 计算，还未链上操作。

# 🔥 **2. 前端根据 reserve 计算报价（恒定乘积公式）**

Uniswap V2 使用 AMM：

`x * y = k`

比如：  
池子里有：

`100 ETH （x） 300,000 USDT（y）`

用户想卖 1 ETH 换 USDT。

### 公式计算：

1. 先扣手续费（0.3%）
    

`amountInWithFee = amountIn * 997 / 1000`

2. 计算输出：
    

`amountOut = (reserveOut * amountInWithFee) / (reserveIn + amountInWithFee)`

UI 用此公式计算出用户预估可获得多少 Token B。

---

# 🔥 **3. 用户先授权 approve（只有第一次需要）**

因为 Token（如 USDT / ERC20）存在钱包内，  
要让 router 合约有权限转用户资产。

执行：

`USDT.approve( UniswapRouter, amount )`

链上交易，用户付 gas。

---

# 🔥 **4. 用户提交 swap 交易（真正的 Swap）**

前端调用 router：

`swapExactTokensForTokens(     amountIn,     amountOutMin,     path,     to,     deadline )`

### Router 做的事：

#### ✔ 1）从用户钱包扣 tokenIn（如 ETH → WETH → router → pair）

Router 将 tokenIn 转入对应的交易对（Pair）合约。

#### ✔ 2）Router 调用 Pair.swap()

router 把数据传给 Pair 合约：

`pair.swap(amount0Out, amount1Out, to, data)`

参数含义：

- 哪个 token 输出多少
    
- 输出给谁（用户地址）
    

---

# 🔥 **5. Pair 合约完成 swap（核心步骤）**

### 流程如下：

#### ✔ Step 1：更新储备量 reserveX reserveY

先记录旧储备量：

`reserve0 reserve1`

#### ✔ Step 2：把 tokenIn 变成 pair 资产

router 已经把 tokenIn 转入 pair 合约中。

pair 读取当前余额 = reserveIn + amountIn。

#### ✔ Step 3：计算可输出的 tokenOut（已在 router 中计算）

pair 接收：

`amountOut`

并发送给用户。

#### ✔ Step 4：检查恒定乘积不被破坏（最重要的安全机制）

`(newBalance0 * newBalance1) >= oldK * 1000^2`

否则 revert，防止闪电贷欺诈。

#### ✔ Step 5：更新全局 reserve（作为最新平衡点）

`reserve0 = newBalance0 reserve1 = newBalance1`

交易完成。

---

# 🔥 **6. LP 获得手续费（自动加入池子）**

0.3% 手续费不直接转给 LP，而是：

`手续费直接加入池子，使 k 增大`

LP 份额不变，但池子总量变大 → LP 自动变富。

---

# 🧊 **重要：为什么 Uniswap V2 会滑点？**

因为池子使用：

`x * y = k`

交易数量越大，x/y 变化越大，价格影响就越大 → 滑点出现。

如果你交易量占池子 10%，你基本把价格砸穿。

---

# 🌊 **Uniswap V3 交易流程的不同（简述）**

V3 核心是：

✓ 有限价格区间（集中流动性）  
✓ 每次 swap 要计算多个 price ticks  
✓ 流动性不是全局，而是在一个区间内  
✓ 手续费可以是 0.05% / 0.3% / 1% 等不同档位

V3 的 swap 本质是：

`在多个价格区间内迭代 根据当前区间的 liquidity & tick 计算输出 如果区间流动性耗尽，进入下一个区间`

但总体流程仍然：

`Router → Pool → tokenIn → tokenOut → 更新 tick → 更新 liquidity`

---

# ⭐ 面试可背诵总结

> **Uniswap V2 的 Swap 流程是：**  
> 用户输入金额 → 前端用 x*y=k 计算输出 → approve 授权 → Router 调用 → Router 把代币转到 Pair → Pair 根据恒定乘积公式计算输出 → 更新储备 → LP 自动赚手续费。

> **Uniswap V3 只是在 swap 时要穿越多个流动性价格区间，但基本流程一致。**