### Uniswap V2 Token Swap流程课程要点总结

#### 一、概述
本课程主要讲解Uniswap V2中的合约及函数调用，重点在于用户调用swap函数时，如何在两个ERC20代币之间进行交换，及其在单跳和多跳交换中的具体流程和机制。

#### 二、ERC20到ERC20代币交换示例
1. **用户调用router合约的swap函数**  
   用户在Uniswap V2中调用`router`合约中的`swapExactTokensForTokens`函数，用于将全部输入代币转换成用户期望的目标代币。在本例中，用户将`WETH`（包装版的ETH，兼容ERC20标准）兑换成`DAI`。

2. **代币转移至pair合约**  
   用户的`WETH`首先从用户钱包转移到`WETH/DAI`的pair合约中。

3. **执行Swap函数计算并转移DAI**  
   在pair合约中，`router`调用`swap`函数，计算用户应收到的`DAI`数量，并将相应的`DAI`转移给用户。这是一个单跳交换（Single Hop Swap）示例，即只涉及一个pair合约的代币交换。

#### 三、ETH到ERC20代币交换
1. **调用swapExactETHForTokens函数**  
   如果用户希望用`ETH`交换成某种ERC20代币，例如`DAI`，则需要调用`swapExactETHForTokens`函数。注意：`ETH`并非ERC20代币，因此调用的是不同的函数。

2. **过程类似，具体区别**  
   `swapExactETHForTokens`的过程与ERC20到ERC20的流程类似，不同之处在于`ETH`首先会被包装为`WETH`以适配ERC20标准，然后再进行后续的交换。

#### 四、多跳交换示例
多跳交换（Multi-Hop Swap）用于在没有直接pair合约的情况下，完成两个代币的交换。在本例中，用户希望将`WETH`转换为`MKR`，但`WETH/MKR`的pair合约不存在，因此采用多跳路径完成兑换。

1. **步骤一：WETH换为DAI**  
   用户首先通过调用`swapExactTokensForTokens`函数，将`WETH`兑换为`DAI`。此时，`WETH`转入`WETH/DAI` pair合约，pair合约中`swap`函数计算`DAI`的数量，并将相应`DAI`转入下一个pair合约。

2. **步骤二：DAI换为MKR**  
   接下来，`router`合约在`DAI/MKR` pair合约中再次调用`swap`函数，将`DAI`兑换为`MKR`。`DAI/MKR` pair合约中计算出用户应得到的`MKR`数量，并将其最终转给用户。

3. **总结：多跳交换的功能**  
   多跳交换允许在没有直接pair合约的情况下，通过中间代币完成代币兑换，以提供更多的流动性选项并优化用户体验。

#### 五、总结
本课程内容覆盖了Uniswap V2中最常用的交换函数和多跳交换流程，让用户了解不同情况下的代币交换逻辑及其实现的具体步骤，帮助用户加深对Uniswap V2合约调用过程的理解和掌握。