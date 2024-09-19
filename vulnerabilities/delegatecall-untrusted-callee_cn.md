# 委托调用(Delegatecall)到不可信的被调用方

`Delegatecall` 是消息调用的一种特殊变体。它几乎与常规消息调用相同，区别在于目标地址在调用合约的上下文中执行,并且`msg.sender`和`msg.value`保持不变. 简而言之，`delegatecall`允许其他合约修改调用合约的存储.

由于`delegatecall`对合约有很大的控制力，因此必须仅在与可信合约(如你自己的合约)交互时使用. 如果目标地址来自用户输入，务必确保它是一个可信的合约.

## 示例

考虑以下错误使用`delegatecall`导致漏洞的合约:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.16;

contract Proxy {
    address public owner; // 合约所有者地址

    constructor() { // 构造函数，初始化合约所有者为部署合约的地址
        owner = msg.sender;
    }

    // 转发函数,使用delegatecall将调用转发到指定的合约地址(callee)
    // 参数:
    // callee - 被调用合约的地址
    // _data - 要调用的函数及其参数的数据
    function forward(address callee, bytes calldata _data) public { 
        require(callee.delegatecall(_data), "Delegatecall failed"); // 使用delegatecall调用目标合约，并确保调用成功
    }
}

contract Target {
    address public owner; // 合约所有者的地址

    function pwn() public { // pwn 函数，恶意者调用该函数后，会讲所有者改为调用者
        owner = msg.sender;
    }
}

contract Attack {
    address public proxy; // 代理合约的地址

    constructor(address _proxy) { // 构造函数，初始化代理合约的地址
        proxy = _proxy;
    }

    // 攻击函数, 利用代理合约将pwn函数调用转发到目标合约
    // 参数:
    // target - 目标合约的地址
    function attack(address target) public {
        Proxy(proxy).forward(target, abi.encodeWithSignature("pwn()")); // 通过代理合约的forward函数调用目标合约的pwn函数
    }
}
```

在此示例中，`Proxy`合约使用`delegatecall`将它接收到的任何调用转发到用户提供的地址. `Target`合约包含一个`pwn()`函数调用，该函数会讲合约的所有者改为调用者.

`Attack`合约利用这种设置，通过调用Proxy合约的`forward`函数，传递`Target`合约的地址和编码后的函数调用pwn(). 这导致`Proxy`合约的存储被修改, 具体来说，`owner` 变量被设置为攻击者的地址.

## 缓解措施

为了缓解与不可信被调用方使用`delegatecall`相关的风险，可考虑以下策略:

1. **白名单可信合约**: 确保用于`delegatecall`的目标地址是你控制的合约，或是经过验证和信任的合约列表中的合约.

2. **限制委托调用的范围**: 仅在特定的、可控的操作中使用`delegatecall`. 除非绝对必要，否则避免将其作为通用函数暴露.

## 参考资料

- [SWC Registry: SWC-112](https://swcregistry.io/docs/SWC-112)
- [Solidity Documentation: Delegatecall](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#delegatecall-and-libraries)
- [Sigma Prime: Solidity Security](https://blog.sigmaprime.io/solidity-security.html#delegatecall)

```markdown

```

- [Ethereum Stack Exchange: Difference Between Call, Callcode, and Delegatecall](https://ethereum.stackexchange.com/questions/3667/difference-between-call-callcode-and-delegatecall)
