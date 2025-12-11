### Uniswap V2 的重要合约解析

在 Uniswap V2 中，为实现自动做市商（AMM）功能，主要使用了三个合约：Factory 合约、Router 合约和 Pair 合约。以下是每个合约的关键功能及作用。

#### 1. Factory 合约
- **功能**：Factory 合约用于部署 Pair 合约。
- **作用**：它是一个简单的合约，负责创建和管理不同的 ERC20 代币对的 Pair 合约。例如，当需要创建 ETH/USDT 或 DAI/ETH 的交易对时，Factory 合约会部署对应的 Pair 合约。

#### 2. Pair 合约
- **功能**：Pair 合约用于存储一对 ERC20 代币，同时允许用户添加或移除流动性，并支持代币互换（Swap）。
- **作用**：
  - **流动性提供者**：流动性提供者可以将一对代币（例如 ETH 和 USDT）添加到 Pair 合约中，成为该交易对的流动性提供者，获得交易手续费分成。
  - **交易者**：交易者可以通过 Pair 合约在两个代币间进行兑换（例如 ETH 换 USDT 或 USDT 换 ETH）。
  - **灵活性**：每一对 ERC20 代币会对应一个独立的 Pair 合约，且不必一定是与 ETH 配对。任何两个 ERC20 代币都可以组成一对，如 DAI/MKR。
  
#### 3. Router 合约
- **功能**：Router 合约是用户与 Pair 合约之间的中间合约，旨在安全地管理流动性和代币兑换操作。
- **作用**：
  - **流动性管理**：Router 合约通过调用 Pair 合约，帮助用户安全地添加或移除流动性，避免用户直接与 Pair 合约交互时可能出现的代币锁定错误。
  - **安全兑换**：Router 合约可在单个或多个 Pair 合约间进行代币兑换。例如，当用户需要将 DAI 兑换为 ETH 时，Router 会调用 DAI/ETH 的 Pair 合约完成兑换；若用户需要将 ETH 换为 MKR，Router 会先调用 DAI/ETH 的 Pair 合约，将 ETH 换为 DAI，再调用 DAI/MKR 的 Pair 合约，将 DAI 换为 MKR，最终将 MKR 转给用户。
  - **简化操作**：Router 合约也可以帮助用户创建新的交易对（Pair），如果交易对不存在，Router 会调用 Factory 合约进行创建。

### 总结
Uniswap V2 通过 Factory、Router 和 Pair 这三个合约的相互协作，实现了自动化的流动性提供和代币兑换。