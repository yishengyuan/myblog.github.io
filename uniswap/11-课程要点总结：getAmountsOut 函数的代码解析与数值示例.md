课程要点总结：`getAmountsOut` 函数的代码解析与数值示例

1. **初始化参数与路径设置**  
   - 在 Uniswap V2 的 `UniswapV2Library` 合约中，`getAmountsOut` 函数用于根据输入代币的数量与交换路径，计算出每次交换的输出。
   - 示例中，我们设置了：
     - **输入数量**：1 WETH（`1e18`）。
     - **路径 `path`**：包含代币地址 WETH、DAI 和 MKR。

2. **初始化 `amounts` 数组**  
   - 根据 `path` 的长度（3）初始化一个 `uint` 数组 `amounts`，用于存储每次交换的代币数量。
   - 将 `amounts[0]` 初始化为 `amountIn`（即 1e18）。

3. **循环处理路径中的每次交换**  
   - `for` 循环遍历路径中的每对代币（例如，WETH -> DAI 和 DAI -> MKR）。
   - **第一次循环 (`i = 0`)**：
     - `path[0]` 为 WETH，`path[1]` 为 DAI。
     - 调用 `getReserves` 函数获取 `WETH/DAI` 交易对的储备量 `ReserveIn` 和 `ReserveOut`。
     - 将 `amounts[0]`（1e18）作为输入，调用 `getAmountOut` 计算输出 DAI 的数量，假设结果约为 `2500` DAI。
     - 更新 `amounts[1]` 为 `2500`，即 `amounts` 数组现在为 `[1e18, 2500, 0]`。

   - **第二次循环 (`i = 1`)**：
     - `path[1]` 为 DAI，`path[2]` 为 MKR。
     - 调用 `getReserves` 函数获取 `DAI/MKR` 交易对的储备量。
     - 将 `amounts[1]`（2500 DAI）作为输入，调用 `getAmountOut` 计算输出 MKR 的数量，假设结果约为 `1.24` MKR。
     - 更新 `amounts[2]` 为 `1.24`，最终的 `amounts` 数组为 `[1e18, 2500, 1.24]`。

4. **结果验证与总结**  
   - 通过实际数值的示例，我们验证了 `getAmountsOut` 函数的工作原理。
   - 函数逐步计算多跳交易路径中的每个交换输出量，最终返回每一步的代币数量数组（例如 `[1 WETH, 2500 DAI, 1.24 MKR]`）。