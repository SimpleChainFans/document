---
id: docs_26
title: 跨链API
sidebar_label: 跨链API
---

**跨链API接口列表**

- [`eth_ctxQuery`](#eth_ctxQuery)
- [`eth_ctxOwner`](#eth_ctxOwner)
- [`eth_ctxOwnerByPage`](#eth_ctxOwnerByPage)
- [`eth_ctxContent`](#eth_ctxContent)
- [`eth_ctxContent`](#eth_ctxContent)
- [`eth_getRemoteCtx`](#eth_getRemoteCtx)
- [`eth_ctxStats`](#eth_ctxStats)
- [`eth_poolStats`](#eth_poolStats)

## JSON-RPC Endpoint

Default JSON-RPC endpoints:

| Client | URL                                             |
| ------ | ----------------------------------------------- |
| Go     | [http://localhost:8545](http://localhost:8545/) |

---

## eth_ctxQuery

通过TxHash跨链交易单。

**参数**

- `DATA`, 32个字节 - 交易的哈希

**返回**

- `value `: `QUANTITY`, 交易数额。
- `ctxId`: `DATA`, 32个字节- 该交易ID。
- `txHash`: `DATA` 32个字节- 交易的哈希。
- `from`: `DATA`, 20个字节 -交易发送者的地址
- `blockHash`: `DATA`, 32个字节- 该交易所在的区块的哈希。
- `destinationId`: `QUANTITY` - 跨链目的链ID。
- `destinationValue`: `QUANTITY`, 目的链币种数额
- `input`: `DATA`, 交易的input数据。
- `v`: `Array` - 交易多签V值数组。
- `r`: `Array` - 交易多签R值数组。
- `s`: `Array` - 交易多签S值数组。

**示例**

```javascript
//request
curl localhost:8545 -X POST -H "Content-Type:application/json" -d '{"jsonrpc":"2.0","method":"eth_ctxQuery","params":["0xd99bd3a1d9b79a2c5564abf698b6da2dcb478d42e97ceb161d31d043fa86b842"],"id":67}'

//response
{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "value": "0xde0b6b3a7640000",
        "ctxId": "0x7bf717aba73c062b9ffc8a8afb78db1cf0e05be1d2728f4fa6b1fd5225a24194",
        "txHash": "0xd99bd3a1d9b79a2c5564abf698b6da2dcb478d42e97ceb161d31d043fa86b842",
        "from": "0x3db32cdacb1ba339786403b50568f4915892938a",
        "blockHash": "0xf72950c97f05a563aa54af8d6e84e7ae7b3a40c99cc100102658a69c7190808b",
        "destinationId": "0x57f0",
        "destinationValue": "0xde0b6b3a7640000",
        "input": "0x",
        "v": [
            "0x1de",
            "0x1de"
        ],
        "r": [
            "0xcfc9d48d8c0425329245c1768fb42140584ad7afa71c4d05c755bce77970c6d",
            "0x56e5a3205ceefe7e688adcab2f3c71f5245b4d1f2ae0465bd15a462a045b6dcc"
        ],
        "s": [
            "0x5e25bcc98cbc23b86250a974b911c7dc5d628365b85855a50873be7c26a07be3",
            "0x1a3726f533b935183599c7480df03bbec6ced583330a8aedfb5ac50ab9e7f7a5"
        ]
    }
}
```
------

## eth_ctxOwner

通过跨链发起人查询交易列表。

**参数**

- `ADDRESS`, 20个字节 - 交易的发起地址

**返回**

- `local`: 本地跨链列表
- `time`: 跨链交易所在区块的时间戳
- `value `: `QUANTITY`, 交易数额。
- `ctxId`: `DATA`, 32个字节- 该交易ID。
- `txHash`: `DATA` 32个字节- 交易的哈希。
- `from`: `DATA`, 20个字节 -交易发送者的地址
- `blockHash`: `DATA`, 32个字节- 该交易所在的区块的哈希。
- `destinationId`: `QUANTITY` - 跨链目的链ID。
- `destinationValue`: `QUANTITY`, 目的链币种数额
- `input`: `DATA`, 交易的input数据。
- `v`: `Array` - 交易多签V值数组。
- `r`: `Array` - 交易多签R值数组。
- `s`: `Array` - 交易多签S值数组。

**示例**

```javascript
//Request
curl localhost:8545 -X POST -H "Content-Type:application/json" -d '{"jsonrpc":"2.0","method":"eth_ctxOwner","params":["0x3db32cdacb1ba339786403b50568f4915892938a"],"id":67}'

//Response
{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "local": {
            "221": [
                {
                    "value": "0xde0b6b3a7640000",
                    "status": 0,
                    "ctxId": "0x7bf717aba73c062b9ffc8a8afb78db1cf0e05be1d2728f4fa6b1fd5225a24194",
                    "txHash": "0xd99bd3a1d9b79a2c5564abf698b6da2dcb478d42e97ceb161d31d043fa86b842",
                    "from": "0x3db32cdacb1ba339786403b50568f4915892938a",
                    "blockHash": "0xf72950c97f05a563aa54af8d6e84e7ae7b3a40c99cc100102658a69c7190808b",
                    "destinationId": "0x57f0",
                    "destinationValue": "0xde0b6b3a7640000",
                    "input": "0x",
                    "time": "0x5ebb9cd2",
                    "v": [
                        "0x1de",
                        "0x1de"
                    ],
                    "r": [
                        "0xcfc9d48d8c0425329245c1768fb42140584ad7afa71c4d05c755bce77970c6d",
                        "0x56e5a3205ceefe7e688adcab2f3c71f5245b4d1f2ae0465bd15a462a045b6dcc"
                    ],
                    "s": [
                        "0x5e25bcc98cbc23b86250a974b911c7dc5d628365b85855a50873be7c26a07be3",
                        "0x1a3726f533b935183599c7480df03bbec6ced583330a8aedfb5ac50ab9e7f7a5"
                    ]
                }
            ]
        }
    }
}
```
------

## eth_ctxOwnerByPage

通过跨链发起人查询交易列表。

**参数**

- `ADDRESS`, 20个字节 - 交易的发起地址
- `PAGESIZE`, 每页的交易数量
- `STARTPAGE`, 从第几页开始(初始页为第0页)

**返回**

- `data`:`OBJECT` - 本地跨链列表:
- `time`: 跨链交易所在区块的时间戳
- `value `: `QUANTITY`, 交易数额。
- `ctxId`: `DATA`, 32个字节- 该交易ID。
- `txHash`: `DATA` 32个字节- 交易的哈希。
- `from`: `DATA`, 20个字节 -交易发送者的地址
- `blockHash`: `DATA`, 32个字节- 该交易所在的区块的哈希。
- `destinationId`: `QUANTITY` - 跨链目的链ID。
- `destinationValue`: `QUANTITY`, 目的链币种数额
- `input`: `DATA`, 交易的input数据。
- `v`: `Array` - 交易多签V值数组。
- `r`: `Array` - 交易多签R值数组。
- `s`: `Array` - 交易多签S值数组。

`total`:`QUANTITY` - 总交易数量

##### 示例

```json
//Request
curl localhost:8545 -X POST -H "Content-Type:application/json" -d '{"jsonrpc":"2.0","method":"eth_ctxOwner","params":["0x3db32cdacb1ba339786403b50568f4915892938a"],"id":67}'

//Respinse
{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "data": {
            "221": [
                {
                    "value": "0xde0b6b3a7640000",
                    "status": 0,
                    "ctxId": "0x7bf717aba73c062b9ffc8a8afb78db1cf0e05be1d2728f4fa6b1fd5225a24194",
                    "txHash": "0xd99bd3a1d9b79a2c5564abf698b6da2dcb478d42e97ceb161d31d043fa86b842",
                    "from": "0x3db32cdacb1ba339786403b50568f4915892938a",
                    "blockHash": "0xf72950c97f05a563aa54af8d6e84e7ae7b3a40c99cc100102658a69c7190808b",
                    "destinationId": "0x57f0",
                    "destinationValue": "0xde0b6b3a7640000",
                    "input": "0x",
                    "time": "0x5ebb9cd2",
                    "v": [
                        "0x1de",
                        "0x1de"
                    ],
                    "r": [
                        "0xcfc9d48d8c0425329245c1768fb42140584ad7afa71c4d05c755bce77970c6d",
                        "0x56e5a3205ceefe7e688adcab2f3c71f5245b4d1f2ae0465bd15a462a045b6dcc"
                    ],
                    "s": [
                        "0x5e25bcc98cbc23b86250a974b911c7dc5d628365b85855a50873be7c26a07be3",
                        "0x1a3726f533b935183599c7480df03bbec6ced583330a8aedfb5ac50ab9e7f7a5"
                    ]
                }
            ]
        },
        "total": 1
    }
}
```
------

## eth_ctxContent

返回当前所有跨链交易单。

**参数**

- `无`

**返回**

- `local` - 本地链跨链交易列表
- `remote` - 跨链目的链交易列表

**示例**

```json
//Request
curl localhost:8545 -X POST -H "Content-Type:application/json" -d '{"jsonrpc":"2.0","method":"eth_ctxContent","params":[],"id":67}'

//Response
{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "local": {
            "22512": [
                {
                    "value": "0xde0b6b3a7640000",
                    "ctxId": "0x7bf717aba73c062b9ffc8a8afb78db1cf0e05be1d2728f4fa6b1fd5225a24194",
                    "txHash": "0xd99bd3a1d9b79a2c5564abf698b6da2dcb478d42e97ceb161d31d043fa86b842",
                    "from": "0x3db32cdacb1ba339786403b50568f4915892938a",
                    "blockHash": "0xf72950c97f05a563aa54af8d6e84e7ae7b3a40c99cc100102658a69c7190808b",
                    "destinationId": "0x57f0",
                    "destinationValue": "0xde0b6b3a7640000",
                    "input": "0x",
                    "v": [
                        "0x1de",
                        "0x1de"
                    ],
                    "r": [
                        "0xcfc9d48d8c0425329245c1768fb42140584ad7afa71c4d05c755bce77970c6d",
                        "0x56e5a3205ceefe7e688adcab2f3c71f5245b4d1f2ae0465bd15a462a045b6dcc"
                    ],
                    "s": [
                        "0x5e25bcc98cbc23b86250a974b911c7dc5d628365b85855a50873be7c26a07be3",
                        "0x1a3726f533b935183599c7480df03bbec6ced583330a8aedfb5ac50ab9e7f7a5"
                    ]
                }
            ]
        },
        "remote": {
            "22512": [
                {
                    "value": "0xde0b6b3a7640000",
                    "ctxId": "0xf9747951a56fb6392d1c2bddedf5f9cb0f380a43ac7e6345aa2fea73646e14d6",
                    "txHash": "0x43f6150c0c308d2b5325f4f6f5820f00862a14beb2e961780eec518b4f6497c1",
                    "from": "0x8029fcfc954ff7be80afd4db9f77f18c8aa1ecbc",
                    "blockHash": "0x5391c792a14a3ec6be49201dbbf6a023fbb03025e0c600990107276dc3574ff5",
                    "destinationId": "0xdd",
                    "destinationValue": "0xde0b6b3a7640000",
                    "input": "0x",
                    "v": [
                        "0xb003",
                        "0xb003"
                    ],
                    "r": [
                        "0xae8928d14f2d6389803e29d94be0c1a7ef517818cf1a8a2305b282fb9f7cbf68",
                        "0xb7d137e78069d4d2ad39940a4d2a82363fbff32920125853c5feef0b87e91cb4"
                    ],
                    "s": [
                        "0x7ab9cd7f753f4fc1c89a66df1ef8cf6d3c37aaa678164bb4df3fed8b19fc234f",
                        "0x6d36ffed2dcf26278518def89a92e01b74f81e2ff12ecb29b0fc2b49db70566b"
                    ]
                }
            ]
        }
    }
}
```
------

## eth_ctxContentByPage

分页返回当前所有跨链交易单。

**参数**

- `LOCALSIZE`, - 本地跨链列表每页条数。
- `LOCALPAGE`, - 本地跨链列表从第几页开始(初始页为第0页)。
- `REMOTESIZE`, - 远端跨链列表每页条数。
- `REMOTEPAGE`, - 远端跨链列表从第几页开始(初始页为第0页)。

**返回**

- `local` - 本地链跨链交易列表
- `remote` - 跨链目的链交易列表

**示例**

```json
//Request
curl localhost:8545 -X POST -H "Content-Type:application/json" -d '{"jsonrpc":"2.0","method":"eth_ctxContentByPage","params":[1,0,1,0],"id":67}'

//Response
{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "local": {
            "data": {
                "22512": [
                    {
                        "value": "0xde0b6b3a7640000",
                        "ctxId": "0x7bf717aba73c062b9ffc8a8afb78db1cf0e05be1d2728f4fa6b1fd5225a24194",
                        "txHash": "0xd99bd3a1d9b79a2c5564abf698b6da2dcb478d42e97ceb161d31d043fa86b842",
                        "from": "0x3db32cdacb1ba339786403b50568f4915892938a",
                        "blockHash": "0xf72950c97f05a563aa54af8d6e84e7ae7b3a40c99cc100102658a69c7190808b",
                        "destinationId": "0x57f0",
                        "destinationValue": "0xde0b6b3a7640000",
                        "input": "0x",
                        "v": [
                            "0x1de",
                            "0x1de"
                        ],
                        "r": [
                            "0xcfc9d48d8c0425329245c1768fb42140584ad7afa71c4d05c755bce77970c6d",
                            "0x56e5a3205ceefe7e688adcab2f3c71f5245b4d1f2ae0465bd15a462a045b6dcc"
                        ],
                        "s": [
                            "0x5e25bcc98cbc23b86250a974b911c7dc5d628365b85855a50873be7c26a07be3",
                            "0x1a3726f533b935183599c7480df03bbec6ced583330a8aedfb5ac50ab9e7f7a5"
                        ]
                    }
                ]
            },
            "total": 1
        },
        "remote": {
            "data": {
                "22512": [
                    {
                        "value": "0xde0b6b3a7640000",
                        "ctxId": "0xf9747951a56fb6392d1c2bddedf5f9cb0f380a43ac7e6345aa2fea73646e14d6",
                        "txHash": "0x43f6150c0c308d2b5325f4f6f5820f00862a14beb2e961780eec518b4f6497c1",
                        "from": "0x8029fcfc954ff7be80afd4db9f77f18c8aa1ecbc",
                        "blockHash": "0x5391c792a14a3ec6be49201dbbf6a023fbb03025e0c600990107276dc3574ff5",
                        "destinationId": "0xdd",
                        "destinationValue": "0xde0b6b3a7640000",
                        "input": "0x",
                        "v": [
                            "0xb003",
                            "0xb003"
                        ],
                        "r": [
                            "0xae8928d14f2d6389803e29d94be0c1a7ef517818cf1a8a2305b282fb9f7cbf68",
                            "0xb7d137e78069d4d2ad39940a4d2a82363fbff32920125853c5feef0b87e91cb4"
                        ],
                        "s": [
                            "0x7ab9cd7f753f4fc1c89a66df1ef8cf6d3c37aaa678164bb4df3fed8b19fc234f",
                            "0x6d36ffed2dcf26278518def89a92e01b74f81e2ff12ecb29b0fc2b49db70566b"
                        ]
                    }
                ]
            },
            "total": 2
        }
    }
}
```

------

## eth_getLocalCtx

分页获取本地跨链交易列表。

**参数**

- `PAGESIZE`, 每页的交易数量
- `STARTPAGE`, 从第几页开始(初始页为第0页)

**返回**

`data`:`OBJECT` - 本地跨链列表:

- `value `: `QUANTITY`, 交易数额。
- `ctxId`: `DATA`, 32个字节- 该交易ID。
- `txHash`: `DATA` 32个字节- 交易的哈希。
- `from`: `DATA`, 20个字节 -交易发送者的地址
- `blockHash`: `DATA`, 32个字节- 该交易所在的区块的哈希。
- `destinationId`: `QUANTITY` - 跨链目的链ID。
- `destinationValue`: `QUANTITY`, 目的链币种数额
- `input`: `DATA`, 交易的input数据。
- `v`: `Array` - 交易多签V值数组。
- `r`: `Array` - 交易多签R值数组。
- `s`: `Array` - 交易多签S值数组。
`total`:`QUANTITY` - 总交易数量

##### 示例

```json
//Request
curl localhost:8545 -X POST -H "Content-Type:application/json" -d '{"jsonrpc":"2.0","method":"eth_getLocalCtx","params":[1,0],"id":67}'

//Response
{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "data": {
            "22512": [
                {
                    "value": "0xde0b6b3a7640000",
                    "ctxId": "0x7bf717aba73c062b9ffc8a8afb78db1cf0e05be1d2728f4fa6b1fd5225a24194",
                    "txHash": "0xd99bd3a1d9b79a2c5564abf698b6da2dcb478d42e97ceb161d31d043fa86b842",
                    "from": "0x3db32cdacb1ba339786403b50568f4915892938a",
                    "blockHash": "0xf72950c97f05a563aa54af8d6e84e7ae7b3a40c99cc100102658a69c7190808b",
                    "destinationId": "0x57f0",
                    "destinationValue": "0xde0b6b3a7640000",
                    "input": "0x",
                    "v": [
                        "0x1de",
                        "0x1de"
                    ],
                    "r": [
                        "0xcfc9d48d8c0425329245c1768fb42140584ad7afa71c4d05c755bce77970c6d",
                        "0x56e5a3205ceefe7e688adcab2f3c71f5245b4d1f2ae0465bd15a462a045b6dcc"
                    ],
                    "s": [
                        "0x5e25bcc98cbc23b86250a974b911c7dc5d628365b85855a50873be7c26a07be3",
                        "0x1a3726f533b935183599c7480df03bbec6ced583330a8aedfb5ac50ab9e7f7a5"
                    ]
                }
            ]
        },
        "total": 1
    }
}
```
------

## eth_getRemoteCtx

分页获取本地跨链交易列表。

**参数**

- `PAGESIZE`, 每页的交易数量
- `STARTPAGE`, 从第几页开始(初始页为第0页)


**返回**

- `data`:`OBJECT` - 本地跨链列表:
- `value `: `QUANTITY`, 交易数额。
- `ctxId`: `DATA`, 32个字节- 该交易ID。
- `txHash`: `DATA` 32个字节- 交易的哈希。
- `from`: `DATA`, 20个字节 -交易发送者的地址
- `blockHash`: `DATA`, 32个字节- 该交易所在的区块的哈希。
- `destinationId`: `QUANTITY` - 跨链目的链ID。
- `destinationValue`: `QUANTITY`, 目的链币种数额
- `input`: `DATA`, 交易的input数据。
- `v`: `Array` - 交易多签V值数组。
- `r`: `Array` - 交易多签R值数组。
- `s`: `Array` - 交易多签S值数组。
- `total`:`QUANTITY` - 总交易数量

**示例**

```json
//Request
curl localhost:8545 -X POST -H "Content-Type:application/json" -d '{"jsonrpc":"2.0","method":"eth_getRemoteCtx","params":[1,0],"id":67}'

//Response
{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "data": {
            "22512": [
                {
                    "value": "0xde0b6b3a7640000",
                    "ctxId": "0xf9747951a56fb6392d1c2bddedf5f9cb0f380a43ac7e6345aa2fea73646e14d6",
                    "txHash": "0x43f6150c0c308d2b5325f4f6f5820f00862a14beb2e961780eec518b4f6497c1",
                    "from": "0x8029fcfc954ff7be80afd4db9f77f18c8aa1ecbc",
                    "blockHash": "0x5391c792a14a3ec6be49201dbbf6a023fbb03025e0c600990107276dc3574ff5",
                    "destinationId": "0xdd",
                    "destinationValue": "0xde0b6b3a7640000",
                    "input": "0x",
                    "v": [
                        "0xb003",
                        "0xb003"
                    ],
                    "r": [
                        "0xae8928d14f2d6389803e29d94be0c1a7ef517818cf1a8a2305b282fb9f7cbf68",
                        "0xb7d137e78069d4d2ad39940a4d2a82363fbff32920125853c5feef0b87e91cb4"
                    ],
                    "s": [
                        "0x7ab9cd7f753f4fc1c89a66df1ef8cf6d3c37aaa678164bb4df3fed8b19fc234f",
                        "0x6d36ffed2dcf26278518def89a92e01b74f81e2ff12ecb29b0fc2b49db70566b"
                    ]
                }
            ]
        },
        "total": 2
    }
}
```
------

## eth_ctxStats

获取跨链交易条数。

**参数**

- `无`

**返回**

- `local`: - 本地跨链条数:
- `remote`: - 远程链跨链条数:

**示例**

```json
//Request
curl localhost:8545 -X POST -H "Content-Type:application/json" -d '{"jsonrpc":"2.0","method":"eth_ctxStats","params":[],"id":67}'

//Response
{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "local": 1,
        "remote": 2
    }
}
```
------

## eth_poolStats

获取跨链待确认交易池交易条数。

**参数**

- `无`

**返回**

- `pending`: - 本地已确认跨链条数:
- `queue`: - 远程同步跨链条数:

**示例**

```json
//Request
curl localhost:8545 -X POST -H "Content-Type:application/json" -d '{"jsonrpc":"2.0","method":"eth_poolStats","params":[],"id":67}'

//Response
{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "pending": 0,
        "queue": 0
    }
}
```
------