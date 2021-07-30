---
id: docs_44
title: 平台对接
sidebar_label: 平台对接
---

## 搭建SimpleChain节点

### docker 搭建

**获取镜像：**

```sh
docker pull simplechain/sipe:latest
```
**开启RPC**

```sh
docker run -it -p 8545:8545 -p 30312:30312 simplechain/sipe --rpc --rpcaddr "0.0.0.0"
```
可以通过以下命令查看自己的节点是否启动成功：

```sh
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":68}'
```

### 源码搭建

`前期准备:Go 语言环境(1.10 或以上版本)、C 语言编译器`

**1.下载 SimpleChain**

可以通过 git 将项目 clone 到本地，也可以在 https://github.com/simplechain-org/go-
simplechain 页面直接下载。

    git clone https://github.com/simplechain-org/go-simplechain.git 

**2.安装 sipe**

1.进入 go-simplechain 根目录。

```javascript
cd go-simplechain
```

2.使用 make 工具安装 sipe。

    make sipe
    >>> /usr/local/go/bin/go install -ldflags -X main.gitCommit=9d73f67e1dc5587a95f52c13fee93be6434b42ac -s -v ./cmd/sipe github.com/simplechain-org/go-simplechain/core
    ...
    github.com/simplechain-org/go-simplechain/cmd/sipe
    Done building.
    Run "/Users/yuanchao/go/src/github.com/simplechain-org/go-simplechain/build/bin/sipe" to launch sipe.

当终端出现以上输出时，表示 make 执行成功，此时在 go-simplechain/build/bin 目录下 将会生成 sipe 可执行文件。可以将其移动到任何目录下或将其加入到环境变量中，以此 来便利得运行sipe程序。

## 启动sipe

**1.创建用于存储节点数据的文件夹,如果不**

    mkdir chaindata

**2.启动sipe主网节点**

开启 RPC 服务并指定 RPC 监听地址为 127.0.0.1，端口 `8545`。节点数据存储目录为 `chaindata`

```bash
./sipe --rpc --rpcaddr 127.0.0.1 --rpcport 8545 --datadir chaindata 
```

当出现类似以下输出时，表示启动成功，并开始同步 SimpleChain 主网区块。

```bash
INFO [06-19|09:35:01.481] Maximum peer count               ETH=25 LES=0 total=25
INFO [06-19|09:35:01.492] Starting peer-to-peer node       instance=Sipe/v1.0.2-stable-0cbf2a41/darwin-amd64/go1.12.1
...
INFO [06-19|09:35:33.700] Block synchronisation started
INFO [06-19|09:35:36.756] Imported new block headers       count=192\
elapsed=22.273ms number=192 hash=bb758a...bea1b6 ignored=0
```

## 社区节点

**rpc地址和端口号：**

```bash
47.110.48.207:8545
```
**测试：**
```json
//Request
curl -X POST 47.110.48.207:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":68}'
//Response
{
	"jsonrpc": "2.0",
	"id": 68,
	"result": "Sipe/v1.1.0-stable-0800d402/linux-amd64/go1.14.1"
}
```

## 连接SimpleChain节点

创建一个 sipc.js，然后编写如下代码：

```javascript
const config = require('../../config') //节点服务器的配置信息
const Web3 = require('web3')
module.exports = new Web3(config.uri)
```
## 创建SimpleChain钱包服务

现在开始进入Simplechain钱包服务的核心特性开发阶段。

### 创建新的Simplechain账户

交易所和支付网关需要为客户生成新地址，以便用户可以向服务充值，或者为产品付费。生成一个没有用过的`SIMPLE`地址是任何数字资产服务的基本需求，因此看一下具体实现。

首先，创建一个commands.js，在其中我们订阅队列中的消息。主要包括以下几个步骤：

连接到command主题，监听新的create_account命令
当收到新的create_account命令时，创建新的密钥对并存入密码库
生成account_created消息并发送到队列的account_created主题
代码如下：

```javascript
const web3 = require("./ethereum")
/**
 * Create a new ethereum address and return the address 
 */
async function create_account(meta = {}) {
  // generate the address
  const account = await web3.eth.accounts.create()
  
  // disable checksum when storing the address
  const address = account.address.toLowerCase()
  
  // save the public address in Redis without any transactions received yet
  await redis.setAsync(`eth:address:public:${address}`, JSON.stringify({}))
  
  // Store the private key in a vault.
  // For demo purposes we use the same Redis instance, but this should be changed in production
  await redis.setAsync(`eth:address:private:${address}`, account.privateKey)
  
  return Object.assign({}, meta, {address: account.address})
}

module.exports.listen_to_commands = listen_to_commands
```
### 处理新交易

我们的钱包还没写完，当我们创建的地址收到用户充值时应当得到通知才对。为此，Simplechain的web3客户端提供了newBlockHeaders订阅机制。此外，如果我们的服务偶然宕机，那么服务就会错过在宕机期间生产的区块，因此我们还需要检查钱包是否已经同步到了网络的最新区块。

创建 `sync_blocks.js` 文件，编写如下代码：

```javascript
const web3 = require('./ethereum')

/**
 * Sync blocks and start listening for new blocks
 * @param {Number} current_block_number - The last block processed
 * @param {Object} opts - A list of options with callbacks for events
 */
async function sync_blocks(current_block_number, opts) {
  // first sync the wallet to the latest block
  let latest_block_number = await web3.eth.getBlockNumber()
  let synced_block_number = await sync_to_block(current_block_number, latest_block_number, opts)

  // subscribe to new blocks
  web3.eth.subscribe('newBlockHeaders', (error, result) => error && console.log(error))
  .on("data", async function(blockHeader) {
    return await process_block(blockHeader.number, opts)
  })

  return synced_block_number
}

// Load all data about the given block and call the callbacks if defined
async function process_block(block_hash_or_id, opts) {
  // load block information by id or hash
  const block = await web3.eth.getBlock(block_hash_or_id, true)
  // call the onTransactions callback if defined
  opts.onTransactions ? opts.onTransactions(block.transactions) : null;
  // call the onBlock callback if defined
  opts.onBlock ? opts.onBlock(block_hash_or_id) : null;
  return block
}

// Traverse all unprocessed blocks between the current index and the lastest block number
async function sync_to_block(index, latest, opts) {
  if (index >= latest) {
    return index;
  }
  await process_block(index + 1, opts)
  return await sync_to_block(index + 1, latest, opts)
}

module.exports = sync_blocks
```

在上面的代码中，我们从钱包服务之前处理的最新区块开始，一直同步到区块链的当前最新区块。一旦我们同步到最新区块，就开始订阅新区块事件。对于每一个区块，我们都执行如下的回调函数以处理区块头以及区块中的交易列表：

- `onTransactions`
- `onBlock`

**通常包含如下的处理步骤：**

- `监听新区块，获取区块中的全部交易`
- `过滤掉与钱包地址无关的交易`
- `将每个相关的交易都发往队列`
- `将地址上的资金归集到安全的存储`
- `更新已处理的区块编号`

**最终的代码如下：**

```javascript
const web3 = require("web3") //调用web3
const redis = require('./redis') //调用redis数据库，将区块数据获取下来存入redis数据库中
const queue = require('./queue') //调用消息队列
const sync_blocks = require('./sync_blocks')  //同步区块

/**
 * Start syncing blocks and listen for new transactions on the blockchain
 */
async function start_syncing_blocks() {
  // start from the last block number processed or 0 (you can use the current block before deploying for the first time)
  let last_block_number = await redis.getAsync('eth:last-block')
  last_block_number = last_block_number || 0
  // start syncing blocks
  sync_blocks(last_block_number, {
    // for every new block update the latest block value in redis
    onBlock: update_block_head,
    // for new transactions check each transaction and see if it's new
    onTransactions: async (transactions) => {
      for (let i in transactions) {
        await process_transaction(transactions[i])
      }
    }
  })
}

// save the lastest block on redis
async function update_block_head(head) {
  return await redis.setAsync('eth:last-block', head)
}

// process a new transaction
async function process_transaction(transaction) {
  const address = transaction.to.toLowerCase()
  const amount_in_ether = web3.utils.fromWei(transaction.value)

  // check if the receiving address has been generated by our wallet
  const watched_address = await redis.existsAsync(`eth:address:public:${address}`)
  if (watched_address !== 1) {
    return false
  }

  // then check if it's a new transaction that should be taken into account
  const transaction_exists = await redis.existsAsync(`eth:address:public:${address}`)
  if (transaction_exists === 1) {
    return false
  }

  // update the list of transactions for that address
  const data = await redis.getAsync(`eth:address:public:${address}`)
  let addr_data = JSON.parse(data)
  addr_data[transaction.hash] = {
    value: amount_in_ether
  }

  await redis.setAsync(`eth:address:public:${address}`, JSON.stringify(addr_data))
  await redis.setAsync(`eth:transaction:${transaction.hash}`, transaction)
  
  // move funds to the cold wallet address
  // const cold_txid = await move_to_cold_storage(address, amount_in_ether)
  
  // send notification to the kafka server
  await queue_producer.send('transaction', [{
    txid: transaction.hash,
    value: amount_in_ether,
    to: transaction.to,
    from: transaction.from,
    //cold_txid: cold_txid,
  }])

  return true
}

module.exports = start_syncing_blocks
```

# 总结

我们已经完成了交易所Simplechain钱包服务的设计与实现，这个服务还可以从以下几个方面加以改进：

- 增加错误处理
- 增加命令类型
- 交易签名与交易广播
- 部署合约



