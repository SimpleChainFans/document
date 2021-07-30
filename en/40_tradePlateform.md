---
id: docs_44
title: 平台对接
sidebar_label: 平台对接
---

## Build SimpleChain nodes

### docker build

**Obtain an image:**

```sh
docker pull simplechain/sipe:latest
```
**Enable RPC**

```sh
docker run -it -p 8545:8545 -p 30312:30312 simplechain/sipe --rpc --rpcaddr "0.0.0.0"
```
You can run the following command to check whether your node is successfully started:

```sh
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":68}'
```

### Source code construction

`Preliminary preparations: Go language environment (1.10 or above), C language compiler`

**1.Download SimpleChain**

You can clone a project locally through git or download directly from the https://github.com/simplechain-org/go-
simplechain page.

    git clone https://github.com/simplechain-org/go-simplechain.git 

**2.Install sipe**

1.Enter the go-simplechain root directory.

```javascript
cd go-simplechain
```

2.Use the make tool to install sipe.

    make sipe
    >>> /usr/local/go/bin/go install -ldflags -X main.gitCommit=9d73f67e1dc5587a95f52c13fee93be6434b42ac -s -v ./cmd/sipe github.com/simplechain-org/go-simplechain/core
    ...
    github.com/simplechain-org/go-simplechain/cmd/sipe
    Done building.
    Run "/Users/yuanchao/go/src/github.com/simplechain-org/go-simplechain/build/bin/sipe" to launch sipe.

When the above output appears on the terminal, the make execution is successful. In this case, the sipe executable file is generated in the go-simplechain/build/bin directory. You can move it to any directory or add it to environment variables to facilitate the running of sipe programs.

## Start sipe

**1.Create a folder for storing node data, if not**

    mkdir chaindata

**2.Start the sipe Master network node**

Enable the RPC service and specify the RPC listening address as 127.0.0.1, Port 8545 . The node data storage directory is `chaindata`

```bash
./sipe --rpc --rpcaddr 127.0.0.1 --rpcport 8545 --datadir chaindata 
```
When an output similar to the following appears, the startup is successful and the SimpleChain master Network block is synchronized.

```bash
INFO [06-19|09:35:01.481] Maximum peer count               ETH=25 LES=0 total=25
INFO [06-19|09:35:01.492] Starting peer-to-peer node       instance=Sipe/v1.0.2-stable-0cbf2a41/darwin-amd64/go1.12.1
...
INFO [06-19|09:35:33.700] Block synchronisation started
INFO [06-19|09:35:36.756] Imported new block headers       count=192\
elapsed=22.273ms number=192 hash=bb758a...bea1b6 ignored=0
```

## Community node

**rpc address and port number:**

```bash
47.110.48.207:8545
```
**Test:**
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

## Connect to the SimpleChain node

Create a simple.js file and write the following code:

```javascript
const config = require('../../config') //Node serve compile information
const Web3 = require('web3')
module.exports = new Web3(config.uri)
```
## Create SimpleChain wallet service

Now we are entering the development phase of the core features of Simplechain wallet service.

### Create a new Simplechain account

The exchange and payment gateway need to generate a new address for the customer so that the user can recharge the service or pay for the product. Generate an unused SIMPLE Address is the basic requirement of any digital asset service, so let's take a look at the specific implementation.

First, create a command.js where we subscribe to messages in the queue. It mainly includes the following steps:

Connect to the command topic and listen to the new create_account command When a new create_account command is received, a new key pair is created and stored in the password library.

The account_created topic that generates the account_created message and sends it to the queue.
The code is as follows:

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
### Handle new transactions

We haven't finished our wallet yet. When the address we created receives the user's recharge, it should be notified. To this end, the web3 client of Simplechain provides the newBlockHeaders subscription mechanism. In addition, if our service goes down accidentally, the service will miss the blocks produced during the downtime. Therefore, we also need to check whether the wallet has been synchronized to the latest blocks on the network.

Create `sync_blocks.js` file, write the following code:

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

In the preceding code, we synchronize the latest block processed by the wallet service to the current latest block in the blockchain. Once we synchronize to the latest block, we begin to subscribe to the new Block event. For each block, we execute the following callback function to process the block header and the transaction list in the block:

- `onTransactions`
- `onBlock`

**Generally, the following processing steps are included:**

- `Monitor new blocks and get all transactions in the block`
- `Filter out transactions that are not related to the wallet address`
- `Send every related transaction to the queue`
- `Gather funds on the address to secure storage`
- `Update processed block number`

**The final code is as follows:**

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

#  Summary

We have completed the design and implementation of Simplechain wallet service of the exchange. This service can also be improved from the following aspects:

- Add error handling
- Add command type
- Transaction Signature and transaction broadcast
- Deployment Contract



