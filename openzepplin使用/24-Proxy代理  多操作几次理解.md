代理模式 解决合约无法变更问题

![[Pasted image 20251024224706.png]]


ERC1967 
UUPS：
	

## 透明和升级合约 TransparentUpgradeableProxy

![[Pasted image 20251024225014.png]]

~~~ js

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.26;

import "@openzeppelin/contracts/proxy/utils/Initializable.sol";

  

contract BoxV1 is Initializable{

  

    uint public x;

    function initialize(uint _val) external initializer{

        x = _val;

    }

  

    function cal() external{

        x = x+1;

    }

  

    function showInvoke() external pure returns(bytes memory){

        return abi.encodeWithSelector(this.initialize.selector,1);

    }

  

}
~~~

~~~ js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.23;

import "@openzeppelin/contracts/proxy/utils/Initializable.sol";

  

contract BoxV2 is Initializable{

  

     uint public x;

  

    function initialize(uint _val) external initializer{

        x = _val;

    }

  

    function cal() external{

        x = x*2;

    }

  

        //有什么作用  呢？  这个函数的作用是返回调用initialize函数的abi编码  

      function showInvoke() external pure returns(bytes memory){

        return abi.encodeWithSelector(this.initialize.selector,1);

    }

  

}
~~~

代理合约 
~~~ js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.23;

import "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";

/**

        * 透明代理模式

        * 透明代理模式是OpenZeppelin提供的一种代理模式

        * 透明代理模式中，代理合约和逻辑合约是分开的    

 */

contract TPUProxy is TransparentUpgradeableProxy{

  

        //  这个构造函数的作用是初始化代理合约

        constructor(address _logic,address initialOwner,bytes memory _data) payable TransparentUpgradeableProxy(_logic,initialOwner,_data){

  

        }

  

        // Receive function to accept plain Ether transfers

        receive() external payable {}

  

        function proxyAdmin() external view returns(address){

                return _proxyAdmin();

        }

        function getImplements() external view returns(address){

            return _implementation();

        }

  

}

~~~




![[Pasted image 20251024234055.png]]