# 阻塞攻击

阻塞攻击是一种针对智能合约漏洞的攻击，通常与业务逻辑相关。这类攻击虽然不会直接为攻击者带来利润，但会对智能合约系统的运行产生负面影响。在某些情况下，阻塞攻击可能被用于破坏整个系统的运作或在某些关键时刻制造混乱.

以下Solidity代码展示了一个典型的阻塞攻击场景:

```solidity
pragma solidity ^0.8.17;

contract DelayedWithdrawal {
    address beneficiary; // 受益人地址
    uint256 delay; // 延迟时间
    uint256 lastDeposit; // 最近一次存款的时间戳

    constructor(uint256 _delay) {
        beneficiary = msg.sender; // 将受益人设置为合约部署者
        lastDeposit = block.timestamp; // 初始化最近存储时间为当前区块时间
        delay = _delay; // 设置延迟时间
    }

    modifier checkDelay() {
        // 用于检查是否已经过了延迟时间
        require(block.timestamp >= lastDeposit + delay, "Keep waiting");
        _;
    }

    function deposit() public payable {
        // 存款函数, 任何人都可以向合约存款
        require(msg.value != 0); // 确保存款金额不为0
        lastDeposit = block.timestamp; // 更新最后存款时间为当前时间
    }

    function withdraw() public checkDelay {
        // 提款函数, 受益人可以在延迟时间后提取所有资金
        (bool success, ) = beneficiary.call{value: address(this).balance}(""); // 将合约中所有余额转给受益人
        require(success, "Transfer failed"); // 确保转账成功
    }
}
```

这个例子展示了一个`DelayedWithdrawal` 合约。在构造函数中，受益人被设置为合约部署者的地址，并设定了一个自定义延迟(例如，24小时),任何人都可以向合约存款，这些资金将在配置的延迟时间后可供受益人提取并转移。然而，每次存款必须向智能合约转移非零金额的ETH.

**阻塞攻击**的问题在于任何人都可以调用`deposit`函数，并重置`lastDeposit`时间戳，这使得攻击者可以通过向智能合约转少量金额(例如1wei),触发`lastDeposit`时间戳的重置，从而阻止受益人提取他们的资金.

攻击者可以在延迟期结束前提交一笔交易来重置时间戳，或者通过抢跑交易来预先阻止受益人调用`withdraw`函数，从而制造一个更加低成本的拒绝服务攻击(Dos).

## 燃料不足的阻塞攻击

燃料不足的阻塞攻击时一种阻塞攻击的子类型，主要影响在外部调用时没有检查返回值成功与否的智能合约。在这种攻击中，攻击者可能只提供足够的燃料保证顶层函数的成功执行，但不足以让外部调用完成，导致燃料耗尽。顶层寒月可以完成其函数调用，导致状态更改不完整，这种问题尤其常见于执行通用调用的智能合约中，如中继器和多重签名钱包.

以下是一个展示燃料不足阻塞攻击潜在风险的简化中继器合约:

```solidity
pragma solidity ^0.8.17;

contract Relayer {
    mapping (bytes => bool) executed;
    address target;

    function forward(bytes memory _data) public {
        require(!executed[_data], "Replay protection");
        // 中间可能有更多的签名验证代码
        executed[_data] = true;
        target.call(abi.encodeWithSignature("execute(bytes)", _data));
    }
}
```

注意,在`forward`函数中的外部调用失败时，合约可以选择回滚整个交易或继续执行。提供的示例合约并未检查外部调用的成功与否，而是直接继续执行。因此，一旦`forward`函数执行完毕，提交的数据将被标记为已执行，这样任何人都不能再次提交相同的数据.

任何第三方转发者都可以调用`forward`函数，带起用户执行其交易。如果转发者以最少的燃料调用`forward`函数，只提供足够的燃料让中继器合约成功执行，但导致外部调用因燃料不足而回滚，此时用户的交易并未执行，他们的签名失效，结果，目标合约上期望的状态更改不会发生.
