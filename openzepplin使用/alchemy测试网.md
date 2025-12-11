![[Pasted image 20251027174814.png]]


不想太麻烦 直接找闲鱼  
账户里面必须有钱  》》 ethereum  以太坊 水龙头领取   》》 
下载okx和币安app


|   |   |
|---|---|
|官方 Google Faucet（需要 GitHub 登录）|https://cloud.google.com/application/web3/faucet/ethereum/sepolia|

|                   |                            |
| ----------------- | -------------------------- |
| 🔗 Alchemy Faucet | https://sepoliafaucet.com/ |
| 72小时可以领取一次        |                            |
|                   |                            |

|   |   |
|---|---|
|🔗 Infura Faucet（需要 Infura 账户）|https://www.infura.io/faucet/sepolia|

|   |   |
|---|---|
|🔗 QuickNode Faucet|https://faucet.quicknode.com/ethereum/sepolia|

领取水龙头 
https://www.alchemy.com/faucets/ethereum-sepolia  
需要人机验证reCrypto
需要有0.0001 eth 大约30人民币

 

![[Pasted image 20251027210059.png]]



## metamask如何切换到sepolia网



https://chainlist.org/?search=sepoli&testnets=true

![[Pasted image 20251027210603.png]]

sepolia区块链浏览器 
https://sepolia.etherscan.io/

![[Pasted image 20251027211340.png]]



alchemy是一个平台 可以部署到区块链网络上 

![[Pasted image 20251027212621.png]]




![[Pasted image 20251028162639.png]]

不同account也有不同的私钥
![[Pasted image 20251028162921.png]]
每个账号每个链上的私钥是一样的 
![[Pasted image 20251028162840.png]]
Deploying contract with account: 0x6a5201C57C78378b06647DE991214716975227f2
counter address is 0x195F67475679D0CE5780a814930c07B1F5b5e4E0

合约地址
https://sepolia.etherscan.io/address/0x195F67475679D0CE5780a814930c07B1F5b5e4E0
![[Pasted image 20251028163431.png]]

部署代码 
~~~ js
// import "@nomicfoundation/hardhat-ethers";

// import { ethers } from "hardhat";

  

// async function deploy() {

  
  

//     const Counter = await ethers.getContractFactory("Counter");

//     const counter = await Counter.deploy();

//     await counter.waitForDeployment();

//     console.log("count address is:", await counter.getAddress());

//     return counter;

// }

  

// async function count(counter:any){

//     await counter.count();

//     console.log("Counter value is:", await counter.getCount());

// }

  

// // 这个相当于main方法吗？ 是的 相当于将两个function串联起来了

// deploy().then(count);

  
  

import "@nomicfoundation/hardhat-ethers";

import { ethers } from "hardhat";

  

async function deploy() {

  
  

    // 获取部署者账户信息 这里没有读取到sepolia_eth中的账户 很奇怪  

    const [deployer] = await ethers.getSigners();

    console.log("Deploying contract with account:", deployer.address);

  

    const Counter = await ethers.getContractFactory("Counter");

    const counter = await Counter.deploy();

    await counter.waitForDeployment();

    console.log('counter address is', await counter.getAddress());

    return counter;

}

  

async function count(counter: any) {

    await counter.count();

    console.log('count is',await counter.getCount());

}

  

deploy().then(count); 
~~~