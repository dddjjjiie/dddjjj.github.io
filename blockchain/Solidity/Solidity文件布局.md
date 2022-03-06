# Solidity文件布局

## SPDX License Identifier

如果代码开源可见添加如下声明

```js
// SPDX-License-Identifier: MIT
```

如果代码不开源可以添加如下声明

```js
// SPDX-License-Identifier: UNLICENSED
```

编译器不会验证该许可，但该声明会被包括到字节码元数据中

## Pragmas

启用编译器功能或检查。当使用import导入另一文件时，另一文件的pragmas不会被自动导入

### 版本号

```js
pragma solidity ^0.5.2;
```

指示不使用0.5.2版本之前的编译器并且对于0.6.0以及之上的版本都不支持

## 导入其它文件

```js
import "filename"
```

<font color='red'>filename</font>是文件路径，该语句会将<font color='red'>filename</font>中的所有全局符号导入到当前的领域中

```js
import * as symbolName from "filename"
```

该语句创建了全局符号<font color='red'>symbolName</font>，其包含了<font color='red'>filename</font>中的所有全局符号，通过<font color='red'>symbolName.symbol</font>可以使用<font color='red'>filename</font>中的全局符号

```js
import "filename" as symbolName
```

等同于<font color='red'>import * as symbolName from "filename"</font>

```js
import {symbol1 as alias, symbol2} from "filename"
```

将<font color='red'>filename</font>中的<font color='red'>symbol1</font>重命名为<font color='red'>alias</font>导入到当前文件

## 注释

可以使用单行注释(<font color='red'>//</font>)和多行注释(<font color='red'>/**/</font>)

```js
// single-list comment

/*
multi-line comment
*/
```

## 参考

[Layout of a Solidity Source File](https://docs.soliditylang.org/en/develop/layout-of-source-files.html)

