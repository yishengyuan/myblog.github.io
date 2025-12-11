
# 一、提案投票到底可以提案哪些内容？

提案（Proposal）本质上就是**让社区治理合约执行某件事**，一般可以分为 3 类：

## **① 参数调整类（最常见）**

用于修改协议关键参数，例如：

- 手续费比例调整（如 0.3% → 0.05%）
    
- 流动性激励分配比例
    
- 质押利率修改
    
- DAO 国库资金使用（拨款/资助）

## **协议升级类（技术层面变更）**

需要链上执行的技术修改，例如：

- 升级智能合约版本（V2 -> V3）
    
- 新增功能（例如增加新交易路由）
    
- 修复漏洞
    
- 调整治理逻辑本身
    

## **③ 社区 / 管理类（链下决策）**

不直接修改链上合约，但影响社区方向：

- 修改品牌、UI、网站内容
    
- 社区预算规划
    
- 生态发展路线图
    

这种往往通过 Snapshot 线下投票，然后由多签执行。


# ✔ 二、提案通过/执行需要多少比例？

不同 DAO 的阈值差别极大，但常见的流程是：

# **DAO 提案通过 = 达到投票门槛 + 赞成票占多数**

一般分为三个关键参数：

## **① 提案门槛（Proposal Threshold）**

最低要持有多少治理代币才能提出提案。

例子（真实协议）：

- Uniswap：**250k UNI** 才能发起链上提案
    
- Compound：**100k COMP** 才能发起提案
    

---

## **② 法定投票量（Quorum）**

要有多少票参与，提案才能生效，避免“少数人操控”。

例子：

- Uniswap：**4000万 UNI** 必须参与（约总量 4%）
    
- Compound：**400k COMP**
    

---

## **③ 赞成率（For / Against）**

只要参与投票的票里，**赞成票 > 反对票** 就会通过。  
没有固定比例要求，一般是：

> ✔ 赞成票 > 反对票  
> ✔ 并满足“法定投票量”

就会执行到链上。

---

# ✔ 三、以 Uniswap 为例（你最近问 Uniswap 比较多）

**Uniswap 的提案内容：**

你可以提案修改：

- 手续费分成（Fee Switch）
    
- 新链部署（例如 Uniswap V4 迁移）
    
- DAO 国库资金动用
    
- 治理机制修改（投票系统）
    

**执行条件（真实参数）：**

|条件|要求|
|---|---|
|提案资格|至少持有 **250,000 UNI**|
|投票法定人数|至少 **40,000,000 UNI** 参与|
|通过条件|赞成票 > 反对票|
|执行延迟|Timelock 延迟 2 天（给用户安全预期）|

---

# ✔ 四、简单总结（你只需要记住）

### **提案可以修改：协议参数、合约升级、资金使用、治理规则等。**

### **一般执行条件：**

1. **发起提案需要一定代币数量**
    
2. **投票参与量必须达到法定人数（Quorum）**
    
3. **赞成票超过反对票**
    
4. （通常会有 time-lock 延迟）
    

不同 DAO 不一样，但这是最常见模式。




Dao  去中心化自治组织 
没有行政部门  提案投票执行

![[Pasted image 20251025110740.png]]


![[Pasted image 20251025162422.png]]


## 核心合约
![[Pasted image 20251025162619.png]]

getVotes
getPastBotes
delegate  
delegateBySig


GovernerSetting： 
![[Pasted image 20251025163044.png]]

提案需要的代币数量
在提案移除多长时间，我们发起投票 
投票的持续时间 

GovernerCountingSimple： 
	定义了三种类型： 拒绝支持弃权  
	提供了投票是否成功
	投票是否有效 
	GovernorBotesQuorumFraction
	表决多少比例才能生效  占比达到40% 才是有效的结果
	
![[Pasted image 20251025163309.png]]


![[Pasted image 20251025163429.png]]

提案的事件锁：
	schedule：将交易执行加入时间队列
	cancel：取消执行
	excute：执行操作
_queueOperation:调用schedule_

_excuteOperation:会执行操作 

## 演示

![[Pasted image 20251025163828.png]]


MyToken.sol
~~~json
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

  

// 导入所需的 OpenZeppelin 合约

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

import {ERC20Votes} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";

import {Nonces} from "@openzeppelin/contracts/utils/Nonces.sol";

import "@openzeppelin/contracts/access/Ownable.sol";

  

/**

 * @title MyToken - 治理代币合约

 * @dev 实现了以下功能：

 * - ERC20: 基础代币功能

 * - ERC20Permit: 允许通过签名进行授权，无需发送交易

 * - ERC20Votes: 为治理提供投票功能，包括投票权委托

 * - Ownable: 所有权管理，只有所有者可以铸造代币

 */

contract MyToken is ERC20, ERC20Permit, ERC20Votes, Ownable {

    /**

     * @dev 构造函数

     * ERC20("MyToken", "MTK"): 设置代币名称和符号

     * ERC20Permit("MyToken"): 初始化 permit 功能

     * Ownable(msg.sender): 将部署者设置为合约所有者

     */

    constructor() ERC20("MyToken", "MTK") ERC20Permit("MyToken") Ownable(msg.sender) {}

  

    /**

     * @dev 重写 _update 函数以支持 ERC20Votes 功能

     * @param from 发送方地址

     * @param to 接收方地址

     * @param amount 转账金额

     * 当代币转移时，需要同时更新投票权重

     */

    function _update(address from, address to, uint256 amount) internal override(ERC20, ERC20Votes) {

        super._update(from, to, amount);

    }

  

    /**

     * @dev 重写 nonces 函数以支持 ERC20Permit

     * @param owner 账户地址

     * @return 返回该账户的当前 nonce 值

     * 用于防止重放攻击，每个签名只能使用一次

     */

    function nonces(address owner) public view virtual override(ERC20Permit, Nonces) returns (uint256) {

        return super.nonces(owner);

    }

  

    /**

     * @dev 铸造新代币

     * @param account 接收代币的地址

     * @param amount 铸造数量（会自动乘以 decimals）

     *

     * 特点：

     * 1. 只有合约所有者可以调用（onlyOwner）

     * 2. 如果接收者还没有投票代表，自动将其设为自己的代表

     * 3. 金额会自动乘以 decimals() 转换为正确的小数位数

     */

    function mint(address account, uint amount) external onlyOwner {

         // 如果账户没有委托投票权，将投票权委托给自己

         if(delegates(account) == address(0)){

            _delegate(account, account);

         }

         // 铸造代币，金额需要乘以 decimals

        _mint(account, amount * 10**decimals());

    }

}
~~~

MyGoverner.sol

~~~js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

  

import {IGovernor, Governor} from "@openzeppelin/contracts/governance/Governor.sol";

import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

import {GovernorVotes} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

import {GovernorVotesQuorumFraction} from "@openzeppelin/contracts/governance/extensions/GovernorVotesQuorumFraction.sol";

import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

import {IERC165} from "@openzeppelin/contracts/interfaces/IERC165.sol";

  

contract MyGovernor is

    Governor,

    GovernorCountingSimple,

    GovernorVotes,

    GovernorVotesQuorumFraction

{

    /**

     * @dev 构造函数初始化治理合约

     * @param _token 用于投票的代币合约地址（实现了IVotes接口）

     * Governor("MyGovernor") - 设置治理合约名称

     * GovernorVotes(_token) - 设置投票代币

     * GovernorVotesQuorumFraction(4) - 设置法定人数为4%

     */

    constructor(

        IVotes _token

    ) Governor("MyGovernor") GovernorVotes(_token) GovernorVotesQuorumFraction(4) {}

  

    /**

     * @dev 设置提案创建后到投票开始的延迟区块数

     * @return 返回延迟区块数为2（约1天）

     * 作用：让代币持有者有时间了解提案内容并决定是否投票

     */

    function votingDelay() public pure override returns (uint256) {

        return 2; // 约1天 (按每15秒一个区块计算)

    }

  

    /**

     * @dev 设置投票持续的区块数

     * @return 返回投票期区块数为2（约1周）

     * 作用：定义投票的开放时间，让代币持有者有充足时间参与投票

     */

    function votingPeriod() public pure override returns (uint256) {

        return 2; // 约1周 (按每15秒一个区块计算)

    }

  

    /**

     * @dev 设置提交提案所需的最低代币数量

     * @return 返回0，表示任何代币持有者都可以提交提案

     * 作用：控制提案权限，防止垃圾提案，这里设为0表示完全开放

     */

    function proposalThreshold() public pure override returns (uint256) {

        return 0; // 无门槛，任何人都可以提案

    }

}
~~~


~~~js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.24;

contract BizContract{

  

    uint public x;

  
  

    function call(uint _var) external  payable {

        x=_var;      

    }

  

    function showCallData() external pure  returns(bytes memory){

        return abi.encodeWithSelector(this.call.selector,1);

    }

}
~~~

~~~js

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.26;

contract Test{

    function clock() external   returns(uint256){

        return block.number;

    }

  

}
~~~


- 以太坊主网平均出块时间约为12秒
- 某些测试网络可能会有不同的出块时间
- 本地开发环境可能需要较小的数值以便测试


治理合约中的投票流程和主要功能：

### 1. 投票的主要流程

1. **提案创建**
    
    - 代币持有者可以创建提案
    - 提案包含要执行的操作（如调用某个合约的函数）
    - 需要达到 `proposalThreshold()` 设定的最低代币数量才能创建提案
2. **延迟期** (`votingDelay`)
    
    - 提案创建后不会立即开始投票
    - 等待 `votingDelay` 个区块
    - 这段时间用于让社区了解和讨论提案
3. **投票期** (`votingPeriod`)
    
    - 持续 `votingPeriod` 个区块
    - 代币持有者可以进行投票
    - 投票选项通常有：赞成(For)、反对(Against)、弃权(Abstain)
4. **法定人数**
    
    - 通过 `GovernorVotesQuorumFraction(4)` 设置为4%
    - 意味着至少需要4%的总投票权参与投票
    - 防止少数人控制决策
### 2. 投票权的来源和管理

1. **投票权重**
    
    - 来自 ERC20Votes 代币的余额
    - 可以委托给其他地址
    - 计算投票权时会检查提案创建时的余额快照
2. **委托机制**
~~~
// 在代币合约中
function delegate(address delegatee) public {
    _delegate(msg.sender, delegatee);
}
~~~

**投票计数** (通过 GovernorCountingSimple)
~~~
function castVote(
    uint256 proposalId,
    uint8 support
) public virtual returns (uint256) {
    // 记录投票
    // 0 = Against, 1 = For, 2 = Abstain
}
~~~

![[Pasted image 20251025165827.png]]

![[Pasted image 20251025165911.png]]

## 为什么点击clock后块高会增加1  块高是什么？ 

![[Pasted image 20251025183923.png]]



![[Pasted image 20251025183949.png]]

keccak256 加密
https://emn178.github.io/online-tools/keccak_256.html
![[Pasted image 20251025184514.png]]