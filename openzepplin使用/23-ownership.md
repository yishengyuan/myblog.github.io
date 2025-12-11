Ownable组件： 
所有权管理
权限控制
所有权转移 
所有权放弃 


~~~ js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/Ownable.sol";

  

contract MyContract is Ownable{

  

    constructor(address initialOwner) Ownable(initialOwner){

    }

  

    // transferOwner 可以转移权限

  

    function normalThing() external{

  

    }

  

    function specialThing() external onlyOwner{

  

    }

  

}
~~~
![[Pasted image 20251024224403.png]]
所有权转移 
renounceOwner放弃所有者  合约处于无主状态




