# 状态变量在Storage中的布局

状态变量以紧凑的方式存储在storage中，因此一个storage槽可以存放多个变量值。除了动态数组和映射，其它变量都是从0开始连续地存储，存储规则如下：

* storage槽中第一个元素是以右对齐方式存储的
* 值类型使用其所需的大小存储
* 如果一个值类型大小大于槽中剩余的大小，则将该值类型存储到下一个槽
* 结构体和数组会占用新槽，但其中的元素会使用这些规则进行存储
* 在结构体或数组之后的元素会占用新槽

对于使用了继承的合约，状态变量的顺序是以基合约开始的C3线性化顺序决定的，不同合约的状态变量可以共享一个slot

## 映射和动态数组

动态数组和映射大小使用如上规则占用一个槽p，对于动态数组，p存储了数组元素的个数，对于映射，该槽为空

动态数组的数据从keccak256(p)开始存储，如果元素没有超过16字节则共享一个槽。对于二位数组元素x\[i\]\[j\]，假设x本身存储在p槽上，则其存储位置计算如下：

```js
keccak256(keccak256(p) + i) + floor(j / floor(256 / 24))
```

映射的不会存储key只会存储value，其存储位置计算如下：

```js
keccah256(h(k).p) //.为连接,h取决于k的类型
```

其中\.为连接符号，h取决于k的类型：

* 对于值类型，h是将数据填充
* 对于strings和字节数组，h是不填充数据

```js
contract C {
    struct S { uint16 a; uint16 b; uint256 c; }
    uint x;
    mapping(uint => mapping(uint => S)) data;
}
data[4][9].c = keccak256(uint256(9) . keccak256(uint256(4) . uint256(1))) + 1
```

x为uint256，单独占一个槽(槽0)，data存放在槽1中，data\[4\]存放在keccak256(uint(4).uint256(1))，data\[4\]是一个mapping，因此data\[4\]\[9\]存放在keccak256(uint256(9) . keccak256(uint(4).uint256(1)))，由于a，b共用一个槽，c单独占一个操，因此data\[4\]\[9\]位置为data\[4\]\[9\].c = keccak256(uint256(9) . keccak256(uint256(4) . uint256(1))) + 1

### bytes和string

bytes和string编码完全一样，通常会有一个槽存储该数组，其它槽存储数据，但是对于短数据(小于32字节)，槽的前31个字节存储数据，最后一个字节存储len * 2，对于非端数据，主槽p存储len * 2 + 1，数据存储的槽通过keccak256(p)计算得到

## JSON输出

合约的storage布局可以通过JSON显示，如下显示了fileA文件中contract A{uint x;}的storage布局

```json
{
    "astId": 2,
    "contract": "fileA:A",
    "label": "x",
    "offset": 0,
    "slot": "0",
    "type": "t_uint256"
}
```

* astId：为状态变量声明的AST节点的id
* contract：合约名称以及文件名
* label：状态变量名称
* offset：storage中bytes偏移量
* slot：状态变量所在槽
* type：变量类型信息的标识，t_uint256的形式如下

```json
{
    "encoding": "inplace",
    "label": "uint256",
    "numberOfBytes": "32",
}
```

* encoding：数据的编码
  * inplace：数据连续排序
  * mapping：keccak-256
  * dynamic_array：keccak-256
  * bytes：当个槽或者keccak-256
* label：类型名
* numberOfBytes：所使用字节数量

## 参考

[Layout of State Variables in Storage](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html)

[详解SOLIDITY合约数据存储布局](https://learnblockchain.cn/books/geth/part7/storage.html)





