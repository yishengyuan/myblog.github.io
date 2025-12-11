
**eb3j** 

web3j是一个轻量级的Java库，用于在Ethereum网络上集成客户端（节点）。

核心特性： 

- 通过Java类型的JSON-RPC与Ethereum客户端进行交互
- 支持所有的JSON-RPC方法类型
- 支持所有Geth和Parity方法，用于管理账户和签署交易
- 同步或异步的发送客户端请求
- 可从Solidity ABI文件自动生成Java只能合约功能包

引入maven插件 注意版本 编辑器jdk17  此处为4.11
~~~js
<plugin>  
    <groupId>org.web3j</groupId>  
    <artifactId>web3j-maven-plugin</artifactId>  
    <version>4.11.0</version>  
    <configuration>        <soliditySourceFiles/>    </configuration></plugin>
~~~


将json文件复制到resource目录下  
![[Pasted image 20251028110547.png]]
