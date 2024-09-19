# 燃料不足的阻塞攻击

燃料不足的阻塞攻击可以在接受数据并在另一个合约上进行子调用的合约中实施. 这种方法常用于多重签名钱包以及交易中继器。如果子调用失败，要么整个交易被回滚，要么继续执行.

我们以一个简单的中继器合约为例。如下所示，中继器合约允许某人创建并签署一笔交易,而无需自己执行该交易. 这通常用于用户无法支付交易所需的燃料费用的场景.

```solidity
contract Relayer {
    mapping (bytes => bool) executed;

    function relay(bytes _data) public {
        // replay protection; do not call the same transaction twice
        require(executed[_data] == 0, "Duplicate call");
        executed[_data] = true;
        innerContract.call(bytes4(keccak256("execute(bytes)")), _data);
    }
}
```

执行交易的用户，即“转发者”,可以通过使用刚好足够的燃料来有限的审查交易，使得交易得以执行，但不足以让子调用成功完成.

有两种方法可以防止这种情况发生. 第一种解决方案是只允许受信任的用户中继交易。另一种解决方案是要求转发者提供足够的燃料, 如下所示.

```solidity
// contract called by Relayer
contract Executor {
    function execute(bytes _data, uint _gasLimit) {
        require(gasleft() >= _gasLimit);
        ...
    }
}
```

### 来源

- [SCSFG - 阻塞攻击](../misc/griefing/griefing_attacks.md)
- [Ethereum Stack Exchange - What does griefing mean?](https://ethereum.stackexchange.com/questions/62829/what-does-griefing-mean)
- [Ethereum Stack Exchange - Griefing attacks: Are they profitable for the attacker?](https://ethereum.stackexchange.com/questions/73261/griefing-attacks-are-they-profitable-for-the-attacker)
- [Wikipedia - Griefer](https://en.wikipedia.org/wiki/Griefer)
