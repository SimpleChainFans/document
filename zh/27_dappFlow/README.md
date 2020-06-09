---
id: docs_31
title: 合约编译器
sidebar_label: 合约编译器
---
Dapp开发案例是基于Simplechain开发一个公益众筹项目，下面是整个众筹项目开发的流程梳理。

### 在remix中写合约并测试

下载安装`remix`，并尝试写合约方法。并进行测试。

###  创建React的空工程

创间React的空工程主要包含两条命令，首先要安装`create-react-app`这条命令：

    nstall-过create-react-app

它会安装你的node所需要依赖的目录，然后创建项目:

    ./create-react-app

创建好项目以后，可以使用一下的命令对项目进行初始化的清理工作：

    npm run start

###  创建comple.js文件

创建compile.js文件，然后安装solc编译器，命令如下：

    npm install solc

安装完了solc以后，就可以调用solc.compile(sourceCode,1)对源码进行编译。编译完成后导出编译过的bytecode(字节码)，interface.

### 创建js文件

搭建界面，调用web3,以及与区块链进行数据交互都需要js方法去实现，所以这一步需要创建多个js文件。具体需要创建的js文件如下图：

![31.1.png](https://i.loli.net/2020/06/09/9KU2YTZQ5PDS1GB.png)

### 发起合约功能

发起合约功能主要有两个具体逻辑需要实现，一个是interaction中写具体方法，另一个是在CreateFundingTab中完成调用。

### 发起参与众筹功能

发起参与众筹功能的实现具体主要有三个。一个如下：

1. 在主界面传递一个回调函数onItemClick给CardList，用于返回用户点击的合约的详情
2. 在CardList中传给MyCard
3. 在MyCard中，当触发onClick的时候，调用这个onItemClick，返回相应的detail

通过上面这个三步方法可以得到合约的地址以及得到支持的金额。

第二个是在interaction中写具体方法，最后一个是在CreatorFundingTab中完成调用。图如下：

![31.2.png](https://i.loli.net/2020/06/09/J7ZKT6YdCp15aVO.png)





