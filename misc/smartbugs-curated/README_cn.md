# [SmartBugs Curated](https://github.com/smartbugs/smartbugs-curated)

SB Curated是一个数据集，用于研究用Solidity(以太坊使用的主要语言)编写的智能合约的自动推理和测试。它是作为执行框架SmartBugs的一部分而开发的.该框架允许轻松继承工具，以便对他们进行自动比较(并复制其结果).据我们所知，SmartBugs Curated是同类中最大的数据集.

## 漏洞

SmartBugs Curated提供了根据DASP分类法整理的易受攻击Solidity智能合约集:

|                                     |                                                                        |            |
| :---------------------------------- | :--------------------------------------------------------------------- | :--------- |
| 漏洞                                | 描述                                                                   | 级别       |
| 重入攻击                            | 可重入函数调用导致合约表现出意外的行为                                 | Solidity   |
| [访问控制](dataset/access_control/) | 未正确使用函数修饰符或使用`tx.origin`进行认证                          | Solidity   |
| 未检查的低级调用                    | `call()`,`callcode()`,`delegatecall()`或`send()`调用失败但未检查返回值 | Solidity   |
| 拒绝服务攻击                        | 合约被耗时的计算任务压垮，无法正常运行                                 | Solidity   |
| 随机性错误                          | 恶意矿工可以形象结果的偏向性                                           | Blockchain |
| 抢跑攻击                            | 两个依赖同一合约的交易在同一个区块中被打包，从而导致交易次序被影响     | Blockchain |
| 时间操控                            | 矿工操控区块的时间戳，从而影响合约行为                                 | Blockchain |
| 短地址攻击                          | EVM本身接受不正确填充的参数                                            | EVM        |
