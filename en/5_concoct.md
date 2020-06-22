[SimpleChain](https://www.simplechain.com/)the basic configuration of the service chain, start[SimpleChain](https://www.simplechain.com/) node, you need to load the chain configuration. Therefore, in[SimpleChain](https://www.simplechain.com/) the main network (mainnet) and test network (testnet) configurations are built in.

When the node is initially started, different chain configurations are loaded by default according to different parameters (-dev,-testnet).

## Chain Configuration

Different from traditional software, because of the non-tamper nature of the blockchain, it requires the same block, regardless of the software version when the block is released or the software version n years later. All must ensure that the software does the same operation on the out-of-block block. Therefore, the blockchain configuration of the blockchain cannot be changed at will, and important historical changes need to be maintained.

The following is the core configuration information of the chain, which is defined in params/config.go:

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

The blockchain cannot be tampered with. Non-centralized programs make the upgrade of blockchain network programs more complicated. The core configuration of Simplechain reflects the critical moments of the entire Simplechain network. 

As above[SimpleChain](https://www.simplechain.com/)Chain configuration, not the program is initially written, but [SimpleChain](https://www.simplechain.com/) development is accumulated during major changes in consensus agreements. The following is a description of the role of each configuration:

### ChainID

`ChianID`is the identifier of the current chain to prevent replay attacks.

### SingularityBlock

Hard Fork height. This means that from this height, the new District block is restricted by the new version of consensus rules. Because consensus changes are involved, if you want to continue accepting new blocks, you must upgrade the Simplechain program, which belongs to the blockchain hard fork.

If you do not want to accept consensus changes, you can use the new ChainID to continue the original consensus independently, and the version must be maintained independently.
  
### EWASMBlock



### Ethash

`Ethash`the consensus algorithm engine configuration. Ethash is the consensus algorithm of Ethereum. It is a PoW consensus algorithm. It can be used as a consensus algorithm for Simplechain sub-chains.

### Clique

`Clique` POA consensus algorithm is configured as a consensus engine. PoA consensus algorithm is also one of the consensus algorithms that Simplechain can choose. You can choose this consensus algorithm when building a test chain and a private chain.

### Scrypt

`Scrypt`the consensus algorithm engine configuration. Scrypt is the consensus algorithm of the Simplechain main chain, which belongs to the PoW consensus algorithm.

### DPoS

`DPoS` is a consensus algorithm of Simplechain sub-chains. Because Simplechain is a blockchain architecture with one master and multiple sub-chains, multiple consensus algorithms can be selected for sub-chains, DPos is a consensus algorithm that can be selected for sub-chains.

### Raft

`Raft`is a consensus algorithm that you can choose from the Simplechain sub-chain. Raft selects a noble leader and gives him all the responsibilities to manage and copy logs to achieve consistency.性。

### Istanbul

`Istanbul`Simplechain consensus engine configuration


