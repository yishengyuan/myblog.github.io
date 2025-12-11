![[Pasted image 20251024210209.png]]

## Hardhat 的两种运行模式

### 1. **内存模式（In-process Hardhat Network）** - 你当前使用的

当你直接运行 `npx hardhat run script.js` 时，Hardhat 会：

- 在**内存中**启动一个临时的以太坊网络
- 自动创建20个测试账户
- 脚本运行完毕后，这个临时网络就消失了

### 2. **独立节点模式（Standalone Hardhat Network）**

当你运行 `npx hardhat node` 时，会启动一个持久的本地节点。


npx hardhat node

npx hardhat run .\scripts\network-info.js --network localhost   独立节点模式  



npx hardhat run .\scripts\network-info.js --network  内存模式

~~~ js

// SPDX-License-Identifier: UNLICENSED

pragma solidity ^0.8.24;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

import "@openzeppelin/contracts/utils/cryptography/MessageHashUtils.sol";

  

// 可以对数据进行加签

contract  VerifySignature2{

  

    using ECDSA for bytes32;

    using MessageHashUtils for bytes32;

    // str: C2N

    // signature: 0xb30e06f66494c516e0472bb7afce2dbed3b8800b26fdf12e6593d5abf24167e2647283418db2a446deee32e692f0984a56cdca26ef5ec18cea0b64ab4f7450c11c

    // 返回签名者地址 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266

    function recover(string memory str,bytes memory signature) external pure returns(address){

        //  先对字符串进行哈希处理  

        bytes32 hash = keccak256(bytes(str));

  

        // toEthSignedMessageHash是ecdsa的方法吗？  是的，它是用于将消息哈希转换为以太坊签名消息格式的方法。

        // recover是ecdsa的方法吗？ 是的，它是用于从签名中恢复签名者地址的方法。

        return hash.toEthSignedMessageHash().recover(signature);

    }

}

~~~ 


