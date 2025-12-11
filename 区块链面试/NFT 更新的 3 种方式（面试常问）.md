## **方式 1：更改 tokenURI（链上操作）——可以更新 metadata**

ERC721 标准提供：

`function _setTokenURI(uint256 tokenId, string memory _tokenURI)`

如果合约设计得允许 owner 调用：

`function setTokenURI(uint256 tokenId, string calldata newURI) external onlyOwner {     _setTokenURI(tokenId, newURI); }`

▶ 调用 setTokenURI 就可以 **更新 NFT 的 metadata 地址**。  
元数据文件内容可以换，图片也可以换。

### 常见用法：

- 盲盒 Reveal（从盲盒 → 真正图片）
    
- 游戏 NFT 变身（升级属性）
    
- 动态 NFT（随数据变化）
## **方式 2：metadata link 固定，但链下文件可更新（Web2 服务）**

tokenURI 不变，例如：

`https://myserver.com/nft/123`

后台服务器可以更新 JSON 内容：

- 修改 image 地址
    
- 修改 attributes
    
- 修改描述
    

NFT 查看器（Opensea 等）会刷新缓存后显示新内容。

▶ **这是大多数 NFT 的做法**。

## **方式 3：可升级合约（Proxy）→ 更新 NFT 逻辑**

如果 NFT 用：

- UUPS
    
- Transparent proxy
    

则可以：

- 升级合约逻辑
    
- 修改 mint 规则
    
- 修改 tokenURI 生成方式
    
- 增加/修改 setTokenURI 权限控制
    

▶ 这种方式对 CEX、游戏、链游特别常用。

# 🎮 **动态 NFT 是怎么做到变化的？**

dynamic NFT 技术路径：

### ✔ 外部调用 tokenURI 动态生成内容

tokenURI 返回：

`data:application/json;base64,...`

内容根据链上状态计算，例如：

- 游戏角色等级提升 → 自动生成新图片
    
- 天气 NFT → 根据外部数据变化
    
- 期权 NFT → 随市场价格波动显示不同 UI
    

这是**链上逻辑控制 + 链下渲染**。

---

# 🚫 **不能更新的情况**

如果 NFT 的 tokenURI 指向：

`ipfs://xxxxxx`

并且 metadata 文件是完全不可变的（IPFS CID 写死），则：

❌ metadata 也不能更新  
❌ 图片无法替换  
❌ 属性无法改变

因为 **IPFS CID 是根据内容 hash 计算的**，文件改动内容会产生新 CID。

但如果合约允许修改 tokenURI→新 CID，那就可以更新。

---

# 🧨 **总结（面试可直接背）**

> **NFT 本体（tokenId & owner）不可修改，但 NFT 的 metadata 通常是可更新的。**
> 
> NFT 更新方式主要有三种：
> 
> 1. **链上更新 tokenURI（重指向新的 metadata）**
>     
> 2. **链下服务器更新 metadata 文件内容（tokenURI 不变）**
>     
> 3. **通过可升级合约修改 NFT 逻辑（proxy）**
>     
> 
> 本质是：NFT 是链上的“索引”，真实内容在链下，因此可以更新。