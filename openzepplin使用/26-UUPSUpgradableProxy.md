

没有proxyAdmin的存在 
由业务实现进行管控 
uups更加简洁 
更加节省gas
![[Pasted image 20251025101049.png]]

## 代码集成 

UUPSV1.sol
~~~js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.23;

import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

  
  

contract UUPSV1 is Initializable,UUPSUpgradeable,OwnableUpgradeable{

    uint public x;

  

    constructor(uint _var){

        x = _var;

    }

  

    // 检查本次更新是否具备更新权限 需要有业务合约自己负责 负责权限分配

    function _authorizeUpgrade(address implement) internal override{

  

    }

    //  构造函数不会被代理，所以使用initialize函数代替

    // 提供initilizer修饰符确保初始化函数只能调用一次

    // 初始化 相当于constructor

    function initialize(uint _var) external initializer{

        x = _var;

        __Ownable_init(msg.sender);

    }

  

    function cal() external{

        x = x+1;

    }

  

    function showCode() external pure returns(bytes memory){

        // 做什么用？   这个函数用于展示合约的初始化代码

        return abi.encodeWithSelector(UUPSV1.initialize.selector, 1);

    }

}


contract UUPSV2 is Initializable,UUPSUpgradeable,OwnableUpgradeable{

    uint public x;

  

     constructor(uint _var){

        x = _var;

    }

  

    function _authorizeUpgrade(address implement) internal override{

  

    }

  

    function initialize(uint _var) external initializer{

        x = _var;

        __Ownable_init(msg.sender);

    }

  

    function cal() external{

        x = x*2;

    }

  

}
~~~


![[Pasted image 20251025104155.png]]


## hardhat中集成


~~~js
const  hre = require("hardhat");

  

async function main() {

  

  // getContractFactory 会自动从 artifacts 中加载对应编译后的 ABI 和字节码

  const UUPSV1 = await hre.ethers.getContractFactory("UUPSV1");

  

  console.log("🚀 Deploying UUPS proxy...");

  

  // ---------------------------

  // 第一步：部署 UUPS 代理合约

  // ---------------------------

  // upgrades.deployProxy() 会自动：

  // 1. 部署逻辑合约（implementation）

  // 2. 部署代理合约（proxy）

  // 3. 通过 proxy 调用 initialize() 完成初始化

  const uupsV1 = await hre.upgrades.deployProxy(

    UUPSV1,  // 要部署的逻辑合约工厂

    [1],      // initialize() 的参数，这里传入 1

    {

      initializer: "initialize", // 指定初始化函数名

      kind: "uups",              // 指定代理类型为 UUPS（还有 transparent 可选）如果不传默认为透明可升级合约

    }

  );

  

  // 等待部署完成

  await uupsV1.waitForDeployment();

  

  // 输出代理合约地址

  console.log("✅ Proxy deployed to:", await uupsV1.getAddress());

  

  // ---------------------------

  // 第二步：与合约交互

  // ---------------------------

  // 从代理读取变量 x 的值（注意：所有读写都通过代理合约完成）

  console.log("x =", (await uupsV1.x()).toString());

  

  // 调用逻辑函数 cal()（例如让 x 自增）

  const tx = await uupsV1.cal();

  await tx.wait();  // 等待交易上链

  

  // 再次读取 x，看是否更新成功

  console.log("x after cal() =", (await uupsV1.x()).toString());

  

  // ---------------------------

  // 第三步：升级逻辑合约到 V2

  // ---------------------------

  // 获取新的逻辑合约工厂 UUPSV2

  const _UUPSV2 = await hre.ethers.getContractFactory("UUPSV2");

  

  // 调用 upgrades.upgradeProxy() 进行升级

  // 它会：

  // 1. 部署新的逻辑合约 UUPSV2

  // 2. 修改代理的实现指针（implementation）指向新合约

  // 3. 保留原有的存储状态（如变量 x）

  await hre.upgrades.upgradeProxy(await uupsV1.getAddress(), _UUPSV2);

  

  console.log("🆙 Proxy upgraded to V2");

  

  // 升级后继续使用旧的代理对象与合约交互

  // 存储仍然保持不变（升级不会重置变量）

  console.log(await uupsV1.x());

  

  // 调用升级后的 cal()（可能有不同逻辑）

  await uupsV1.cal();

  

    console.log(await uupsV1.x());

}

  

// main() 入口函数（标准 Hardhat 脚本写法）

// 捕获异常并退出

main().catch((error) => {

  console.error(error);

  process.exitCode = 1;

});
~~~

