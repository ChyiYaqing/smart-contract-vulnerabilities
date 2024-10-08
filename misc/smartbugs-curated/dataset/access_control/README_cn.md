# 访问控制

访问控制问题在所有的程序中都很常见，不仅仅是智能合约。实际上，它在[OWASP](https://owasp.org/www-project-top-ten/)(Open Worldwide Application Security Project 开发式全球应用安全项目)前10大安全问题中排名第五.通常，人们通过合约的`public`或`external`函数来访问其功能。不安全的可见性设置为攻击者提供了直接访问合约私有值或逻辑的方法，但有时会更加隐蔽的绕过访问控制.这些漏洞可能会发生在合约使用已弃用的`tx.origin`来验证调用者时,或是在处理复杂的授权逻辑时使用冗长的`require`语句,再或者在代理库或代理合约中随意的使用`delegatecall`.

估计损失：约150，000 ETH(当时约合3,000万美金)

## 攻击场景

智能合约将初始化他的地址指定为合约的所有者。这是一个常见的模式，用于授予特殊权限,例如提取合约资金的能力。不幸的是，该初始化函数可以被任何人调用--即使他已经被调用过.这样就允许任何人称为合约的所有者,并提取其中的资金.

## 示例

在下面的示例中，合约的初始化函数将调用者设置为合约的所有者。然而，该逻辑与合约的构造函数是分离的，并且没有记录它是否已经被调用过.

```solidity
function initContract() public {
 owner = msg.sender;
}
```

在Partity多签名钱包中，初始化函数与钱包本身分离，被定义在一个"库"合约中。用户预期通过`delegateCall`调用库的函数来初始化自己的钱包.不幸的是，如同我们的示例，函数并没有检查钱包是否已经被初始化。更糟糕的是，由于库是一个智能合约，任何人都可以初始化这个库并调用其销毁.

## 参考

摘自[DASP TOP10](https://dasp.co)
