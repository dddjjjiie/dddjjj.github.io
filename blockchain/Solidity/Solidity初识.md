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

```shell
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