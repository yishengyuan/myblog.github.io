# ✅ 一、**Solidity 0.8 之前 vs 0.8 之后的核心区别**

Solidity 0.8 是一次**重大更新**，对安全性、编译器、特性都带来影响。面试官最关心的其实是：

✔ **0.8 之后内置溢出检查，不需要 SafeMath**  
✔ **0.8 之后会自动 revert 溢出 / 下溢**  
✔ **SafeMath 在 0.8 之后不再必须**（但部分库仍使用）

下面详细讲：

---

# 1️⃣ **数值安全性（重中之重）**

## **Solidity < 0.8**

- `uint256`、`int256` 的加减乘除 **不会自动检查溢出**
    
- 上溢（Overflow） 和 下溢（Underflow）不会 revert
    
- 结果会发生“回绕（wrap around）”
    

例如：

`uint8 a = 255; uint8 b = a + 1;  // 结果变成 0，不会 revert`

这导致很多资金安全 Bug，甚至被专门利用。

👉 所以前 0.8 必须用 OpenZeppelin 的 SafeMath。

## **Solidity >= 0.8**

✔ 内置**自动溢出检查**  
✔ 溢出/下溢会自动 revert  
✔ 不需要 SafeMath

例如：

`uint8 a = 255; uint8 b = a + 1;  // Solidity 0.8 直接 revert: arithmetic overflow`

这是 0.8 最大的安全提升。

---

# 2️⃣ **Solidity 0.8 新特性（面试常问）**

| 功能                             | <0.8 | >=0.8       |
| ------------------------------ | ---- | ----------- |
| **算术溢出检查**                     | ❌ 无  | ✔ 自动 revert |
| **自定义错误（custom error）**        | ❌ 无  | ✔ 节省 gas    |
| **try/catch 支持自定义错误捕获**        | ❌    | ✔           |
| **ABI coder v2 默认开启**          | ❌    | ✔           |
| **更多内联汇编检查**                   | 较弱   | 更严格         |
| **gas 成本变化，对 revert 的 gas 优化** | -    | ✔           |

---

# 3️⃣ **0.8 之后为什么大部分合约不再需要 SafeMath？**

因为：

- 所有算术操作已经内置 overflow 检查
    
- 编译器提供更快的机器码
    
- SafeMath 反而**多余且增加 gas**
    

所以 OpenZeppelin 在 4.x 起：

`SafeMath for uint256 只在 0.8 以下版本使用`

# 二、SafeMath 是如何保证上溢/下溢的？

下面你一定要会说，这是面试官爱问的细节：

---

# ✅ SafeMath 的核心机制（写出这三条就满分）

### **1）使用 `require` 检查结果是否超界**

例如：

### **加法 add**

```
function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    require(c >= a, "SafeMath: addition overflow"); 
    return c;
}

```

逻辑：

- 如果 b>0 且 c < a，就说明回绕了 → overflow → revert
    

---

### **减法 sub**

```
function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    require(b <= a, "SafeMath: subtraction overflow");
    return a - b;
}

```

逻辑：

- 如果 b > a，则结果会回绕 → 下溢 → revert
    

---

### **乘法 mul**

```
uint256 c = a * b;
require(a == 0 || c / a == b, "SafeMath: multiplication overflow");

```

逻辑：

- 如果除不回去 → overflow
    

---

# ⚡ 高分总结（你可直接背）

> Solidity 0.8 开始内置了算术溢出检查，上溢/下溢会自动 revert，因此不再需要 SafeMath。  
> 0.8 之前算术不会检查，会发生 wrap-around，因此必须使用 SafeMath 通过 require 做边界检查来防止溢出。  
> SafeMath 的核心原理就是在加减乘之前检查结果是否超过类型最大最小值，不满足就 require revert。