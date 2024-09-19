# Solidity

## 关键字 Public, Private, Internal, External

在Solidity中, `public`,`private`,`internal`和`external`四个不同的访问修饰符,用于限定函数或状态变量的可见性和可访问性.

1. **public**:

`public`修饰符表示函数或状态变量可以从合约内部和外部进行访问,意味着其他合约和外部地址都可以调用公共函数或读取公共状态变量.

```solidity
uint public value;

function name() public {}
```

2. **private**:

`private`修饰符表示函数或状态变量仅能在当前合约内部访问。私有函数和状态变量对于外部地址是不可见的,只能在合约内的其他函数中使用.

```solidity

```

3. **internal**:

`internal`修饰符标识函数或状态变量可以在当前合约内部以及继承合约中进行访问. 与私有修饰符不同, `internal`修饰符允许继承的合约访问被修饰的函数或状态变量.

4. **external**:

`external`修饰符标识函数只能被其他合约调用，不能在当前合约内部直接调用，不能用于修饰变量, 只能修饰函数。外部函数通常用于提供合约的接口，可以从其他合约或外部交易调用。
