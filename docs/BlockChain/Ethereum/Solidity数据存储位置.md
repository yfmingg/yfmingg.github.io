# Solidity数据存储位置

EVM 把数据保存在 存储（Storage） 和 内存（Memory） 中。存储（Storage）用于永久存储数据，而 内存（Memory）仅在函数调用期间保存数据。还有一个地方保存了函数参数，叫做 调用数据（calldata），这种存储方式有点像内存，不同的是不可以修改这类数据。

## 存储（Storage）

你可以在智能合约上永久地存储数据，以便将来执行时可以访问它。每个智能合约都在自己的永久存储中保持其状态。它就像*"智能合约的迷你数据库 "*，但与其他数据库不同，这个数据库是可以公开访问的。所有存储在智能合约存储器中的值可供外部免费读取（通过静态调用），无需向区块链发送交易。然而，向存储空间写入是相当昂贵的。事实上，就Gas成本而言，它是EVM中最昂贵的操作。存储的内容可以通过sendTransaction调用来改变。这种调用会改变状态。这就是为什么合约变量被称为状态变量的原因。需要记住的一件事是，在以太坊和EVM的设计中，一个合约既不能读也不能写非自身定义的任何存储。合约A可以从另一个合约B的存储中读取或写入的唯一方法是当合约B暴露出使其能够这样做的函数。

智能合约的存储是一个持久的可读可写的数据位置。意思是说，如果数据在一次交易中被写入合约存储，一旦交易完成，它就会持久存在。在这个交易之后，读取合约存储将检索到之前这个交易所写入/更新的数据。每个合约都有自己的存储，可以用以下规则来描述和绑定：

- 持有状态变量
- 在交易和函数调用之间持久存在
- 读取是免费的，但写入是昂贵的
- 合约存储在合约构建期间被预先分配。

驻留在存储中的变量在 Solidity 中被称为状态变量。你应该记住关于合约存储的唯一事情是：存储是持久保存和昂贵的!

成本并不总是相同的，计算写入存储的Gas是相当复杂的公式，尤其是在最新的以太坊2.0升级后）。作为一个简单的总结，写入存储的成本如下：

- 初始化一个存储槽（第一次，或如果该槽不包含任何值），从零到非零值，花费20,000 gas
- 修改一个存储槽的值需要5,000个Gas
- 删除存储槽中的数值，需要退还15,000 Gas。

默认情况下，一个智能只能在执行环境中读取自己的存储（通过SLOAD）。但是，如果一个智能合约在其公共接口（ABI）中公开了能够从特定的状态变量或存储槽中读取数据的函数，那么该智能合约也可以读取其他智能合约的存储。

### 存储的布局

智能合约的存储是一个键值映射（=数据库），其中键对应于存储中的一个槽号，而值是存储在这个存储槽中的实际值。智能合约的存储是由槽组成的，其中：

- 每个存储槽可以包含长度不超过32字节的字。
- 存储槽从位置0开始（就像数组索引）。
- 总共有2²⁵⁶个存储槽可用（用于读/写）。

综上所述：一个智能合约的存储由2²⁵⁶个槽组成，其中每个槽可以包含大小不超过32字节的值。

在底层，合约存储是一个键值存储，其中256位的键映射到256位的值。每个存储槽的所有值最初都被设置为零，但也可以在合约部署期间（即 "构造函数"）初始化为非零或一些特定的值。

可以将智能合约的存储描述为 "一个天文数字的大数组，最初充满了零，数组中的条目（索引）就是合约的存储槽。" 。

智能合约的存储是在合约构建过程中（在合约被部署时）预置的。这意味着合约存储的布局在合约创建时就已经确定了。该布局是基于你的合约级变量声明而 "成型 "的，并且这种布局不能被未来的方法调用所改变。

Solidity为你合约中的每一个定义的状态变量分配了一个存储槽。对于静态大小的状态变量，存储槽是连续分配的，从0号槽开始，按照定义状态变量的顺序。请看下面示例：

```solidity
contract Owner {
    
    address _owner;
    
    function owner() public returns (address) {
        return _owner;
    }
}
```

让我们用solc命令行工具看看上一个合约的实际存储布局，如果你运行下面的命令:

solc contracts/Owner.sol --storage-layout --pretty-json

你将得到以下JSON输出：

```json
======= contracts/Owner.sol:Owner =======
Contract Storage Layout:
{
  "storage":
  [
    {
      "astId": 3,
      "contract": "contracts/Owner.sol:Owner",
      "label": "_owner",
      "offset": 0,
      "slot": "0",
      "type": "t_address"
    }
  ],
  "types":
  {
    "t_address":
    {
      "encoding": "inplace",
      "label": "address",
      "numberOfBytes": "20"
    }
  }
}
```

从上面的JSON输出中，我们可以看到一个storage字段，它包含一个对象数组。这个数组中的每个对象都是指一个状态变量名。我们还可以看到，每个变量都被映射到一个 插槽（slot），并有一个基本的 类型（type）。

这意味着变量_owner可以被改变为同一类型（在我们的例子中为地址）的任何有效值。然而，槽0是为这个变量保留的，并将永远在那里。

多个状态变量的时候，所有静态大小的变量都是按照它们被定义的顺序依次放入存储槽的。

```solidity
pragma solidity ^0.8.0;

contract StorageContract {
    
    uint256 a = 10;
    uint256 b = 20;
}
```

记住：每个存储槽最多可以容纳32字节长的值。

在我们上面的例子中，a和b是32字节长（因为它们的类型是uin256）。因此，它们被分配了自己的存储槽。

### 将状态变量打包在一个存储槽中

在我们之前的例子中没有什么特别之处。但是现在让我们考虑这样的情况：你有几个不同大小的uint变量，如下所示：

```solidity
pragma solidity ^0.8.0;

contract StorageContract {

    uint256 a = 10; // 16进制为a
    uint64 b = 20;  // 0x14
    uint64 c = 30;  // 0x1e
    uint128 d = 40; // 0x28
    
    // output:  0:bytes32: result 0x000000000000000000000000000000000000000000000000000000000000000a
    function readStorageSlot0() public view returns (bytes32 result) {
     assembly {
            result := sload(0)
        }
    }
    
    // output:  0:bytes32: result 0x00000000000000000000000000000028000000000000001e0000000000000014
    function readStorageSlot1() public view returns (bytes32 result) {
       assembly {
            result := sload(1)
        }
    }
}
```

我们已经写了两个基本的函数来读取低级别的合约存储槽，输出中，状态变量 a 被低阶对齐存储，b、c、d被打包在一个存储槽中。因此，当变量小于32字节时，Solidity尝试将一个以上的变量打包到一个存储槽中，如果它们能被容纳的话。

一个存储槽可以容纳一个以上的状态变量，但如果一个基本类型不适合存储槽的剩余空间，它将被移到下一个存储槽。对于以下Solidity合约：

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

contract StorageContract {

    uint256 a = 10;
    uint64 b = 20;
    uint128 c = 30;
    uint128 d = 40;
    
    // soltNum = 0: bytes32: result 0x000000000000000000000000000000000000000000000000000000000000000a
    // soltNum = 1: bytes32: result 0x00000000000000000000000000000000000000000000001e0000000000000014
    // soltNum = 2: bytes32: result 0x0000000000000000000000000000000000000000000000000000000000000028
    function readStorageSlot(uint soltNum) public view returns (bytes32 result) {
     assembly {
            result := sload(soltNum)
        }
    }
    
}
```

让我们看一个更具体的例子，一个流行的Defi协议: Aave。 AAVE协议使用Pools作为管理流动性的主要智能合约。这些是主要的 "面向用户的合约"。用户直接与 Aave pool合约交互，以提供或借用流动性（通过 Solidity 的其他合约，或使用 web3/ethers 库）。定义在 Pool.sol 中的主要 Pool 合约继承了一个名字很有趣的合约 PoolStorage 。正如协议的Aave v3的Natspec注释中所描述的，PoolStorage合约有一个目的：定义了Pool合约的存储布局 。

### 存储布局与继承性

合约存储的布局也是基于继承的。如果一个合约继承了其他合约，它的存储布局就会遵循继承的顺序。

在最基础的合约中定义的状态变量从0槽开始。
在下面的派生合约中定义的状态变量被放在次序槽中（槽1、2、3，等等......）。
另外，请注意，与将状态变量打包在一个存储槽中的规则同样适用。如果可以通过继承，来自不同父子合约的状态变量确实共享同一个存储槽。

### 与存储交互

EVM提供了两个操作码来与存储进行交互：SLOAD来读取，SSTORE来写入存储。这两个操作码只在内联汇编中可用。Solidity在编译后将写到状态变量转换为这些操作码。

SLOAD从存储中加载一个字到栈中。SLOAD操作码在内联汇编中可用。它可以用来轻松检索存储在特定存储槽的整个字值。

```solidity
// 使用内联汇编读取存储
function readStorageNb(uint256 slotNb) 
    public 
    view 
    returns (bytes32 result) 
{
    assembly {
        result := sload(slotNb)
    }
}

// 使用内联汇编写入存储
function writeToStorageSlot(uint256 slotNb) public {
 string memory value = "All About Solidity";
  assembly {
        sstore(slotNb, value)
    }
}
```

### 函数参数中的存储指针

当storage在一个函数参数中被指定时，这意味着传递给函数的参数必须是一个状态变量。

### 在函数体中的存储指针

当变量为基本类型时，将存储变量赋值给局部变量（在函数体中定义的）总是复制。

然而，对于复杂或动态类型，规则有所不同。如果你不希望被克隆，你可以将关键字storage传递给一个值。我们将这些变量描述为存储指针或存储引用类型的局部变量。

在一个函数中，任何存储引用的变量总是指的是在合约的存储上预先分配的一块数据。换句话说，一个存储引用总是指的是一个状态变量。

### 从汇编和Yul访问存储

你可以通过指定一个存储槽和存储偏移量，在内联汇编中读写合约存储。

我们之前看到，存储中的一些变量不一定占据一个完整的存储槽，但有时会被挤在一起。

我们还看到，SLOAD作为一个操作码只接受存储槽号作为参数，并返回存储在这个槽下的完整的bytes32值。

但是，如何读取一个挤在同一个存储槽中的状态变量？

Solidity 文档解释：对于本地存储变量或状态变量，使用一个Yul标识符是不够的，因为它们不一定占据一个完整的存储槽。

因此，它们的 "地址 "是由一个槽和该槽内的一个字节偏移量组成。以下面的合约为例:

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

contract Storage {
    uint64 a = 1;
    uint64 b = 2;
    uint128 c = 3;

    function getSlotNumbers() public view returns(uint256 slotA, uint256 slotB, uint256 slotC) {
        assembly {
            slotA := a.slot
            slotB := b.slot
            slotC := c.slot
        }
    }
        
    function getVariableOffsets() public view returns(uint256 offsetA, uint256 offsetB, uint256 offsetC) {
        assembly {
            offsetA := a.offset
            offsetB := b.offset
            offsetC := c.offset
        }
    }
}
```

要检索变量c所指向的槽，使用c.slot，要检索字节偏移量，使用c.offset。仅使用c本身会导致错误，有一点也要提到的是，在内联汇编中，你不能向存储变量的.slot或.offset赋值。

Yul中存储指针的偏移量的值是多少呢？在函数体中，一些变量可以是存储指针/存储引用。例如，这包括struct、array和mapping。对于这样的变量，在Yul中.offset总是为零，因为这样的变量总是占据了一个完整的存储槽，不能与其他变量紧密地挤在一起存储。
