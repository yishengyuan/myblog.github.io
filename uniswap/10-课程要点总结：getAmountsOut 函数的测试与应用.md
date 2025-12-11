课程要点总结：`getAmountsOut` 函数的测试与应用

1. **`getAmountsOut` 函数的背景与测试目的**  
   - `getAmountsOut` 函数位于 `UniswapV2Library` 中，用于计算每次交换的输出数量。为了理解其实际效果，我们将编写一个测试并在主网模拟环境（Main Network Fork）上执行。
   - 通过测试，我们可以获得真实的代币交换结果，进一步验证 `getAmountsOut` 的计算逻辑。

2. **测试准备**  
   - 在 Foundry 中编写测试，导入相关合约接口与地址，例如 ERC20 接口、Uniswap V2 路由器（`UniswapV2Router02`）和代币合约地址（如 WETH、DAI、MKR）。
   - 初始化测试所需的代币与路由器：
     - 设置交易路径 `path`：数组长度为 3，包含 WETH、DAI、MKR 的合约地址。
     - 设置输入数量 `amountIn` 为 `1e18`，表示 1 WETH（WETH 有 18 位小数）。

3. **调用 `getAmountsOut` 函数**  
   - 在 `UniswapV2Router02` 合约中调用 `getAmountsOut` 函数，传入以下参数：
     - 输入数量 `amountIn`：即 1 WETH。
     - 路径 `path`：包括 WETH -> DAI -> MKR。
   - 函数返回一个数组 `amounts`，其中：
     - `amounts[0]` 为输入数量（1 WETH）。
     - `amounts[1]` 为 WETH 兑换出的 DAI 数量。
     - `amounts[2]` 为 DAI 兑换出的 MKR 数量。

4. **控制台输出与验证**  
   - 使用 Foundry 中的 `console2.log` 输出 `amounts` 数组的值，用于记录每个交换步骤的结果。
     - `amounts[0]`：输入的 WETH 数量（应为 1e18）。
     - `amounts[1]`：由 WETH 兑换得出的 DAI 数量。
     - `amounts[2]`：由 DAI 兑换得出的 MKR 数量。
   - 通过 `Forge test --fork-url` 命令在主网分叉上执行测试，输出的结果将显示 WETH 兑换为 DAI 及 DAI 兑换为 MKR 的数量。

5. **实际数值验证**  
   - 测试结果的数值可以作为 `UniswapV2Library.getAmountsOut` 函数的实例数据，用于进一步理解其计算方式与逻辑验证。