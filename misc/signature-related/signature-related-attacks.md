# 签名相关攻击

In smart contract systems, signatures are powerful tools that serve critical functions. The EVM features the ecrecover precompile, allowing for native signature validity checks and recovery. This function is predominantly used in the context of authorization, data validity checks, and facilitating gas-less transactions. However, signature systems in smart contracts can malfunction in various ways, often leading to severe consequences.

在智能合约系统中，签名是关键的工具，用于实现重要的功能。EVM提供了[`ecrecover`](https://soliditydeveloper.com/ecrecover)预编译函数, 允许进行原生签名有效性检查和恢复。这个函数通常用于授权、数据有效性检查以及无燃气费交易。然而，签名系统在智能合约中kennel会出现多种故障，通常会导致严重的后果.

## 缺失的验证

最常见的漏洞之一是当`ecrecover`遇到错误并返回无效地址时，缺少必要的验证.

```solidity
function recover(uint8 v, bytes32 r, bytes32 s, bytes32 hash) external {
    address signer = ecrecover(hash, v, r, s);
    // 对 hash 进行更多操作
}
```

在这个示例中，缺少对`address(0)`的检查.这种遗漏使攻击者可以提交无效签名并通过任意有效载荷作为合法签名。一个简单有效的解决方案是在代码中加入以下检查:

```solidity
require(signer != address(0), "invalid signature");
```

更好的方法是使用OpenZeppelin的ECDSA库，因为它会在遇到无效签名时自动回滚.

## 重放攻击

重放攻击发生在签名和系统之间没有去重机制的情况下。重放攻击的原因通常是签名没有被正确作废,或者系统中缺少nonce. 以下示例展示了智能合约签名系统的不同攻击角度及其迭代。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract OwnerAction {
    using ECDSA for bytes32;

    address public owner;

    constructor() payable {
        owner = msg.sender;
    }

    function action(uint256 _param1, bytes32 _param2, bytes memory _sig) external {
        bytes32 hash = keccak256(abi.encodePacked(_param1, _param2));
        bytes32 signedHash = hash.toEthSignedMessageHash();
        address signer = signedHash.recover(_sig);

        require(signer == owner, "Invalid signature");

        // 使用 `param1` 和 `param2` 执行授权操作
    }
}
```

在此场景中，如果攻击者拥有签名人的签名，则可以重复执行同一操作。例如，如果签名人签署了一笔授权转账的交易，攻击者可以重放签名，多次转移资金，从而耗尽合约资金。为了解决此问题，可以通过在提交签名后作废每个签名，或者在签名有效载荷中增加nonce以防止多次签署相同的操作.

以下代码展示签名作废机制和与nonce相关的业务逻辑

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract OwnerAction {
    using ECDSA for bytes32;

    address public owner;
    mapping(bytes32 => bool) public seenSignatures;

    constructor() payable {
        owner = msg.sender;
    }

    function action(uint256 _param1, bytes32 _param2, uint256 _nonce, bytes memory _sig) external {
        bytes32 hash = keccak256(abi.encodePacked(_param1, _param2, _nonce));
        require(!seenSignatures[hash], "Signature has been used");

        bytes32 signedHash = hash.toEthSignedMessageHash();
        address signer = signedHash.recover(_sig);
        require(signer == owner, "Invalid signature");

        seenSignatures[hash] = true;

        // 使用 `param1` 和 `param2` 执行授权操作
    }
}
```

即使是这个增强版合约也并非完全安全。如果系统部署在多个链上，或者签名地址在其他链上有使用，重放攻击依然是一个潜在威胁。

## 跨链重放攻击

跨链重放攻击发生在签名可以在不同区块链系统中重复使用的情况下. 一旦签名在某条链上使用并作废，攻击者仍可以复制该签名并在另一条链上使用，出发不必要的状态变化。这对跨链部署、且代码相同的智能合约系统构成了重大威胁.

为了缓解这种风险，可以在签名有效载荷中编码链ID并在执行操作时对其进行验证。

```solidity
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract OwnerAction {
    using ECDSA for bytes32;

    address public owner;
    mapping(bytes32 => bool) public seenSignatures;

    constructor() payable {
        owner = msg.sender;
    }

    function action(uint256 _param1, bytes32 _param2, uint256 _nonce, uint256 _chainId, bytes memory _sig) external {
        require(_chainId == block.chainid, "Invalid chain ID");

        bytes32 hash = keccak256(abi.encodePacked(_param1, _param2, _nonce, _chainId));
        require(!seenSignatures[hash], "Signature has been used");

        bytes32 signedHash = hash.toEthSignedMessageHash();
        address signer = signedHash.recover(_sig);
        require(signer == owner, "Invalid signature");

        seenSignatures[hash] = true;

        // 使用 `param1` 和 `param2` 执行授权操作
    }
}
```

值得注意的是，当签名与EIP712类型化数据有效载荷一起使用时，域分隔符值已经包含了链ID。

## 抢先交易Frontrunning

抢先交易攻击是另一种常见问题。攻击者可以监控内存池中的交易，特别是那些使用ECDSA签名的交易系统，比如第三方通过执行有效载荷而获取奖励的系统。根据签名有效载荷中的信息，攻击者可以通过抢跑原始交易，操作特定参数并利用系统.

对于上面的示例，如果有效签名哈希的计算如下:

```solidity
bytes32 hash = keccak256(abi.encodePacked(_param2, _nonce, _chainId));
```

由于缺少`_param1`参数，抢跑攻击者可以随意设置`_param1`的值，进而潜在地利用系统。因此，必须将所有参与签名出发的业务逻辑执行的参数都包含在签名中.

## 签名可塑性

签名可塑性时数字签名的一种特性。在以太坊中，ECDSA签名由两个32字节大小的`r`和`s`值以及一个1字节的恢复值`v`组成。椭圆曲线的对称结构意味着没有签名是唯一的.这些“可塑性”签名的结果是，他们可以在不失效的情况下被更改。对于魅族用于生成签名的参数{r,s,v},都会有另一组不同的参数{r',s',v'}生成一个等效签名。因此，当智能合约系统直接使用`ecrecover`而不是像OpenZeppelin的ECDSA这样的知名库时，检测和丢弃可塑性签名至关重要.

OpenZeppelin的ECDSA库包含以下代码来防止伪造签名:

```solidity
if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
    return (address(0), RecoverError.InvalidSignatureS);
}
```

这一措施可以阻止签名可塑性攻击，因为目前大多数库生成的签名`s`值都在较低的一半范围内。对于签名验证库来说，确保这一检查时必须的.

## EIP-2098 紧凑签名

ECDSA的`recover` 和 `tryRecover`方法容易收到特定形式的签名可塑性攻击，因为他们能够同时处理EIP-2098紧凑签名和传统的65字节签名格式。然而，这个问题仅对接受单字节参数的函数相关，不影响那些接受{r,v,s}或{r,vs}的函数

可能收到影响的合约是哪些通过标记签名本身为"已使用"而不是标记签名消息来实现签名重用活重放保护策略的合约。在这种情况下，用户可能会重新提交已经提交的签名，换成不同的格式(如紧凑签名),并规避已建立的保护机制.

此问题仅影响OpenZeppelin 4.7.3之前的合约。相关的安全公告可以在此处找到.
