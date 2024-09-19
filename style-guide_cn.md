# 智能合约漏洞分割指南

本指南该书了为本仓库贡献的格式和内容,重点关注部署在[兼容以太坊虚拟机](https://ethereum.org/en/developers/docs/evm/)链上的智能合约漏洞.

## 文档结构

- *Markdown (.md)文件*: 主要内容将使用[markdown格式](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)编写，以提高可读性和版本管理.
- *命名统一*: 使用描述性文件名，能够传达讨论的漏洞内容。例如: `unsupported-opcodes.md`, `default-visibility.md`
- *标题层级:* 使用清晰的标题层级 (##, ###, etc.) 来组织内容并改善导航.

## 内容指南

- *漏洞类型:* 在文档开头表明漏洞类型(例如， Unsupported Opcodes不支持的操作码, Reentrancy 重入漏洞, Access Control 访问控制).
- *技术解释:* 几桶简明的技术解释，包含漏洞的潜在影响和利用场景,必要时使用代码片段来说明问题.
- *受影响的链(可选):* 指出哪些兼容EVM的链容易收到漏洞影响，强调任何链特定的注意事项.
- *检测与缓解(可选):* 概述在智能合约审计中检测漏洞的推荐方法，并未开发者提出缓解策略，可在此包含工具和最佳实践.
- *举例(可选):* 如果适用，包含受该漏洞影响的智能合约的实际案例
- *严重性评级(可选):* 考虑引入严重性评级系统，以根据潜在影响对漏洞进行优先级排序.
- *更新README:* 当添加新漏洞及其对应的markdown文件时，记得在`README.md`中更新新条目

## 代码片段格式

- *代码块:* 使用[markdown代码块](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-block) 展示代码片段.
- *语法高亮:* 使用适当的markdown扩展或工具为Solidity代码启用[语法高亮](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks#syntax-highlighting), 以增强可读性
- *注释:* 在代码片段中包含必要的注释，以解释特定行或逻辑.

## 外部引用

- *链接:* 链接到相关资源，比如官方文档，漏洞报告和博客文章，供进一步探索
- *引用:* 使用侵袭的文本内引用，并在专门的"来源"部分引用外部来源

## 其他注意事项

- *目标读者:* 针对智能合约安全感兴趣的广泛受众调整技术细节的深度
- *简明且可执行:* 重点提供可操作的信息，帮助开发者识别并防止漏洞
- *社区贡献:* 鼓励社区贡献，并保持对拉取请求和讨论的包容态度
- *版本控制:* 保持清晰的版本控制系统，以跟踪漏洞的更新和更改
