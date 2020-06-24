创世区块作为第零个区块，其他区块直接或间接引用到创世区块。
因此节点启动之初必须载入正确的创世区块信息，且不得任意修改。

[SimpleChain](https://www.simplechain.com/)允许通过创世配置文件来初始化创世区块，也可使用选择使用内置的多个网络环境的创世配置。
默认使用[SimpleChain](https://www.simplechain.com/)主网创世配置。

## 创世配置文件

如果你需要搭建[SimpleChain](https://www.simplechain.com/)私有链，那么了解创世配置是必须的，否则你大可不关心创世配置。

下面是一份 JSON 格式的创世配置示例：

```json    
{
"config": {
    "chainId": 100, 
    "homesteadBlock": 0, 
    "eip155Block": 0, 
    "eip158Block": 0
},
"alloc" : {},
"coinbase" : "0x0000000000000000000000000000000000000000",
"difficulty" : "0x20000",
"extraData" : "",
"gasLimit" : "0x2fefd8",
"nonce" : "0x0000000000000042",
"mixhash" : "0x0000000000000000000000000000000000000000000000000000000000000000", "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000", "timestamp" : "0x00"
}
```

根据配置用途可分为三大类：

1.链配置

+ `config`项是定义链配置，会影响共识协议，虽然链配置对创世影响不大，但新区块的出块规则均依赖链配置。

2.创世区块头信息配置  

+ `nonce`：随机数，对应创世区块 `Nonce` 字段。
+ `timestamp`：UTC时间戳，对应创世区块 `Time`字段。
+ `extraData`：额外数据，对应创世区块 `Extra` 字段。
+ `gasLimit`：必填，燃料上限，对应创世区块 `GasLimit` 字段。
+ `difficulty`：必填，难度系数，对应创世区块 `Difficulty` 字段。搭建私有链时，需要根据情况选择合适的难度值，以便调整出块。
+ `mixHash`：一个哈希值，对应创世区块的`MixDigest`字段。和 nonce 值一起证明在区块上已经进行了足够的计算。
+ `coinbase`：一个地址，对应创世区块的`Coinbase`字段。

3.初始账户资产配置

+ `alloc` 项是创世中初始账户资产配置。在生成创世区块时，将此数据集中的账户资产写入区块中，相当于预挖矿。
这对开发测试和私有链非常好用，不需要挖矿就可以直接为任意多个账户分配资产。

## 创世区块加载流程

在运行 Simplechain 时需根据配置文件加载创世配置以及创世区块，并校验其合法性。
如果配置信息随意变更，易引起共识校验不通过等问题。只有在加载并检查通过时，才能继续运行程序。

![创世加载流程](1.png)

上图是一个简要流程，下面分别讲解“加载创世配置”和“安装创世区块”两个子流程。

## 加载创世配置

应使用哪种创世配置，由用户在启动 Sipe 时决定。下图是创世配置选择流程图：

![f6.png](2.png)

通过 Sipe 命令参数可选择不同网络配置，可以通过 `networkid` 选择，也可使用网络名称启用。

1.使用 networkid, 不同网络使用不同ID标识。

-  1=Frontier，主网环境，是默认选项。
-  2=Morden 测试网络，但已禁用。
-  3=Ropsten 测试网络。
-  4=Rinkeby 测试网络。

2.直接使用网络名称：

-  testnet: Ropsten 测试网络。
-  rinkeby: Rinkeby 测试网络。
-  goerli: Görli 测试网络。
-  dev: 本地开发环境。

Sipe 启动时根据不同参数选择加载不同网络配置，并对应不同网络环境。如果不做任何选择，虽然在此不会做出选择，但在后面流程中会默认使用主网配置。

## 安装创世区块

首先，需要从数据库中根据区块高度 0 读取创世区块哈希。
如果不存在则说明本地属于第一次启动，直接使用运行时创世配置来构建创世区块。
属于首次，还需要存储创世区块和链配置。

如果存在，则需要使用运行时创世配置构建创世区块并和本次已存储的创世区块哈希进行对比。
一旦不一致，则返回错误，不得继续。

随后，还需要检查链配置。先从数据库获取链配置，如果不存在，则无需校验直接使用运行时链配置。
否则，需要检查运行时链配置是否正确，只有正确时才能替换更新。
但有一个例外：主网配置不得随意更改，由代码控制而非人为指定。

总的来说，Simplechain默认使用主网配置，只有在首次运行时才创建和存储创世区块，其他时候仅仅用于校验。
而链配置除主网外则在规则下可随时变更。

## 构建创建区块

上面我们已知晓总体流程，这里再细说下Simplechain是如何根据创世配置生成创世区块。
核心代码位于 `core/genesis.go:229`

```go
func (g *Genesis) ToBlock(db ethdb.Database) *types.Block{
    if db == nil {
        db = rawdb.NewMemoryDatabase()
    }
    statedb, _ := state.New(common.Hash{}, state.NewDatabase(db))//1
    for addr, account := range g.Alloc { //2
        statedb.AddBalance(addr, account.Balance)
        statedb.SetCode(addr, account.Code)
        statedb.SetNonce(addr, account.Nonce)
        for key, value := range account.Storage {
            statedb.SetState(addr, key, value)
        }
    }
    root := statedb.IntermediateRoot(false)//3
    head := &types.Header{//4
        Number:     new(big.Int).SetUint64(g.Number),
        Nonce:      types.EncodeNonce(g.Nonce),
        Time:       g.Timestamp,
        ParentHash: g.ParentHash,
        Extra:      g.ExtraData,
        GasLimit:   g.GasLimit,
        GasUsed:    g.GasUsed,
        Difficulty: g.Difficulty,
        MixDigest:  g.Mixhash,
        Coinbase:   g.Coinbase,
        Root:       root,
    }
    //5
    if g.GasLimit == 0 {
        head.GasLimit = params.GenesisGasLimit
    }
    if g.Difficulty == nil {
        head.Difficulty = params.GenesisDifficulty
    }

    statedb.Commit(false)//6
    statedb.Database().TrieDB().Commit(root, true)//7

    return types.NewBlock(head, nil, nil, nil)//8
}
```

上面代码是根据创世配置生成创世区块的代码逻辑，细节如下：

1. 创世区块无父块，从零初始化全新的 `state`（后续文章会详细讲解 `state`对象）。
2. 遍历配置中 `Alloc` 项账户集合数据，直接写入 state 中。
    这里不单可以设置 `balance`，还可以设置 `code`、`nonce` 以及任意多个 `storage` 数据。
    意味着创世时便可以直接部署智能合约。例如下面配置则在创世时部署了一个名为`093f59f1d91017d30d8c2caa78feb5beb0d2cfaf` 的智能合约。

```json
"alloc": {
    "093f59f1d91017d30d8c2caa78feb5beb0d2cfaf": {
        "balance": "0xffffffffffffffff",
        "nonce": "0x3",
        "code":"0x606060",
        "storage":{
            "11bbe8db4e347b4e8c937c1c8370e4b5ed33adb3db69cbdb7a38e1e50b1b82fa":"1234ff"
        }
    }
}
```

3. 将账户数据写入 state 后，便可以计算出 state 数据的默克尔树的根值，称之为 `StateRoot`。
    此值记录在区块头 `Root` 字段中。
4. 创世配置的一部分配置，则直接映射到区块头中，完成创世区块头的构建。
5. 因为 `GasLimit` 和 `Difficulty` 直接影响到下一个区块出块处理。
    因此未设置时使用默认配置(Difficulty=131072，GasLimit=4712388)。
6. 提交 state，将 state 数据提交到底层的内存 trie 数据中。
7. 将内存 trie 数据更新到 db 中。
这是多余的一步，因为提交到数据库是由外部进行，这里只需要负责生成区块。
8. 利用区块头创建区块，且区块中无交易记录。



