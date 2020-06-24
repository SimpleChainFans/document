访问浏览器: [浏览器地址](https://explorer.simplechain.com/)

> 打开区块链浏览器，根据自己的需要切换 Main NetWork(主网) 或者 Test NetWork(测试网)。如下图：

![1.png](1.png)

**区块搜索**

> 有关所有块的信息 - 从创世块到所有当前块 - 可以在此页面上找到，包括块高度，其前一个块以及相应的字节大小。 

![2.png](2.png)

**交易搜索**

> 您可以在此页面上搜索交易记录。 可以找到有发送交易地址和接收地址的信息，传输的SIPC数量，交易记录的块高度，相应的哈希和生产时间。 您还可以使用搜索栏查找哈希的特定交易。

![3.png](3.png)

**查看叔块**

> 再此页面上，可以查看每一个区块高度，以及其叔块对应的区块高度，出块时间，以及打包矿工和矿工获得奖励。如下图：

![4.png](4.png)

**验证合约**

> 如果想验证合约，输入要验证的合约地址，选择合约编译器类型和编译器版本，就可以验证合约。如下图：

![5.png](5.png)

**验证后的合约列表**

> 点击验证合约列表，可以查看到20条验证的合约信息。信息展示着合约地址，合约名称，编译器，版本，账户余额，验证时间等！

![6.png](6.png)

合约验证一共有三种验证方式：

1. 单文件文件验证方式
2. 多文件验证方式
3. Json文件验证方式

## 单文件验证

```go
pragma solidity ^0.5.12;
contract PayALL{
    event PayLog(address name,uint reward);
    event PayLog2(address name);
    constructor() public payable{
    }
    function() external payable{
        emit PayLog(msg.sender, msg.value);
    }
    function setMethd() public payable {
        emit PayLog2(msg.sender);
    }
}
```
![29.1.png](7.png)

粘贴源码：

![29.2.png](8.jpg)

验证通过后；
 
![29.3.png](9.png)

## 多文件验证

因在合约书写过程中，文件的依赖可以引用另一个文件。

```go
	// test1.sol文件
pragma solidity ^0.5.17;
contract Hello {
    uint value;
    
    function hello() public pure returns(string memory){
        return "hello world_1";
    }
    function set(uint x) public {
        value = x;
    }
}
```

```go
// test2.sol文件
pragma solidity ^0.5.17;
import  {Hello} from "./test1.sol" ;
contract Test2 is Hello {
    function test() public pure returns(string memory){
        return  Hello.hello();
    }
}
```
![29.4.png](10.png)
 
执行结果：
	
![29.5.png](11.png)

此类验证使用，多文件验证。验证方式如下，获取已部署的合约地址：

`0x6e1ace6e6cf09a4ab096f272cfc029c0a1d883ac`
 
![29.6.png](12.png)

![29.7.png](13.png)
 
`Optimization`指的是合约编译的次数，如何合约的部署是选择了优化次数，那么在验证时，也需要选择相应的次数，默认不优化。

`Constructor Arguments ABI-encoded`表示合约是否填有构造参数，构造参数是部署时产生，验证这里不需要用户填写。

`Contract Library Address`涉及library合约，普通合约不做填写。后续会增加此类验证的详解。

**查看验证成功**

![29.8.png](14.png) 

当前验证后的多文件合约源码

![29.9.png](15.png)

源码通过json 格式展示，分别包含两个文件源码。

**Json 文件验证：**

Json 文件验证方式较为复制，需要用户了解solidity的编译参数，solidity会通过用户传递的json文件进行编译，但前提是这份json文件必须满足编译的规范。

Json文件：
```json
{
  "language": "Solidity",
  "sources": {
    "myFile.sol": {
      "content": "pragma solidity ^0.5.12;contract Multiply7 {  event Print(uint); event CjLog(address, uint);   uint public a ;   uint public b ;   constructor (uint _a, uint _b) public {        a = _a;       b = _b;   }  function multiply(uint input1) public view returns (uint) {     return input1 * 6 + a + b;  }   function multiplyplus(uint input1, uint input2) public returns (uint) {   emit Print(input1 * 6 * input2);  emit CjLog(msg.sender, a+b);    return input1 * 6 * input2 + a + b;   }}"
    }
  },
  "settings": {
    "metadata": {
      "useLiteralContent": true
    },
    "outputSelection": {
      "*": {
        "*": [
          "*"
        ],
        "": [
          "ast"
        ]
      }
    }
  }
}
```
![29.10.png](16.png)

![29.11.png](17.png)

## Library 的合约验证

[首先了解一下什么是library合约](https://blog.csdn.net/weixin_43343144/article/details/88251579)

浏览器是支持`library`库的合约验证的，`Library` 合约在部署过程中，会连续并且嵌套依次部署合约，所有会部署多条合约（合约内部包括library库，会被一同部署)并产生多条hash。

**合约源码:**

```go
/**
 *Submitted for verification at Etherscan.io on 2020-02-27
*/

pragma solidity ^0.6.0;

library SetHFG {
  // 我们定义了一个新的结构体数据类型，用于在调用合约中保存数据。

  struct Data { mapping(uint => bool) flags; }

  // 注意第一个参数是“storage reference”类型，因此在调用中参数传递的只是它的存储地址而不是内容。
  // 这是库函数的一个特性。如果该函数可以被视为对象的方法，则习惯称第一个参数为 `self` 。
  function insert(Data storage self, uint value)
      public
      returns (bool)
  {
      if (self.flags[value])
          return false; // 已经存在
      self.flags[value] = true;
      Acc.hi();
      
      return true;
  }

```
如下是部署记录：

![29.12.png](18.png)

CHU 为主合约: `0x1c622ed2035acc8514259788caafb3cf69d9bbe3`

**Library合约地址分别为：**
Bcc：`0x0da8bd0dd96f78ac6e155b2d5e9fb6d15ca89fad`

Fcc：`0x7b05897e605a3878d3511069924d750ef3954da5`

Acc：`0x13a6cb372333394caa8ea0e5876a4a52167a95ce`

SetHFG: `0x5386710eb4d02b41e7a253d3ebb00ca8795c930f`

验证方式与单文件相同, 如下图：

![29.13.png](19.png)

![29.14.png](20.png)

**验证通过**

![29.15.png](21.png)

![29.16.png](22.png)


