# [Solidity安全性: 已知攻击和常见反模式的全面列表](https://blog.sigmaprime.io/solidity-security.html)

虽然Solidity仍处于起步阶段，但它已经被广泛采用，并用于贬义我们今天看到的许多以太坊智能合约的字节码. 开发者和用户在发现该语言及其EVM(以太坊虚拟机)细微差别时，吸取了许多惨痛的教训。这篇文章旨在称为相对深入且最新的介绍性文章，详细描述Solidity开发者过去犯过的错误，以防止未来的开发者重蹈覆辙.

## 目录

### 1. 重入攻击

### 2. 算数溢出/下溢

### 3. 意外接受的以太币

### 4. 委托调用(Delegatecall)

### 5. 默认可见性

### 6. 熵的幻觉

### 7. 外部合约引用

### 8. 短地址/参数攻击

### 9. 未检测的CALL返回值

### 10. 竞态条件/抢跑

### 11. 拒绝服务工具(DOS)

### 12. 区块时间戳操控

### 13. 构造函数

### 14. 未初始化的存储指针

### 15. 浮点数和数值精度

### 16. tx.origin认证

## Ethereum Quirks

* 无密钥以太币
* 一次性地址
* 单次交易空投

## 参考

* [以太坊WIKI-安全](https://github.com/ethereum/wiki/wiki/Safety)
* [Solidity文档-安全考虑](https://github.com/sigp/solidity-security-blog/blob/master/solidity.readthedocs.io/en/latest/security-considerations.html)
* [ConsenSys - 以太坊智能合约最佳实践](https://consensys.github.io/smart-contract-best-practices)
* [以太坊安全漏洞、攻击及其修复历史](https://applicature.com/blog/history-of-ethereum-security-vulnerabilities-hacks-and-their-fixes)
* [去中心化应用安全项目(DASP)2018年十大攻击](http://www.dasp.co/)
* [以太坊智能合约攻击概述](https://eprint.iacr.org/2016/1007.pdf)
* [以太坊智能合约安全性](https://medium.com/cryptronics/ethereum-smart-contract-security-73b0ede73fa8)
* [Lessons Learnt from the Underhanded Solidity Contest](https://medium.com/@chriseth/lessons-learnt-from-the-underhanded-solidity-contest-8388960e09b1)
