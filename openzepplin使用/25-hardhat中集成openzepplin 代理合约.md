## 安装

~~~ 
npm install --save-dev @openzepplin/hardhat-upgrades
~~~

## 配置
~~~
require("@openzeppelin/hardhat-upgrades");
~~~

Boxv1.js

~~~ js
const  hre = require("hardhat");

  

async function deployBoxv1() {

  // 获取 BoxV1 合约的工厂对象（合约名区分大小写）

  // getContractFactory 会自动从 artifacts 中加载对应编译后的 ABI 和字节码

  const BoxV1 = await hre.ethers.getContractFactory("BoxV1");

  const boxv1 = await hre.upgrades.deployProxy(BoxV1,[1],{initializer:"initialize"});

  await boxv1.waitForDeployment();

  console.log("✅ Boxv1 deployed to:", await boxv1.getAddress());

  console.log("x =", await boxv1.x());

  await boxv1.cal();

    console.log("x after cal() =", await boxv1.x());

  

}

  

deployBoxv1().catch((error) => {

  console.error(error);

  process.exitCode = 1;

});
~~~

~~~
npx hardhat node 

npx hardhat run .\test\Boxv1.js --network localhost
✅ Boxv1 deployed to: 0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512
x = 1n
x after cal() = 2n
~~~
Boxv2.js
~~~js
const  hre = require("hardhat");

  

async function deployBoxv2() {

  // 获取 Boxv1 合约的工厂对象（类似 Java 的类模板）

  // getContractFactory 会自动从 artifacts 中加载对应编译后的 ABI 和字节码

  const Boxv2 = await hre.ethers.getContractFactory("BoxV2");


	// 此处地址是v1的合约地址
  const boxv2 = await hre.upgrades.upgradeProxy("0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512",Boxv2);

  
  

  await boxv2.waitForDeployment();

  console.log("✅ Boxv1 deployed to:", boxv2.address);

  console.log("x =", await boxv2.x());

  await boxv2.cal();

    console.log("x after cal() =", await boxv2.x());

  

}

  

deployBoxv2().catch((error) => {

  console.error(error);

  process.exitCode = 1;

});
~~~

~~~
npx hardhat run .\test\Boxv2.js --network localhost
✅ Boxv1 deployed to: undefined
x = 2n
x after cal() = 4n
~~~


结果如下

![[Pasted image 20251025100434.png]]


