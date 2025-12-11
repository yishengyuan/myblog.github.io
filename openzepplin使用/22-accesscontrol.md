![[Pasted image 20251024220334.png]]

founder： adminrole创建者 管理员 为每一个角色分配权限 
miner：矿工
user： 普通用户

defaultAdminRole： 为每一个角色分配角色
![[Pasted image 20251024220626.png]]


![[Pasted image 20251024220654.png]]

角色定义： default_admin_role 自动逸角色
权限控制： onlyRole修饰符  
角色分配： grantRole函数 revokeRole函数 
角色检查： hasRole函数 
角色管理： _setRoleAdmin函数  getRoleAdmin函数


~~~ js
// SPDX-License-Identifier: UNLICENSED

pragma solidity ^0.8.24;

import "@openzeppelin/contracts/access/AccessControl.sol";

  

// 可以对数据进行加签

contract  MyContract is AccessControl{

  

    bytes32 public constant  ROLE_MANAGER = keccak256("ROLE_MANANGER");

  

    bytes32 public constant ROLE_NORMAL = keccak256("ROLE_NORMAL");

  

    constructor(){

        _grantRole(DEFAULT_ADMIN_ROLE,_msgSender());

    }

  

    function setRoleAdmin() external onlyRole(DEFAULT_ADMIN_ROLE){

        //  设置 ROLE_NORMAL 的管理员为 ROLE_MANAGER

        _setRoleAdmin(ROLE_NORMAL,ROLE_MANAGER);

    }

    function normalThing() external onlyRole(ROLE_NORMAL){

  

    }

  

    function specialThing() external onlyRole(ROLE_MANAGER){

  

    }

}
