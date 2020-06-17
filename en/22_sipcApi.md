#### Sipc API method list:

- [`web3_clientVersion`](#web3_clientVersion)
- [`web3_sha3`](#web3_sha3)
- [`net_version`](#net_version)
- [`net_listening`](#net_listening)
- [`net_peerCount`](#net_peerCount)
- [`eth_protocolVersion`](#eth_protocolVersion)
- [`eth_syncing`](#eth_syncing)
- [`eth_coinbase`](#eth_coinbase)
- [`eth_mining`](#eth_mining)
- [`eth_hashrate`](#eth_hashrate)
- [`eth_gasPrice`](#eth_gasPrice)
- [`eth_accounts`](#eth_accounts)
- [`eth_blockNumber`](#eth_blockNumber)
- [`eth_getBalance`](#eth_getBalance)
- [`eth_getStorageAt`](#eth_getTransactionCount)
- [`eth_getBlockTransactionCountByHash`](#eth_getBlockTransactionCountByHash)
- [`eth_getBlockTransactionCountByNumb`](#eth_getBlockTransactionCountByNumb)
- [`eth_getUncleCountByBlockHash`](#eth_getUncleCountByBlockHash)
- [`eth_getUncleCountByBlockNumber`](#eth_getUncleCountByBlockNumber)
- [`eth_getCode`](#eth_getCode)
- [`eth_sign`](#eth_sign)
- [`eth_sendTransaction`](#eth_sendTransaction)
- [`eth_sendRawTransaction`](#eth_sendRawTransaction)
- [`eth_call`](#eth_call)
- [`eth_estimateGas`](#eth_estimateGas)
- [`eth_getBlockByHash`](#eth_getBlockByHash)
- [`eth_getBlockByNumber`](#eth_getBlockByNumber)
- [`eth_getTransactionByNUmber`](#eth_getTransactionByNumber)
- [`eth_getUncleByBlockHashAndIndex`](#eth_getUncleByBlockHashAndIndex)
- [`eth_getUncleByBlockNumberAndIndex`](#eth_getUncleByBlockNumberAndIndex)
- [`eth_newFilter`](#eth_newFilter)
- [`eth_newBlockFilter`](#eth_newBlockFilter)
- [`eth_newPendingTransactionFilter`](#eth_newPendingTransactionFilter)
- [`eth_uninstallFilter`](#eth_uninstallFilter)
- [`eth_getFilterChanges`](#eth_getFilterChanges)
- [`eth_getFilterLogs`](#eth_getFilterLogs)
- [`eth_getLogs`](#eth_getLogs)
- [`eth_getWork`](#eth_getWork)
- [`eth_submitWork`](#eth_submitWork)
- [`eth_submitHashrate`](#eth_submitHashrate)
- [`eth_blockNumber`](#eth_blockNumber)
---

## JSON-RPC Endpoint

Default JSON-RPC endpoints:

| Client | URL                                             |
| ------ | ----------------------------------------------- |
| Go     | [http://localhost:8545](http://localhost:8545/) |

---

##  Default block parameters

The following method has an additional default block parameter:

- eth_getBalance
- eth.getCode
- eth.getTransactionCount
- eth_getStorageAt
- eth_call

When the request acts on the status of simplechain, the last default block parameter determines the height of the block.

The following options can be used for the default block parameters:

- `HEX String` -Integer number of blocks
- `String "earliest"` - The earliest/genesis block
- `String "latest"` - Newly dug blocks
- `String "pending"` - For pending status/transactions


## web3_clientVersion

Returns the current client version.

**Parameter**

- `No`

**Return**

- `string:` Current client version

**示例**

```javascript
// Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":68}'

//Result
{
   "id":67,
   "jsonrpc":"2.0",
   "result": "Mist/v0.9.3/darwin/go1.4.1" 
}
```
-----

## web3_sha3

Returns the keccak-256 of the given data (not a standardized sha3-256)

**Parameter**

- `string:` The data to calculate the SHA3 hash must be (hex string)

**Return value**

- `string:` Specifies the SHA3 result of the string.

**示例**

```javascript
  // Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"web3_sha3","params":["0x68656c6c6f20776f726c64"],"id":64}'

//Result
{
  "id":64,
  "jsonrpc": "2.0",
  "result": "0x47173285a8d7341e5e972fc677286384f802f8ef42a5ec5f03bbfa254cb01fad" 
}
```
----

## net_version

Returns the ID of the current connection network.

**Parameter**

- `no`

**Return value**

- `String:` The ID of the current connection network,`"1": Simplechain mainnet`,`"3": testnet`;

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"net_version","params":[],"id":67}'

//Response
{
   "jsonrpc": "2.0",
   "id":67,
   "result": "3" 
}
```
----

## net_listening

Returns whether the client is in the listening network connection state. If the client is in the listening state, the system returns `true`, otherwise return `false`.

**Parameter**

- `无`

**Return value**

- `bool:` Returns true when the client is listening. Otherwise, false is returned

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"net_listening","params":[],"id":67}'

//Response
{
   "jsonrpc":"2.0",
   "id":67,
   "result":true 
}
```
---

## net_peerCount

Returns the number of nodes currently connected to the client.

**Parameter**

- `No`
  
**Return value**

- `Quantity:` Number of connected nodes

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":74}'

//Response
{
  "id":74,
  "jsonrpc": "2.0",
  "result": "0xf" 
}
```
---

## eth_protocolVersion

Returns the protocol version of simplechain.

**Parameter**

- `no`

**返回**

- `String` The current protocol version of simplechain.

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_protocolVersion","params":[],"id":67}'

//Response
{
  "jsonrpc":"2.0",
  "id":67,
  "result":"0x40"
}
```
------

## eth_syncing

Returns an object that contains data about the synchronization status or false

**Number parameter**

- `No`

**Return value**

- `Object|Boolean`, synchronization status object or false. The structure of the synchronization object is as follows:
- `startingBlock:` QUANTITY - start Block
- `currentBlock:` QUANTITY - current block, and eth_blockNumber
- `highestBlock:` QUANTITY - maximum estimated block

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
//Response
{
   "id":1,
   "jsonrpc": "2.0", 
   "result": {
       startingBlock: '0x384',
       currentBlock: '0x386',
       highestBlock: '0x454' 
   }
}
{
   "jsonrpc": "2.0",
   "id":1,
   "result": false 
}
```
## eth_coinbase

Returns the client miner address. 

**Parameters**

- `No`

**Return value**

- `Data:` 20 bytes - current coinbase address

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_coinbase","params":[],"id":64}'

//Response
{
"id":64,
"jsonrpc": "2.0",
"result": "0x407d73d8a49eeb85d32cf465507dd71d507100c1" }
```

## eth_mining

If the client is mining, return `true`, otherwise, return `false`。

**Parameter**

- `no`

**Return value**

- `boolean` When the client is mining, it returns true , otherwise, return false .

**Exaple**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_mining","params":[],"id":71}'

//Response
{
   "id":71,
   "jsonrpc": "2.0",
   "result": true 
}
```

## eth_hashrate

Returns the hash rate per second that can be calculated during node mining.
 
**Parameter**

- `no`

**Return**

- `quantity:` Hash rate calculated per second

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_hashrate","params":[],"id":71}'

//Response
{
   "jsonrpc": "2.0",
   "id":71,
   "result": "0x38a" 
}
```
## eth_gasPrice
 
Returns each current `gas` price, unit:`wei`。 

**parameter**

- `no`

**Return value**

- `quantity:`Integer, `wei` Is the current unit `gas` price

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":73}'

//Response
{
  "jsonrpc": "2.0",
  "id":73,
  "result": "0x09184e72a000" // 10000000000000 
}
```

## eth_accounts

Returns the list of addresses held by the client.

**Parameter**

- `no`

**Return**

- `string[]:`String array, a list of address strings held by the client.

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":1}'

//Response
{
"id":1,
"jsonrpc": "2.0",
"result": ["0x407d73d8a49eeb85d32cf465507dd71d507100c1"] }
 ```

## eth_blockNumber

Returns the block number (block height) of the latest block.

**Parameter**

- `No`

**Return**

- The integer of the current block number where the client is located.
 
**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

//Response
{
  "jsonrpc":"2.0",
  "id":1,
  "result":"0x5a0d"
}
```
------

## eth_getBalance

Returns the balance of the account at the specified address.
 
**Parameter**

- `data`- 20 bytes, the address of the balance to be checked
- `quantity|tag` - Integer block number, or string "latest", "earliest" or "pending"

**Return**

- `quantity:`Current balance, unit: wei

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x51e766a7f073955c8061073bbba60b10bf12d48a", "latest"],"id":1}'
//Response
{
  "jsonrpc": "2.0",
  "id":1,
  "result": "0x0234c8a3397aab58" // 158972490234375000 
}
```

## eth_getTransactionCount

Returns the number of transactions at the specified address.

**Parameter**

- `DATA:` 20 bytes, address
- `QUANTITY|TAG` Integer block number, or string "latest", "earliest", or "pending"
 
**Return**

- `QUANTITY` The number of transactions sent from the specified address, an integer.

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getTransactionCount","params":["0xd79b8287a827e1387e6f0ff6d300fc663e10f592","latest"],"id":1}'

//Response
{
  "jsonrpc":"2.0",
  "id":1,
  "result":"0x1303"
} // 1
```

## eth_getBlockTransactionCountByHash

Returns the number of transactions in a specified block using a hash.

**Parameter**

- `DATA:` 20 bytes, address

**Return**

- `QUANTITY` The number of transactions in the specified block, integer

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByHash","params":["0x268343647d0fcf63628446a29959feccf57136dac58fd1c17e0df3babafce3b6"],"id":1}'
//Response
{
"id":1,
"jsonrpc": "2.0",
"result": "0x2" 
}
```
 
## eth_getBlockTransactionCountByNumber 

Returns the number of transactions in a block that matches a given block number.

**Parameter**

- `QUANTITY|TAG:` Integer block number, or string "earliest", "latest", or "pending"

**Return**

- `QUANTITY:` • The number of transactions in the specified block.

**Example**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByNumber","params":["0x11c7"],"id":1}'
//Response
{
  "jsonrpc": "2.0",
  "id":1,
  "result": "0xa" 
}
```
## eth_getUncleCountByBlockHash

从与给定块哈希匹配的块中返回叔块数。
 
**参数**

- `QUANTITY` DATA, 32 字节，块哈希

**返回值**

- `QUANTITY:` DATA，指定块的叔数，整数

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getUncleCountByBlockHash","params":["0x58052e3424b8c03643a3cd3595cad1a6104ab195cc78108318699f2bfa429d8f"],"id":1}'
//Response
{
  "jsonrpc": "2.0",
  "id":1,
  "result": "0x1"
}
```

## eth_getUncleCountByBlockNumber

从与给定块号匹配的块中返回块中的叔块数。

**参数**

- `QUANTITY|TAG` -区块数, 或者字符串 "latest", "earliest" or "pending"

**返回**

- `QUANTITY:` DATA，指定块的叔数，整数

**示例**

```json
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getUncleCountByBlockNumber","params":["0x1bb"],"id":1}'

//Response
{
  "jsonrpc":"2.0",
  "id":1,
  "result":"0x1"
}
```

------

## eth_getCode

返回指定地址的代码。

**参数**

- `string:` DATA ,20 字节，地址
- `string`  QUANTITY|TAG, 整数块编号，或字符串"latest"、"earliest" 或"pending"

**返回**

- `string` DATA: 指定地址处的代码

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getCode","params":["0xe558562c906c69787b8fabb6d0efb3f1163a20e4","latest"],"id":1}'

//Response
{
  "jsonrpc": "2.0",
  "id":1,
  "result": "0x"
}
```
## eth_sign

通过向消息添加前缀，可以将计算出的签名识别为特定于simplechain的签名。这可以防止恶意DAPP在签署任意数据（如事务）并使用签名来冒充受害者时的误用。

> **注意:** 必须先解锁要签名的地址.

**参数**

- `DATA` 20字节，地址
- `DATA` 要签名的消息

**返回值**

- `DATA` 数据的签名

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_sign","params":["0xb014763d71459855510255647be8cf39b0e82acb","0xdeadbeaf"],"id":1}'

//Response
{
  "jsonrpc": "2.0",
  "id":1,
  "result": "0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
}
```

## eth_sendTransaction

创建一个新的消息调用交易，如果数据字段中包含代码，则创建一个合约。

**参数**

- `from:` DATA, 20 字节 - 发送交易的源地址
- `to:` DATA, 20 字节 - 交易的目标地址，当创建新合约时可选
- `gas:` QUANTITY - 交易执行可用 gas 量，可选整数，默认值 90000，未用 gas 将返还。
- `gasPrice:` QUANTITY - gas 价格，可选，默认值:待定(To-Be-Determined)
- `value:` QUANTITY - 交易发送的金额，可选整数
- `data:` DATA - 合约的编译代码或被调用方法的签名及编码参数
- `nonce:` QUANTITY - nonce，可选。可以使用同一个 nonce 来重写挂起的交易

**返回**

- `DATA:`32 字节 - 交易哈希，如果交易还未生效则返回 0 值哈希。当创建合约时，在交易生效后，使用 eth_getTransactionReceipt 调用获取合约地址。

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_sendTransaction",
"params":[{
  "from": "0xb014763d71459855510255647be8cf39b0e82acb",
  "to": "0xac49f5e5b9161e0cb42372c90183d8428b060ff1",
  "gas": "0x76c0", 
  "gasPrice": "0x9184e72a000", 
  "value": "0x9184e72a", 
  "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"
}],"id":1}'

//Response
{
"id":1,
 "jsonrpc": "2.0",
"result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
}
```

## eth_sendRawTransaction

为已签名的交易创建新的消息调用交易或合约创建。

**参数**

- `DATA:` 签名的交易数据

**返回值**

- `DATA:` 32字节，交易哈希，如果交易未生效则返回全0哈希。当创建合约时，在交易生效后，使用eth_getTransactionReceipt获取合约地址。

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_sendRawTransaction","params":["0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"],"id":1}'
//Response
{
"id":1,
"jsonrpc": "2.0",
"result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
}
```
## eth_call

立刻执行一个新的消息调用，无需在区块链上创建交易。

**参数**

- `from:` DATA, 20 Bytes - 发送交易的原地址，可选
- `to:` DATA, 20 Bytes - 交易目标地址
- `gas:` QUANTITY - 交易可用 gas 量，可选。eth_call 不消耗 gas，但是某些 执行环节需要这个参数
- `gasPrice:` QUANTITY - gas 价格，可选
- `value:` QUANTITY - 交易发送的 sipc 数量，可选
- `data:` DATA - 方法签名和编码参数的哈希，可选
- `string:` QUANTITY|TAG - 整数块编号，或字符串"latest"、"earliest"或"pending" 

**返回值**

- `result:` DATA- 所执行合约的返回值

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_call","params":["0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675" ],"id":1}'
//Response
{
"id":1,
"jsonrpc": "2.0",
"result": "0x" }
```

## eth_estimateGas

执行并估算一个交易需要的gas用量。该次交易不会写入区块链。注意，由于多种原因，例如`EVM`的机制及节点的性能，估算的数值可能比实际用量大的多。

**参数**

- `from:` DATA, 20 Bytes - 发送交易的原地址，可选
- `to:` DATA, 20 Bytes - 交易目标地址
- `gas:` QUANTITY - 交易可用 gas 量，可选。eth_call 不消耗 gas，但是某些 执行环节需要这个参数
- `gasPrice:` QUANTITY - gas 价格，可选
- `value:` QUANTITY - 交易发送的 sipc 数量，可选
- `data:` DATA - 方法签名和编码参数的哈希，可选
- `string:` QUANTITY|TAG - 整数块编号，或字符串"latest"、"earliest"或"pending"

**返回值**

- `result:` QUANTITY - gas用量估算值

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_estimateGas","params":[{
  "from": "0xb014763d71459855510255647be8cf39b0e82acb",
  "to": "0xac49f5e5b9161e0cb42372c90183d8428b060ff1",
  "gas": "0x76c0", 
  "gasPrice": "0x9184e72a000", 
  "value": "0x9184e72a", 
  "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"
}],"id":1}'

//response
{
"id":1,
"jsonrpc": "2.0",
"result": "0x5cec" // 21000
 }
 ```

## eth_getBlockByHash

根据给定的哈希返回有关块的信息。

**参数**

- DATA, 32个字节 - 区块的哈希
- Boolean - 如果为true，则返回完整的交易对象，否则仅返回交易哈希

**返回值**

- `number:` QUANTITY - 块编号，挂起块为 null
- `hash:` DATA, 32 Bytes - 块哈希，挂起块为 null
- `parentHash:` DATA, 32 Bytes - 父块的哈希
- `nonce:` DATA, 8 Bytes - 生成的 pow 哈希，挂起块为 null
- `sha3Uncles:` DATA, 32 Bytes - 块中叔数据的 SHA3 哈希
- `logsBloom:` DATA, 256 Bytes - 块日志的 bloom 过滤器，挂起块为null
- `ansactionsRoot:` DATA, 32 Bytes - 块中的交易树根节点
- `stateRoot:` DATA, 32 Bytes - 块最终状态树的根节点
- `receiptsRoot:` DATA, 32 Bytes - 块交易收据树的根节点 
- `miner:` DATA, 20 Bytes - 挖矿奖励的接收账户
- `difficulty`: QUANTITY - 块难度，整数
- `totalDifficulty:` QUANTITY - 截止到本块的链上总难度 
- `extraData:` DATA - 块额外数据
- `size:` QUANTITY - 本块字节数
- `gasLimit:` QUANTITY - 本块允许的最大 gas 用量
- `gasUsed:` QUANTITY - 本块中所有交易使用的总 gas 用量
- `timestamp:` QUANTITY - 块时间戳
- `transactions:` Array - 交易对象数组，或 32 字节长的交易哈希数组
- `uncles:` Array - 叔哈希数组

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getBlockByHash","params":["0xbe024aeab6138b6adffd616314c707e2af8d165f871c8068d1a4a9c38a59c69b", true],"id":1}'
//Response
{
"id":1, 
"jsonrpc":"2.0",
"result": {
    "number": "0x1b4", // 436
    "hash": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
    "parentHash": "0x9646252be9520f6e71339a8df9c55e4d7619deeb018d2a3f2d21fc165dde5eb5",
    "nonce": "0xe04d296d2460cfb8472af2c5fd05b5a214109c25688d3704aed5484f9a7792f2",
    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
    "logsBloom": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
    "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
    "stateRoot": "0xd5855eb08b3387c0af375e9cdb6acfc05eb8f519e419b874b6ff2ffda7ed1dff",
    "miner": "0x4e65fda2159562a496f9f3522f89122a3088497a", "difficulty": "0x027f07", // 163591
    "totalDifficulty": "0x027f07", // 163591
    "extraData": "0x0000000000000000000000000000000000000000000000000000000 000000000",
    "size": "0x027f07", // 163591
    "gasLimit": "0x9f759", // 653145 
    "gasUsed": "0x9f759", // 653145 
    "timestamp": "0x54e34e8e" // 1424182926 
    "transactions": [{...},{ ... }]
    "uncles": ["0x1606e5...", "0xd5145a9..."] 
    }
}
```

## eth_getBlockByNumber

根据区块数（区块高度）返回有关块的信息。

**参数**

- `QUANTITY|TAG` 区块数, 或者字符串 `"earliest"`, `"latest"` or `"pending"`
- `Boolean` - 如果为true，则返回完整的交易对象；如果为false，则仅返回交易的哈希值。

**返回**

- `number:` QUANTITY - 块编号，挂起块为 null
- `hash:` DATA, 32 Bytes - 块哈希，挂起块为 null
- `parentHash:` DATA, 32 Bytes - 父块的哈希
- `nonce:` DATA, 8 Bytes - 生成的 pow 哈希，挂起块为 null
- `sha3Uncles:` DATA, 32 Bytes - 块中叔数据的 SHA3 哈希
- `logsBloom:` DATA, 256 Bytes - 块日志的 bloom 过滤器，挂起块为null
- `ansactionsRoot:` DATA, 32 Bytes - 块中的交易树根节点
- `stateRoot:` DATA, 32 Bytes - 块最终状态树的根节点
- `receiptsRoot:` DATA, 32 Bytes - 块交易收据树的根节点 
- `miner:` DATA, 20 Bytes - 挖矿奖励的接收账户
- `difficulty`: QUANTITY - 块难度，整数
- `totalDifficulty:` QUANTITY - 截止到本块的链上总难度 
- `extraData:` DATA - 块额外数据
- `size:` QUANTITY - 本块字节数
- `gasLimit:` QUANTITY - 本块允许的最大 gas 用量
- `gasUsed:` QUANTITY - 本块中所有交易使用的总 gas 用量
- `timestamp:` QUANTITY - 块时间戳
- `transactions:` Array - 交易对象数组，或 32 字节长的交易哈希数组
- `uncles:` Array - 叔哈希数组

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x6bcd", true],"id":1}'
//Response
{
"id":1, 
"jsonrpc":"2.0",
"result": {
    "number": "0x1b4", // 436
    "hash": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
    "parentHash": "0x9646252be9520f6e71339a8df9c55e4d7619deeb018d2a3f2d21fc165dde5eb5",
    "nonce": "0xe04d296d2460cfb8472af2c5fd05b5a214109c25688d3704aed5484f9a7792f2",
    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
    "logsBloom": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
    "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
    "stateRoot": "0xd5855eb08b3387c0af375e9cdb6acfc05eb8f519e419b874b6ff2ffda7ed1dff",
    "miner": "0x4e65fda2159562a496f9f3522f89122a3088497a", "difficulty": "0x027f07", // 163591
    "totalDifficulty": "0x027f07", // 163591
    "extraData": "0x0000000000000000000000000000000000000000000000000000000 000000000",
    "size": "0x027f07", // 163591
    "gasLimit": "0x9f759", // 653145 
    "gasUsed": "0x9f759", // 653145 
    "timestamp": "0x54e34e8e" // 1424182926 
    "transactions": [{...},{ ... }]
    "uncles": ["0x1606e5...", "0xd5145a9..."] 
    }
}
```
## eth_getTransactionByHash

返回有关交易哈希请求的交易的信息

**参数**

- `DATA`, 32个字节 - 交易的哈希

**返回**

- `blockHash`: `DATA`, 32 Bytes -此交易所在的区块的哈希。当它还处于待定状态时为null。
- `blockNumber`: `QUANTITY` - 此交易所在的区块的高度. 当它还处于待定状态时为null。
- `from`: `DATA`, 20 个字节 -交易发起者的地址。
- `gas`: `QUANTITY` -交易发起者提供的gas数。
- `gasPrice`: `QUANTITY` - 发送者提供的gas的价格，以wei为单位。
- `hash`: `DATA`, 32个字节 -交易的哈希
- `input`: `DATA` - 随交易一起发送的数据。
- `nonce`: `QUANTITY` - 发送方在此之前进行的交易数。
- `to`: `DATA`, 20个字节 -接受者的地址，当交易是一个合约创建的时候，它的值为null。
- `transactionIndex`: `QUANTITY` - 交易在块中的索引位置的整数。当它为待定状态时为null。
- `value`: `QUANTITY` - 转账的数额，以wei为单位。
- `v`: `QUANTITY` - ECDSA recovery id
- `r`: `DATA`, 32 Bytes - ECDSA signature r
- `s`: `DATA`, 32 Bytes - ECDSA signature s

**示例**

```json
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getTransactionByHash","params":["0x8e63b031ac4f3a9c38642d69a86f73368fee539d9351e4eb312b5cfb6cd4f3e6"],"id":1}'

//Response
{
	"jsonrpc": "2.0",
	"id": 1,
	"result": {
		"blockHash": "0xbe024aeab6138b6adffd616314c707e2af8d165f871c8068d1a4a9c38a59c69b",
		"blockNumber": "0x6bcd",
		"from": "0xd79b8287a827e1387e6f0ff6d300fc663e10f592",
		"gas": "0x5c20",
		"gasPrice": "0x218711a00",
		"hash": "0x8e63b031ac4f3a9c38642d69a86f73368fee539d9351e4eb312b5cfb6cd4f3e6",
		"input": "0xd3182ceafbaf2da3503237a4f60b74e30756e78f310d2b97761c38db06753c99353535363530",
		"nonce": "0xecc",
		"to": "0xd79b8287a827e1387e6f0ff6d300fc663e10f592",
		"transactionIndex": "0x3",
		"value": "0xde0b6b3a7640000",
		"v": "0x26",
		"r": "0x5076aebd122ba56eaa0d40a44ce950ac792851e97e1a4afcf660afbc8d1e625a",
		"s": "0x7a7af2b80ccbdce36363b338b96a3f6c96aafa7057ea586a4dae0e30da607747"
	}
}
```
------

## eth_getTransactionByBlockHashAndIndex

返回指定块内具有指定索引序号的交易。

**参数**

- `DATA`, 32 字节 - 块哈希
- `QUANTITY`, 交易在块内的索引序号

**返回**

- `hash:` DATA, 32 字节 - 交易哈希
- `nonce:` QUANTITY - 本次交易之前发送方已经生成的交易数量
- `blockHash:` DATA, 32 字节 - 交易所在块的哈希，对于挂起块，该值为 null 
- `blockNumber:` QUANTITY - 交易所在块的编号，对于挂起块，该值为 null
- `transactionIndex:` QUANTITY - 交易在块中的索引位置，挂起块该值为 null
- `from:` DATA, 20 字节 - 交易发送方地址
- `to:` DATA, 20 字节 - 交易接收方地址，对于合约创建交易，该值为 null
- `value:` QUANTITY - 发送的 sipc 数量，单位:wei
- `gasPrice:` QUANTITY - 发送方提供的 gas 价格，单位:wei 
- `gas:` QUANTITY - 发送方提供的 gas 可用量
- `input:` DATA - 随交易发送的数据

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getTransactionByBlockHashAndIndex","params":["0xbe024aeab6138b6adffd616314c707e2af8d165f871c8068d1a4a9c38a59c69b", "0x3"],"id":1}'
//Response
{
"jsonrpc": "2.0",
	"id": 1,
	"result": {
		"blockHash": "0xbe024aeab6138b6adffd616314c707e2af8d165f871c8068d1a4a9c38a59c69b",
		"blockNumber": "0x6bcd",
		"from": "0xd79b8287a827e1387e6f0ff6d300fc663e10f592",
		"gas": "0x5c20",
		"gasPrice": "0x218711a00",
		"hash": "0x8e63b031ac4f3a9c38642d69a86f73368fee539d9351e4eb312b5cfb6cd4f3e6",
		"input": "0xd3182ceafbaf2da3503237a4f60b74e30756e78f310d2b97761c38db06753c99353535363530",
		"nonce": "0xecc",
		"to": "0xd79b8287a827e1387e6f0ff6d300fc663e10f592",
		"transactionIndex": "0x3",
		"value": "0xde0b6b3a7640000",
		"v": "0x26",
		"r": "0x5076aebd122ba56eaa0d40a44ce950ac792851e97e1a4afcf660afbc8d1e625a",
		"s": "0x7a7af2b80ccbdce36363b338b96a3f6c96aafa7057ea586a4dae0e30da607747"
	}
}
```
------

## eth_getTransactionByBlockNumberAndIndex

按区块数（区块高度）和交易索引位置返回有关交易的信息。

**参数**

- `QUANTITY|TAG` - 整数块编号，或字符串"earliest"、"latest" 或"pending" 
- `QUANTITY` - 交易索引序号

**返回值**

- `hash:` DATA, 32 字节 - 交易哈希
- `nonce:` QUANTITY - 本次交易之前发送方已经生成的交易数量
- `blockHash:` DATA, 32 字节 - 交易所在块的哈希，对于挂起块，该值为 null 
- `blockNumber:` QUANTITY - 交易所在块的编号，对于挂起块，该值为 null
- `transactionIndex:` QUANTITY - 交易在块中的索引位置，挂起块该值为 null
- `from:` DATA, 20 字节 - 交易发送方地址
- `to:` DATA, 20 字节 - 交易接收方地址，对于合约创建交易，该值为 null
- `value:` QUANTITY - 发送的 sipc 数量，单位:wei
- `gasPrice:` QUANTITY - 发送方提供的 gas 价格，单位:wei 
- `gas:` QUANTITY - 发送方提供的 gas 可用量
- `input:` DATA - 随交易发送的数据

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getTransactionByBlockNumberAndIn dex","params":["0x29c", "0x0"],"id":1}'
//Response
{
"id":1, 
"jsonrpc":"2.0", 
"result": {
    "hash":"0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d105 5b",
    "nonce":"0x",
    "blockHash": "0xbeab0aa2411b7ab17f30a99d3cb9c6ef2fc5426d6ad6fd9e2a26a6aed1d1055b",
    "blockNumber": "0x15df", // 5599
    "transactionIndex": "0x1", // 1 
    "from":"0x407d73d8a49eeb85d32cf465507dd71d507100c1", "to":"0x85h43d8a49eeb85d32cf465507dd71d507100c1", 
    "value":"0x7f110", // 520464
    "gas": "0x7f110", // 520464 
    "gasPrice":"0x09184e72a000", 
    "input":"0x603880600c6000396000f300603880600c6000396000f3603880600c6000396000f360", 
    }
}
```

## eth_getTransactionReceipt

根据交易哈希返回一个交易的收据,注意:收据不可用于待处理的交易。

**参数**

- DATA, 32字节，交易哈希

**返回**

-  `transactionHash:` DATA, 32 字节 - 交易哈希
-  `transactionIndex:` QUANTITY - 交易在块内的索引序号
-  `blockHash:` DATA, 32 字节 - 交易所在块的哈希
-  `blockNumber:` QUANTITY - 交易所在块的编号
-  `from:` DATA, 20 字节 - 交易发送方地址
-  `to:` DATA, 20 字节 - 交易接收方地址，对于合约创建交易该值为 null
-  `cumulativeGasUsed:` QUANTITY - 交易所在块消耗的 gas 总量
-  `gasUsed:` QUANTITY - 该次交易消耗的 gas 用量
-  `contractAddress:` DATA, 20 字节 - 对于合约创建交易，该值为新创建的合 约地址，否则为 null
-  `logs:` Array - 本次交易生成的日志对象数组
-  `logsBloom:` DATA, 256 字节 - bloom 过滤器，轻客户端用来快速提取相关日志
-  `root:` DATA 32 字节，后交易状态根(pre Byzantium) 
-  `status:` QUANTITY ，1 (成功) 或 0 (失败)

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getTransactionReceipt","params":["0x b903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'
//Response
{
"id":1,
"jsonrpc":"2.0", 
"result": {
  transactionHash: '0xb903239f8543d04b5dc1ba6579132b143087c68db1b216878640 8fcbce568238',
  transactionIndex: '0x1', // 1
  blockNumber: '0xb', // 11
  blockHash: '0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1 d1055b',
  cumulativeGasUsed: '0x33bc', // 13244
  gasUsed: '0x4dc', // 1244
  contractAddress: '0xb60e8dd61c5d32be8058bb8eb970870f07233155', // or null, if none was created logs: [{
// logs as returned by getFilterLogs, etc. }, ...],
logsBloom: "0x00...0", // 256 byte bloom filter
status: '0x1' }
}
```

## eth_getUncleByBlockHashAndIndex

根据哈希和叔块索引位置返回有关块的叔块的信息。

**参数**

- `DATA`, 32字节,块哈希 
- `QUANTITY`, 叔索引位置

**返回值**

- `number:` QUANTITY - 块编号，挂起块为 null
- `hash:` DATA, 32 Bytes - 块哈希，挂起块为 null
- `parentHash:` DATA, 32 Bytes - 父块的哈希
- `nonce:` DATA, 8 Bytes - 生成的 pow 哈希，挂起块为 null
- `sha3Uncles:` DATA, 32 Bytes - 块中叔数据的 SHA3 哈希
- `logsBloom:` DATA, 256 Bytes - 块日志的 bloom 过滤器，挂起块为null
- `ansactionsRoot:` DATA, 32 Bytes - 块中的交易树根节点
- `stateRoot:` DATA, 32 Bytes - 块最终状态树的根节点
- `receiptsRoot:` DATA, 32 Bytes - 块交易收据树的根节点 
- `miner:` DATA, 20 Bytes - 挖矿奖励的接收账户
- `difficulty`: QUANTITY - 块难度，整数
- `totalDifficulty:` QUANTITY - 截止到本块的链上总难度 
- `extraData:` DATA - 块额外数据
- `size:` QUANTITY - 本块字节数
- `gasLimit:` QUANTITY - 本块允许的最大 gas 用量
- `gasUsed:` QUANTITY - 本块中所有交易使用的总 gas 用量
- `timestamp:` QUANTITY - 块时间戳
- `transactions:` Array - 交易对象数组，或 32 字节长的交易哈希数组
- `uncles:` Array - 叔哈希数组

**示例**
```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getUncleByBlockHashAndIndex","params":["0x58052e3424b8c03643a3cd3595cad1a6104ab195cc78108318699f2bfa429d8f",'0x0'],"id":1}'

//Response
{
	"jsonrpc": "2.0",
	"id": 1,
	"result": {
		"difficulty": "0x267c3",
		"extraData": "0xd983010000847369706588676f312e31312e318664617277696e",
		"gasLimit": "0x12804",
		"gasUsed": "0x0",
		"hash": "0x7937f1ac55c87d4fb649c17d0c9411e8837d32e10bf5e8723611120b50154c16",
		"logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
		"miner": "0x41da014ee4c2f583ba222344c095a631e739f2f1",
		"mixHash": "0xa50630bdd5800368de83fa719373ccceaaf69e56f07d32db11b2069c2af9a611",
		"nonce": "0x4f0713dd54bb9301",
		"number": "0x1b5",
		"parentHash": "0xf2ad2fbdd67936df73c447eabb49965d2bced51df9c1a80545b001600547d81a",
		"receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
		"sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
		"size": "0x21b",
		"stateRoot": "0xbfb096cd9832414ad738ff806dcfb1e48d9836c00d130b723a3bb808098b1099",
		"timestamp": "0x5c2e3b64",
		"totalDifficulty": null,
		"transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
		"uncles": []
	}
}
注意: 叔块不包含单独的交易。
```
---

## eth_getUncleByBlockNumberAndIndex

根据区块数（区块高度）和叔块索引位置返回有关叔块的信息.

**参数**

- QUANTITY|TAG - 整数块编号，或字符串"earliest"、"latest" 或"pending"
- QUANTITY – 叔块在块内的索引序号

**返回**

- `number:` QUANTITY - 块编号，挂起块为 null
- `hash:` DATA, 32 Bytes - 块哈希，挂起块为 null
- `parentHash:` DATA, 32 Bytes - 父块的哈希
- `nonce:` DATA, 8 Bytes - 生成的 pow 哈希，挂起块为 null
- `sha3Uncles:` DATA, 32 Bytes - 块中叔数据的 SHA3 哈希
- `logsBloom:` DATA, 256 Bytes - 块日志的 bloom 过滤器，挂起块为null
- `ansactionsRoot:` DATA, 32 Bytes - 块中的交易树根节点
- `stateRoot:` DATA, 32 Bytes - 块最终状态树的根节点
- `receiptsRoot:` DATA, 32 Bytes - 块交易收据树的根节点 
- `miner:` DATA, 20 Bytes - 挖矿奖励的接收账户
- `difficulty`: QUANTITY - 块难度，整数
- `totalDifficulty:` QUANTITY - 截止到本块的链上总难度 
- `extraData:` DATA - 块额外数据
- `size:` QUANTITY - 本块字节数
- `gasLimit:` QUANTITY - 本块允许的最大 gas 用量
- `gasUsed:` QUANTITY - 本块中所有交易使用的总 gas 用量
- `timestamp:` QUANTITY - 块时间戳
- `transactions:` Array - 交易对象数组，或 32 字节长的交易哈希数组
- `uncles:` Array - 叔哈希数组

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getUncleByBlockNumberAndIndex","params":["0x1bb","0x0"],"id":1}'
//Response
{
	"jsonrpc": "2.0",
	"id": 1,
	"result": {
		"difficulty": "0x267c3",
		"extraData": "0xd983010000847369706588676f312e31312e318664617277696e",
		"gasLimit": "0x12804",
		"gasUsed": "0x0",
		"hash": "0x7937f1ac55c87d4fb649c17d0c9411e8837d32e10bf5e8723611120b50154c16",
		"logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
		"miner": "0x41da014ee4c2f583ba222344c095a631e739f2f1",
		"mixHash": "0xa50630bdd5800368de83fa719373ccceaaf69e56f07d32db11b2069c2af9a611",
		"nonce": "0x4f0713dd54bb9301",
		"number": "0x1b5",
		"parentHash": "0xf2ad2fbdd67936df73c447eabb49965d2bced51df9c1a80545b001600547d81a",
		"receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
		"sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
		"size": "0x21b",
		"stateRoot": "0xbfb096cd9832414ad738ff806dcfb1e48d9836c00d130b723a3bb808098b1099",
		"timestamp": "0x5c2e3b64",
		"totalDifficulty": null,
		"transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
		"uncles": []
	}
}

注意: 叔块不包含单独的交易
------
```

## eth_newFilter

根据过滤器器选项创建过滤器对象，以便在状态更改（日志）时通知。要检查状态是否已更改，请调用eth getfilterchanges。

**关于指定主题过滤器的说明:**

关于特定主题过滤器的说明:主题是顺序相关的。如果一个交易的日志有主题[A, ，那么将被以下的主题过滤器匹配:

- []任何主题
- [A]先匹配A主题
- [null,B]先匹配其他主题，再匹配B主题
- [A,B]先匹配A主题，再匹配B主题，最后匹配其他主题
- [[A,B][A,B]"先匹配A主题或B主题，再匹配A主题或B主题,最后匹配其他主题"]

**参数**

- `fromBlock:` QUANTITY|TAG - 可选，默认值:"latest"。整数块编号，或字符串"latesr"表示最后挖出的块，"pending"或"earliest"用于未挖出的交易。 


- `toBlock:` QUANTITY|TAG - 可选，默认值:"latest"。整数块编号，或字符串"latesr"表示最后挖出的块，"pending"或"earliest"用于未挖出的交易。 
- `address:` DATA|Array, 20 字节 - 可选，合约地址或生成日志的一组地址
- `topics:` Array of DATA, - 可选，32 字节主题数组，每个主题可以是数组或使用 or 选项连接

**返回**

- QUANTITY，过滤器编号

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_newBlockFilter","params":[],"id":73}'
//Response
{
"jsonrpc": "2.0", 
"id":1,
"result": "0x1" // 1
}
```

## eth_newBlockFilter

在节点中创建一个过滤器，以便当新块生成时进行通知。要检查状态是否变化，请调用`eth_getFilterChanges`.

**参数**

`无`

**返回**

- `QUANTITY` 过滤器编号

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_newBlockFilter","params":[],"id":73}'
//Response
{
"jsonrpc": "2.0",
"id":1,
"result": "0x1" // 1
}
```

## eth_newPendingTransactionFilter

在节点中创建一个过滤器，以便当产生挂起交易时进行通知。要检查状态是否发生变化，请调用`eth_getFilterChanges`。

**参数**

`无`

**返回值**

- `QUANTITY`, 过滤器编号

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_newPendingTransactionFilter","para ms":[],"id":73}'
//Response
{
"id":1,
"jsonrpc": "2.0",
"result": "0x1" // 1 
}
```
## eth_uninstallFilter

卸载具有指定编号的过滤器。当不在需要监听时，总是需要执行该调用。另外，过滤器如果在一定时间内未接收到`eth_getFilterChanges`调用会自动超时。

**参数**

- `QUANTITY`, 过滤器编号

**返回**

- `Boolean`, 如果成功卸载则返回true,否则返回false

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_uninstallFilter","params":["0xb"],"id":7 3}'
//Response
{
  "jsonrpc": "2.0",
  "id":1,
  "result": true 
}
```

## eth_getFilterChanges

轮询指定的过滤器，并返回自上次轮询之后新生成的日志数组。

**参数**

- `QUANTITY`, 过滤器编号
 
**返回值**

- `Array`, 日志对象数组，如果没有新生成的日志，则返回空数组。
- 使用`eth_newBlockerFilter`创建的过滤器将返回块哈希(32 字节)，例如[")x3454645634534"]。
- 使用`eth_newPendingTransactionFilter`创建的过滤器将返回交易哈希 (32 字节)，例如["0x6345343454645..."]。
使用`eth_newFilter`创建的过滤器，日志对象具有如下数参数：

- `removed:` TAG - 如果日志已被删除则返回 true，如果是有效日志则返回false
- `logIndex:` QUANTITY - 日志在块内的索引序号。对于挂起日志，该值为 null
- `transactionIndex:` QUANTITY - 创建日志的交易索引序号，对于挂起日志，该值为 null
- `transactionHash:` DATA, 32 字节 - 创建该日志的交易的哈希。对于挂起日志，该值为 null
- `blockHash:` DATA, 32 字节 - 该日志所在块的哈希。对于挂起日志，该值为null
- `blockNumber:` QUANTITY - 该日志所在块的编号。对于挂起日志，该值为 null
- `address:` DATA, 20 字节 - 该日志的源地址
- `data:` DATA - 包含该日志的一个或多个 32 字节无索引参数
- `topics:` Array of DATA -0~4 个 32 字节索引日志参数的数据。在 `solidity` 中，第一个主题是事件签名，例如 `Deposit(address,bytes32,uint256)`，除非你声明的是匿名事件

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getFilterChanges","params":["0x16"]," id":73}'
//Response
{
  "id":1,
  "jsonrpc":"2.0", "result": [{
  "logIndex": "0x1", // 1
  "blockNumber":"0x1b4", // 436
  "blockHash": "0x8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcfdf829c5a142f1fccd7d",
  "transactionHash": "0xdf829c5a142f1fccd7d8216c5785ac562ff41e2dcfdf5785ac562f f41e2dcf",
  "transactionIndex": "0x0", // 0
  "address": "0x16c5785ac562ff41e2dcfdf829c5a142f1fccd7d",
  "data":"0x0000000000000000000000000000000000000000000000000000000000000 000",
  "topics": ["0x59ebeb90bc63057b6515673c3ecf9438e5058bca0f92585014eced636878c9a5"] },{... }]
}  
```

## eth_getFilterLogs

返回与给定ID匹配的过滤器的所有日志的数组.

**参数**

- `QUANTITY`, 过滤器编号

**返回**

- `removed:` TAG - 如果日志已被删除则返回true，如果是有效日志则返回false
- `logIndex:` QUANTITY - 日志在块内的索引序号。对于挂起日志，该值为null
- `transactionIndex:` QUANTITY - 创建日志的交易索引序号，对于挂起日志，该值为null
- `transactionHash:` DATA, 32 字节 - 创建该日志的交易的哈希。对于挂起日志，该值为null
- `blockHash:` DATA, 32 字节 - 该日志所在块的哈希。对于挂起日志，该值为null
- `blockNumber:` QUANTITY - 该日志所在块的编号。对于挂起日志，该值为null
- `address:` DATA, 20 字节 - 该日志的源地址
- `data:` DATA - 包含该日志的一个或多个32字节无索引参数
- `topics:` Array of DATA -0~4 个 32字节索引日志参数的数据。在 `solidity` 中，第一个主题是事件签名，例如 `Deposit(address,bytes32,uint256)`，除非你声明的是匿名事件

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getFilterLogs","params":["0x16"],"id": 74}'
//Response
{
  "id":1,
  "jsonrpc":"2.0", "result": [{
  "logIndex": "0x1", // 1
  "blockNumber":"0x1b4", // 436
  "blockHash": "0x8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcfdf829c5a142f1fccd7d",
  "transactionHash": "0xdf829c5a142f1fccd7d8216c5785ac562ff41e2dcfdf5785ac562f f41e2dcf",
  "transactionIndex": "0x0", // 0
  "address": "0x16c5785ac562ff41e2dcfdf829c5a142f1fccd7d",
  "data":"0x0000000000000000000000000000000000000000000000000000000000000 000",
  "topics": ["0x59ebeb90bc63057b6515673c3ecf9438e5058bca0f92585014eced636878c9a5"] },{... }]
}  
```
 
## eth_getLogs

返回与给定过滤器对象匹配的所有日志的数组。

**参数**

- `fromBlock:` QUANTITY|TAG - 可选，默认值:"latest"。整数块编号，或字符串"latesr"表示最后挖出的块，"pending"或"earliest"用于未挖出的交易。 
- `toBlock:` QUANTITY|TAG - 可选，默认值:"latest"。整数块编号，或字符串"latesr"表示最后挖出的块，"pending"或"earliest"用于未挖出的交易。 
- `address:` DATA|Array, 20 字节 - 可选，合约地址或生成日志的一组地址
- `topics:` Array of DATA, - 可选，32字节主题数组，每个主题可以是数组或使用or选项连接

**返回值**

- `Array`, 日志对象数组，如果没有新生成的日志，则返回空数组。
- 使用`eth_newBlockerFilter`创建的过滤器将返回块哈希(32 字节)，例如[")x3454645634534"]。
- 使用`eth_newPendingTransactionFilter`创建的过滤器将返回交易哈希 (32 字节)，例如["0x6345343454645..."]。
使用`eth_newFilter`创建的过滤器，日志对象具有如下数参数：

- `removed:` TAG - 如果日志已被删除则返回 true，如果是有效日志则返回false
- `logIndex:` QUANTITY - 日志在块内的索引序号。对于挂起日志，该值为 null
- `transactionIndex:` QUANTITY - 创建日志的交易索引序号，对于挂起日志，该值为 null
- `transactionHash:` DATA, 32 字节 - 创建该日志的交易的哈希。对于挂起日志，该值为 null
- `blockHash:` DATA, 32 字节 - 该日志所在块的哈希。对于挂起日志，该值为null
- `blockNumber:` QUANTITY - 该日志所在块的编号。对于挂起日志，该值为 null
- `address:` DATA, 20 字节 - 该日志的源地址
- `data:` DATA - 包含该日志的一个或多个 32 字节无索引参数
- `topics:` Array of DATA -0~4 个 32 字节索引日志参数的数据。在 `solidity` 中，第一个主题是事件签名，例如 `Deposit(address,bytes32,uint256)`，除非你声明的是匿名事件

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getLogs","params":[{"topics":["0x0000 00000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b"]}],"id":74}'
//Response
{
  "id":1,
  "jsonrpc":"2.0", "result": [{
  "logIndex": "0x1", // 1
  "blockNumber":"0x1b4", // 436
  "blockHash": "0x8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcfdf829c5a142f1fccd7d",
  "transactionHash": "0xdf829c5a142f1fccd7d8216c5785ac562ff41e2dcfdf5785ac562f f41e2dcf",
  "transactionIndex": "0x0", // 0
  "address": "0x16c5785ac562ff41e2dcfdf829c5a142f1fccd7d",
  "data":"0x0000000000000000000000000000000000000000000000000000000000000 000",
  "topics": ["0x59ebeb90bc63057b6515673c3ecf9438e5058bca0f92585014eced636878c9a5"] },{... }]
}  

```
## eth_getWork

返回当前块的哈希、seedhash和要满足的边界条件

**参数**

`无`

**返回值**

- `DATA`, 32 字节 - 当前块头的 pow-hash
- `DATA`, 32 字节 - 用于 DAG 的种子哈希
- `DATA`, 32 字节 - 边界条件，目标， 2^256 / difficulty
 
```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_getWork","params":[],"id":73}'
//Response
{
   "id":1,
   "jsonrpc":"2.0", 
   "result": [
      "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef", 
      "0x5EED00000000000000000000000000005EED0000000000000000000000000000",
      "0xd1ff1c01710000000000000000000000d1ff1c01710000000000000000000000"
   ] 
}
```

## eth_submitWork

用于提交POW解决方案。
 
**参数**

- `DATA`, 8 字节 - nonce，64 位
- `DATA`, 32 字节 - 头部的 pow 哈希，256 位  DATA, 32 字节 - 混合摘要，256 位

**返回**

- `Boolean`, 如果提交的方案有效则返回true，否则返回false

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0", "method":"eth_submitWork", "params":["0x000000 0000000001", "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890a bcdef", "0xD1GE5700000000000000000000000000D1GE5700000000000000000000000 000"],"id":73}'
//Response
{
  "id":73,
  "jsonrpc":"2.0",
  "result": true 
}

```

## eth_submitHashrate

用于提交挖矿的哈希速率。
 
**参数**

- `hashRate` - 哈希速率，采用 16 进制字符串表示，32 字节
- `ID`, String - 随机 16 进制字符串，32 字节，用于标识客户端的编号

**返回**

- `Boolean`, 如果提交成功则返回 true，否则返回 false
**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0", "method":"eth_submitHashrate", "params":["0x000 0000000000000000000000000000000000000000000000000000000500000", "0x59daa2 6581d0acd1fce254fb7e85952f4c09d0915afd33d3886cd914bc7d283c"],"id":73}'
//Response
{
   "id":73,
   "jsonrpc":"2.0",
   "result": true 
}
```

## eth_blockNumber

返回最新块的编号。

**参数**

`无`

**返回值**

- `QUANTITY`, 节点当前块编号

**示例**

```javascript
//Request
curl -X POST localhost:8545  -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":83}'
//Response
{
"id":83,
"jsonrpc": "2.0",
"result": "0x4b7" // 1207 
}
```
