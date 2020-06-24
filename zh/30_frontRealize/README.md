## 初始化React项目

前端是基于React框架开发的，所以我们首先进行项目初始化操作。

### 初始化项目

```bash
create-react-app funding-eth-react
```
### 精简项目

通lottery项目，src下仅留App.js和index.js，调整对应代码。

```javascript
import React, { Component } from 'react';
class App extends Component {
  render() {
    return (
        <div>Hello World</div>
    ); 
  }
}
export default App;
```
执行如下命令：

npm start

![1.png](1.png)

### 安装依赖库

```bash
npm install --save web3
npm install --save semantic-ui-react
npm install --save semantic-ui-css
```

## 引用web3.js

### 实现

在src下创建名为utils的文件夹，并在内部创建文件getWeb3.js

```javascript
import Web3 from 'web3';
let web3;
if (typeof  window.web3 !== 'undefined') {
    console.log('found injected web3');
    web3 = new Web3(window.web3.currentProvider);
} else {
    console.log('found local web3');
    web3 = new Web3('http://localhost:7545');
}
export default web3;
```
## 调用众筹合约

### 部署Funding至Simplechain测试网络

获取合约地址：

    0x8ff3a13157f1a0aa99beb84d393f1aac4dd470e3 

### 调用Funding合约ABI

![2.png](2.png)

### 获取合约实例

首先在src下创建目录sipc,并创建文件contracts.js。

ABI格式为json格式，可以把ABI拷贝到浏览器的地址栏中格式化为一行再复制回来，这样省空间

```javascript
import web3 from '../utils/getWeb3';
//将ABI添加到这里
const fundingFactoryABI = [ { "constant": true, "inputs": [], "name": "platformProv ider", "outputs": [ { "name": "", "type": "address" } ], "payable": false, "stateMu tability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "getInvestorFunding", "outputs": [ { "name": "", "type": "address[]" } ], "payable" : false, "stateMutability": "view", "type": "function" }, { "constant": true, "inpu ts": [], "name": "getCreatorFunding", "outputs": [ { "name": "", "type": "address[]"
} ], "payable": false, "stateMutability": "view", "type": "function" }, { "constan t": true, "inputs": [], "name": "getAllFunding", "outputs": [ { "name": "", "type": "address[]" } ], "payable": false, "stateMutability": "view", "type": "function" },
{ "constant": true, "inputs": [ { "name": "", "type": "uint256" } ], "name": "crow FundingArray", "outputs": [ { "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "", "type": "address" }, { "name": "", "type": "uint256" } ], "name": "crea torFundingMap", "outputs": [ { "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": false, "inputs": [ { "name": "_projectName", "type": "string" }, { "name": "_supportMoney", "type": "uin t256" }, { "name": "_goalMoney", "type": "uint256" }, { "name": "_duration", "type" : "uint256" } ], "name": "createFunding", "outputs": [], "payable": false, "stateMu tability": "nonpayable", "type": "function" }, { "inputs": [], "payable": false, "s tateMutability": "nonpayable", "type": "constructor" } ];

const fundingFactoryAddress = '0x8ff3a13157f1a0aa99beb84d393f1aac4dd470e3';
//创建fundingFactory合约实例
let fundingFactoryContract = new web3.eth.Contract(fundingFactoryABI, fundingFactor yAddress);
let contracts  = {
    fundingFactoryContract,
}
export default contracts;
```
整个前端的开发还有很多模块，下面的界面开发步骤图我们可以看到整个界面包括我发起的众筹，所有的众筹，以及我参与的众筹等模块。如下图：

![3.png](3.png)

上面的步骤已经实现了整个前端页面的基础框架，以及合约调用案例。其它模块的开发和上面的步骤类似。就不再做详细介绍，可以直接查看源码学习。





