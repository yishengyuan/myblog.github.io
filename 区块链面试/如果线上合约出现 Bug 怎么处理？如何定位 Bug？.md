

> 如果线上合约出现 bug，我们第一时间会止损：如果合约支持 Pausable，则直接 pause；如果不支持，通过后端/前端网关停用。  
> 然后利用 Tenderly 回放出问题交易，分析 callstack 和 storage diff 复现 Bug。  
> 如果合约是可升级的，会修复逻辑合约并通过 Gnosis Safe 多签升级；如果不可升级，则通过 Router 或迁移合约进行资产迁移。  
> 复盘时会补齐单元测试、引入 fuzz、静态分析与更多安全检查，确保同类问题不再发生。