# Solidity初识

## 版本

Solidity通过MAJOR.MINOR.PATCH来制定版本，MAJOR(主版本号)用于不向后兼容的改变，MINOR(次版本号)代表向后兼容的改变，PATCH(补丁号)用于修复BUG向后兼容的版本

实际Solidity版本不同，例如0.4.24，0表示任何东西都可能随时更改，4为主版本号，24为次版本号

## 一个简单的Solidity程序

```python
contract Faucet{
    function withdraw(uint withdraw_amount) public{
        require(withdraw_amount <= 10000000000000000);
        payable(msg.sender).transfer(withdraw_amount);
    }
	fallback() external payable{

	}
}
```

### 编译程序

```shell
solc --optimize --bin Faucet.sol
```

### 应用程序二进制接口(ABI)

ABI指程序模块之间的接口，通常一个在OS层，另一个在用户层，ABI定义了数据结构如何在机器指令中被访问

```json
[
    {
        "stateMutability": "payable", 
        "type": "fallback"
    }, 
    {
        "inputs": [
            {
                "internalType": "uint256", 
                "name": "withdraw_amount", 
                "type": "uint256"
            }
        ], 
        "name": "withdraw", 
        "outputs": [ ], 
        "stateMutability": "nonpayable", 
        "type": "function"
    }
]
```

钱包软件在调用withdraw函数时，需要通过ABI知道，该调用需要提供一个uint256类型的变量，变量的名称为withdraw_amount

## 存储示例

```python
pragma solidity >=0.4.16 <0.9.0;

contract SimpleStorage {
    uint storedData;

    function set(uint x) public {
        storedData = x;
    }

    function get() public view returns (uint) {
        return storedData;
    }
}
```

pragma:该合约是使用solidity 0.4.16甚至更高版本编写，但是版本号得小于0.9

uint：默认为uint256

set、get：这两函数可以用于修改和获得变量storedData值，在函数中无需使用this.便可访问成员变量

## 子货币示例

```javascript
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract Coin {
    // The keyword "public" makes variables
    // accessible from other contracts
    address public minter;
    mapping (address => uint) public balances;

    // Events allow clients to react to specific
    // contract changes you declare
    event Sent(address from, address to, uint amount);

    // Constructor code is only run when the contract
    // is created
    constructor() {
        minter = msg.sender;
    }

    // Sends an amount of newly created coins to an address
    // Can only be called by the contract creator
    function mint(address receiver, uint amount) public {
        require(msg.sender == minter);
        balances[receiver] += amount;
    }

    // Errors allow you to provide information about
    // why an operation failed. They are returned
    // to the caller of the function.
    error InsufficientBalance(uint requested, uint available);

    // Sends an amount of existing coins
    // from any caller to an address
    function send(address receiver, uint amount) public {
        if (amount > balances[msg.sender])
            revert InsufficientBalance({
                requested: amount,
                available: balances[msg.sender]
            });

        balances[msg.sender] -= amount;
        balances[receiver] += amount;
        emit Sent(msg.sender, receiver, amount);
    }
}
```

<font color='red'>address public minter</font>: 声明了一个地址变量，<font color='red'>address</font>是一个160bit的值

<font color='red'>public</font>: 编译器将自动生成一个可以从合约外部访问当前变量的一个函数，如下所示

```javascript
function minter() external view returns(address){return minter;}
```

<font color='red'>mapping (address=>uint) public balances</font>：声明了一个哈希表，并被初始化将每一个存在的key都映射到value全为0，该哈希表无法被遍历，编译器为该变量自动生成的函数如下

```js
function balances(address _account) external view returns(uint){
    return balances[_account];
}
```

<font color='red'>event Sent(address from, address to, uint amount)</font>：声明了一个事件，ethereum客户端可以监听这些事件，当这些事件发生后，监听者可以收到参数from，to以及amount

<font color='red'>constructor</font>：在创建合约时被执行之后不能在被调用，该函数将合约创建者的地址赋值给minter

<font color='red'>require(msg.sender == minter)</font>：确保只有合约创建者才能调用该函数，如果不是则会丢弃以及撤销所有的状态变化

<font color='red'>error</font>：配合<font color='red'>revert</font>使用，与require类似，但是可以提供更多的信息

## 存储

EVM有四个存储区域：storage、memory、calldata、stack

### storage

指永久存储在区块链中的变量所存储的位置，其是一个map，将256bit的字映射到256bit的字，合约的状态变量会存储到该区域，地址从0开始连续布局

### memory

指函数执行期间数据存放的临时位置，其是一个字节数组，每个数组成员大小为256bit，又分为calldata和stack

### calldata

外部函数的参数所存放的位置，也可用于其它变量

### stack

用来保存函数的局部变量，每个栈元素占256bit，最大能存储16个变量

