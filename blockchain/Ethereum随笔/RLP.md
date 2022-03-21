# RLP

RLP函数接收一个元素，该元素定义如下：

* 一个字符串(字节数组)是一个元素
* 元素的列表也是一个元素

例如空字符串是一个元素，单词cat也是一个元素， ["cat",["puppy","cow"],"horse",[[]],"pig",[""],"sheep"] 也是一个元素

## 编码规则

* 对于一个值在[0x00, 0x7f]范围的单字节，该字节就是RLP的编码
* 对于一个长度在[0, 55]字节内的字符串，需将使用0x80加上字符串长度，第一个字节范围在[0x80, 9xb7]
* 对于长度大于55字节的字符串，需要使用0xb7加上字符串长度的二进制长度，接着是字符串长度，接着是字符串编码，例如1024长度的字符串，其前两个字节为0xb9 0x04，第一个字节范围在[0xb8, 0xbf]
* 如果列表长度在[0, 55]字节内，需要使用0xc0加上列表的编码长度，第一个字节范围在[0xc0, 0xf7]
* 如果列表长度大于55字节，需要使用0xf7加上列表长度的二进制长度，接着是列表编码的长度，接着是列表的编码，第一个字节范围在[0xf7, 0xff]

## 实例

dog = [0x83, d, o, g]

[cat, dog] = [0xc8, 0x83, c, a, t, 0x83, d, o, g]

'' = [0x80]

[] = [0xc0]

0 = [0]

1024(0x40 0x00) = [0x82, 0x40, 0x00]

Lorem ipsum dolor sit amet, consectetur adipisicing elit = [0xb8, 0x38, L, o, ... , t]

 ["The length of this sentence is more than 55 bytes, ", "I know it because I pre-designed it"]  = [0xf8, 0x58, 0xB3, T, ..., ' ', 0xA3, I, ...,t]

## 参考

[RLP](https://eth.wiki/en/fundamentals/rlp)

[以太坊源码学习—RLP编码](https://segmentfault.com/a/1190000011763339)