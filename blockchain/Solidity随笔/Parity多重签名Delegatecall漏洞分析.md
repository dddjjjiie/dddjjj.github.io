# Paraity多重签名Delegatecall漏洞分析

在合约中一共有4个类分别是：WalletEvents、WalletAbi、WalletLibrary、，漏洞主要是利用了WalletLibrary以及Wallet合约

## WalletEvents

在该合约中主要定义了一些事件

```js
contract WalletEvents {
  event Confirmation(address owner, bytes32 operation);
  event Revoke(address owner, bytes32 operation);
  event OwnerChanged(address oldOwner, address newOwner);
  event OwnerAdded(address newOwner);
  event OwnerRemoved(address oldOwner);
  event RequirementChanged(uint newRequirement);
  event Deposit(address _from, uint value);
  event SingleTransact(address owner, uint value, address to, bytes data, address created);
  event MultiTransact(address owner, bytes32 operation, uint value, address to, bytes data, address created);
  event ConfirmationNeeded(bytes32 operation, address initiator, uint value, address to, bytes data);
}
```

## WalletAbi

在该合约中主要声明了一些函数但未实现

```js
contract WalletAbi {
  function revoke(bytes32 _operation) external;
  function changeOwner(address _from, address _to) external;
  function addOwner(address _owner) external;
  function removeOwner(address _owner) external;
  function changeRequirement(uint _newRequired) external;
  function isOwner(address _addr) constant returns (bool);
  function hasConfirmed(bytes32 _operation, address _owner) external constant returns (bool);
  function setDailyLimit(uint _newLimit) external;
  function execute(address _to, uint _value, bytes _data) external returns (bytes32 o_hash);
  function confirm(bytes32 _h) returns (bool o_success);
}

```

## WalletLibrary

继承自WalletEvents，包含了对多重签名的所有操作，首先是状态变量

```js
  struct PendingState { //
    uint yetNeeded; //还需要签名的人数
    uint ownersDone; //已签名的人,其使用bitmap,有多少个1表示有多少人签名
    uint index; //与该PendingState对应的operate在m_pendingIndex中的索引
  }

  struct Transaction { //要执行的交易
    address to;
    uint value;
    bytes data;
  }

  //该合约的地址
  address constant _walletLibrary = 0xcafecafecafecafecafecafecafecafecafecafe;

  uint public m_required; //M-N签名中的M

  uint public m_numOwners; //M-N签名中的N

  uint public m_dailyLimit; //一天可以花的ether
  uint public m_spentToday; //当天已花的ether
  uint public m_lastDay; //更新日期的最后一天

  uint[256] m_owners; //N个人的地址

  uint constant c_maxOwners = 250; //N最大值为250
  mapping(uint => uint) m_ownerIndex; //N地址到m_owners索引的映射
  mapping(bytes32 => PendingState) m_pending; //operate到PendingState的映射
  bytes32[] m_pendingIndex; //用于存储operate
  mapping (bytes32 => Transaction) m_txs; //operate到transaction的映射
```

下面列出了该合约的一些关键函数

```js
  modifier onlyowner { //只有owner才可以执行
    if (isOwner(msg.sender))
      _;
  }

  modifier onlymanyowners(bytes32 _operation) { //用于创建一个operate或者
    if (confirmAndCheck(_operation))
      _;
  }
```

