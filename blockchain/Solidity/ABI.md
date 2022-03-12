# 合约ABI规范

## 函数选择器

合约调用交易data的前四个字节制定了要调用的具体函数，其通过计算keccak-256(function_name(parameter_type_list))，取前四个字节得到

```js
function increaseAge(string name, uint num) returns(uint){
    return ++age;
}

keccak256("increaseAge(string,uint256)")
```

## 参数编码



