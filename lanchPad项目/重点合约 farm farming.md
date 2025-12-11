![[Pasted image 20251030231958.png]]
Defi platfrom

lender deposit crpto assets

borrower

lends crypto

smart contracts to liquidate

in case of repayment failure

requests loan

lends crypto

![[Pasted image 20251030232012.png]]

1. 以太坊无法通过cron机制定时计算收益
    
2. 任意用户的充值和提取都影响分配的比例
    
3. 时间变化会影响用户的支配，但无法保存每一次变化的结果
![[Pasted image 20251030232030.png]]
LP池 ：流动性池（Liquidity Pool）

## ==AccERC20PerShare 每一份代币获得的奖励（难点）==

nrOfSecond: 上一次记录到现在为止已经经过的时间

rewar ： 每一秒获得的奖励

mul allocPoint当前池的比重/totalAllocPoint总池的比重

erc20reward lpSupply总流动性供应量

accErc20PerShare： F(seconds，rewardRate)

解决了不同用户，不同时间以及时间的流逝对整个奖励的影响。

```solidity
计算奖励 公式：奖励数量=（最后奖励时间-上次奖励时间）*每秒奖励数量*池子分配分数/总分配分数
```
![[Pasted image 20251030232051.png]]

## rewardDebt
![[Pasted image 20251030232106.png]]

从第三秒到第四秒用户的回馈的奖励
