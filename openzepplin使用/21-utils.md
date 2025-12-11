api： 
https://docs.openzeppelin.com/contracts/5.x/api

工具类的都看看 

~~~ js
// SPDX-Lice// SPDX-License-Identifier: MIT

pragma solidity ^0.8.23;

import "@openzeppelin/contracts/utils/math/Math.sol";

  

contract MathUtils{

  

    //  使用 Math 库   增强 uint256 类型

    using Math for uint256;

  

    function testAdd(uint a ,uint b ) external pure{

        Math.tryAdd(a,b);

    }

    // 这两种方式都可以

    function testAdd2(uint a ,uint b) external pure{

        a.tryAdd(b);

    }

}
~~~