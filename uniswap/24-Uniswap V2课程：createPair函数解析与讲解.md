**Uniswap V2课程：createPair函数解析与讲解**



**要点总结**

​	1.	**Uniswap V2合约部署的两种方式**：

​	•	调用 UniswapV2Router02 合约。

​	•	调用 UniswapV2Factory 合约。

​	2.	addLiquidity**流程**：

​	•	在 UniswapV2Router02 中调用 addLiquidity 函数。

​	•	内部调用 addLiquidity 函数。

​	•	最终调用 UniswapV2Factory 合约的 createPair 函数。

​	3.	createPair**函数的作用**：

​	•	部署 Uniswap V2 Pair 合约，用于管理代币对的交换和流动性操作。

​	•	输入：两个代币地址（Token A 和 Token B）。

​	•	输出：部署的 Pair 合约地址。

​	4.	**关键步骤与逻辑**：

​	•	**地址排序**：

​	•	确保 Token A ≠ Token B。

​	•	地址按数值排序，小的地址作为 token0，大的地址作为 token1。

​	•	地址排序通过将地址视为 160 位数字完成。

​	•	**检查与注册**：

​	•	确保 token0 和 token1 未被注册。

​	•	部署后，将 Pair 合约地址存储在 getPair 映射中。

​	5.	**合约创建过程**：

​	•	使用 Create2 部署合约，确保地址可预测性。

​	•	Create2**地址计算公式**：

​	•	取 keccak256 哈希，包含以下信息：

​	•	固定前缀 0xFF。

​	•	部署者地址。

​	•	盐值（随机的 32 字节数据）。

​	•	创建码哈希（合约运行码和构造函数参数的组合）。

​	•	**创建码（Creation Code）**：

​	•	包括运行码（合约的 EVM 字节码）和构造函数参数。

​	6.	initialize**函数的作用**：

​	•	因为构造函数没有参数，所以在部署合约后，通过 initialize 函数设置 token0 和 token1。

​	•	保证 Create2 生成的地址只依赖于 token0 和 token1。

​	7.	**状态更新**：

​	•	完成初始化后，将生成的 Pair 合约地址存储在 getPair 映射中。

​	•	互换 token0 和 token1 的组合也指向同一 Pair 合约。

​	8.	**练习**：

​	•	学员可以尝试基于所学内容，实现并测试 createPair 函数的调用逻辑，理解其部署过程和数据流转。

