[SimpleChain](https://www.simplechain.com/)的基础配置服务于链，启动[SimpleChain](https://www.simplechain.com/)节点，则需要将链配置载入。 因此，在[SimpleChain](https://www.simplechain.com/)中内置了主网(mainnet)、测试网(testnet)配置。

初始启动节点时，将根据不同的参数（–dev、–testnet）来默认加载不同链配置。

## 链配置

不同于传统软件，因为区块链的不可篡改性，要求对同一个区块，不管出块时的软件版本，还是n年后的软件版本。都需要保证软件对已出块区块做出相同操作。因此区块链的链配置，不得随意更改，还需要维护重要历史变更内容。


下面是链的核心配置信息，定义在 params/config.go 中：

```go
// ChainConfig is the core config which determines the blockchain settings.
// ChainConfig is stored in the database on a per block basis. This means
// that any network, identified by its genesis block, can have its own
// set of configuration options.
type ChainConfig struct {
	ChainID *big.Int `json:"chainId"` // chainId identifies the current chain and is used for replay protection

	SingularityBlock *big.Int `json:"singularityBlock,omitempty"` // Singularity switch block (nil = no fork, 0 = already on singularity)
	EWASMBlock       *big.Int `json:"ewasmBlock,omitempty"`       // EWASM switch block (nil = no fork, 0 = already activated)

	// Various consensus engines
	Ethash   *EthashConfig   `json:"ethash,omitempty"`
	Clique   *CliqueConfig   `json:"clique,omitempty"`
	Scrypt   *ScryptConfig   `json:"scrypt,omitempty"`
	DPoS     *DPoSConfig     `json:"dpos,omitempty"`
	Raft     bool            `json:"raft,omitempty"`
	Istanbul *IstanbulConfig `json:"istanbul,omitempty"`
}
```

区块链的不可篡改性，非中心化程序使得区块链网络程序升级复杂化。从Simplechain的核心配置，可折射出整个Simplechain网络所经历的关键时刻。 

如上的[SimpleChain](https://www.simplechain.com/)链配置，并非程序起初编写，而是随[SimpleChain](https://www.simplechain.com/)发展，在共识协议重大变更时积累而成。 下面是各项配置的作用说明：

### ChainID

`ChianID`是当前链标识符，用于防止重放攻击。

### SingularityBlock

硬分叉高度。意味着从此高度开始，新区块受新版本共识规则约束。因涉及共识变更，如果希望继续接受新区块则必须升级Simplechain程序，属于区块链硬分叉。
如果不愿意接受共识变更，则可以独立使用新的 ChainID 继续原共识，且必须独立维护版本。
  
### EWASMBlock



### Ethash

`Ethash`共识算法引擎配置，Ethash是以太坊的共识算法，它是一种PoW共识算法。可作为Simplechain子链的一种共识算法。

### Clique

`Clique` POA共识算法的共识引擎配置,PoA共识算法也是Simplechain可以选择的共识算法之一，在搭建测试链和私有链的时候可以选择该共识算法。

### Scrypt

`Scrypt`共识算法引擎配置，Scrypt是Simplechain主链的共识算法，该算法属于PoW共识算法。

### DPoS

`DPoS`共识算法引擎配置, DPoS是Simplechain子链的一种共识算法，因为Simplechain是一主多子的区块链架构，所以子链可以选择多种共识算法，DPos就是子链可以选择的一种共识算法。

### Raft

`Raft`共识算法引擎配置，Raft是Simplechain子链可以选择的一种共识算法,Raft通过选举一个高贵的领导人，然后给予他全部的管理复制日志的责任来实现一致性。

### Istanbul

`Istanbul`Simplechain的共识引擎配置


