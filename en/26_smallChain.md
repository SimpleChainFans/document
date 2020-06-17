## Deploy DPOS consensus sub-chain network

### 1. Genesis block

```json
{
  "config": {
    "chainId": 10388,
    "dpos": {
      "period": 3,
      "epoch": 300,
      "maxSignersCount": 21,
      "minVoterBalance": 100000000000000000000,
      "genesisTimestamp": 1554004800,
      "signers": [
        "3d50e12fa9c76e4e517cd4ace1b36c453e6a9bcd",
        "f97df7fe5e064a9fe4b996141c2d9fb8a3e2b53e",
        "ef90068860527015097cd031bd2425cb90985a40"
      ],
      "pbft": false,
      "voterReward": true
    }
  },
  "nonce": "0x0",
  "timestamp": "0x5ca03b40",
  "extraData": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "gasLimit": "0x47b760",
  "difficulty": "0x1",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "alloc": {
    "3d50e12fa9c76e4e517cd4ace1b36c453e6a9bcd": {
      "balance": "0x21e19e0c9bab2400000"
    },
    "ef90068860527015097cd031bd2425cb90985a40": {
      "balance": "0x21e19e0c9bab2400000"
    },
    "f97df7fe5e064a9fe4b996141c2d9fb8a3e2b53e": {
      "balance": "0x21e19e0c9bab2400000"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

+ `period` dpos block-out interval, in seconds
+ `epoch` How many blocks are there in the dpos interval to regularly clear the votes (after clearing, the voters need to re-initiate the voting transaction)
+ `maxSignersCount` Maximum number of producers allowed for dpos
+ `minVoterBalance` The minimum amount of votes allowed by dpos. Unit: Wei
+ `voterReward` Whether dpos voters can get rewards (if enabled, voters can also get dividends when producers issue blocks)
+ `genesisTimestamp` dpos allows the initial block output time, and calculates the subsequent block output time and the producer based on this time.
+ `signers` dpos initial producer list
+ `pbft` Whether dpos uses pbft to confirm each block after each round of block output
+ `alloc` dpos initial producer mortgage vote amount

### 2. Sub-chain initialization process

#### Method 1. Use sipe to initialize

1.Create or import a producer account

```shell script
sipe --datadir=dposdata account new 
``` 
2.Write the created or imported producer address into genesis.json, and write the initial voting amount at the same time (refer to 1. genesis block)

3.Initialize sub-chain nodes
    
```shell script
sipe --datadir=dposdata --role=subchain init genesis.json
```
#### Method 2. Use the consensus tool to initialize the cluster with one click

in `cmd/consensus` run under directory`init_dpos.sh`

```shell script
cd cmd/consensus
./init_dpos.sh --numNodes 3
```
+ `numNodes` Number of cluster nodes generated

After initialization is completed `cmd/consensus/dposdata` create corresponding node files under the Directory

### 3. Sub-chain startup process

1. Startup node
   
```bash
sipe --datadir=dposdata --mine --etherbase=<生产者地址> --unlock=<生产者地址> --password=<密码文件> --port=30303  --role=subchain --v5disc
```
  
2. Connect to other nodes
   
```bash
sipe --datadir=dposdata --mine --etherbase=<生产者地址> --unlock=<生产者地址> --password=<密码文件> --port=30304  --role=subchain --v5disc --bootnodesv5={enode1} --bootnodesv4={enode1}
```

### 4. Votes and proposals

#### 4.1 initiate voting transaction

```bash
> eth.sendTransaction({from:"<投票地址>",to:"<被投票地址>",value:0,data:web3.toHex("dpos:1:event:vote")})
```
#### 4.2 initiate the cancellation of voting transaction

```bash
> eth.sendTransaction({from:"<投票地址>",to:"<投票地址>",value:0,data:web3.toHex("dpos:1:event:devote")})
``` 

#### 4.3 initiate a proposal to change miner rewards

Change the miner block reward ratio`666‰`

```bash
> eth.sendTransaction({from:"<提案地址>",to:"<提案地址>",value:0,data:web3.toHex("dpos:1:event:proposal:proposal_type:3:mrpt:666")})
```

#### 4.4 initiate a proposal to change the minimum allowed vote limit

+ Change the minimum allowed vote limit `10` ether

```bash
> eth.sendTransaction({from:"<提案地址>",to:"<提案地址>",value:0,data:web3.toHex("dpos:1:event:proposal:proposal_type:6:mvb:10")})
```

#### 4.5 pass or oppose the proposal

+ `yes` through the proposal,`no` objection proposal

```bash
> eth.sendTransaction({from:"<投票地址>",to:"<投票地址>",value:0,data:web3.toHex("dpos:1:event:declare:hash:<提案hash值>:decision:yes")})
```

    
### 5. View consensus status

```bash
> dpos.getSnapshot()
```
+ `candidates` Miner candidate list
+ `confirmedNumber` Confirmed block height
+ `historyHash` The block hash of the last two rounds of block output, which is used to calculate the block output order of the producer in the new round.
+ `minerReward` The reward percentage of each block producer, if enabled `voterReward` , the rest is the reward of voters
+ `signers` List producers and block order
+ `punished` List the punishment information of each producer for not releasing blocks on time
+ `tally` List the total votes of each candidate
+ `votes` List voting information
+ `voters` The height of the block where the voters vote
+ `proposals` Proposal list

## Deploy PBFT consensus sub-chain network

### 1. Genesis block

```json
{
  "config": {
    "chainId": 10388,
    "istanbul": {
      "epoch": 30000,
      "policy": 0
    }
  },
  "nonce": "0x0",
  "timestamp": "0x0",
  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000f843f83f941c46d10e91eafaac430718df3658b1a496b827bd94b67ee9395542b227c99941eb4168e3f3c6502dd8949d6510b637970085962c908c69e63e9d36a36cb480c0",
  "gasLimit": "0xe0000000",
  "difficulty": "0x1",
  "mixHash": "0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "alloc": {},
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```
+ `epoch` How many blocks are there at pbft intervals to regularly clear votes
+ `policy` The polling method of pbft proposer: 0 is roundRobin (replaced in order), and 1 is sticky (if the proposer is not wrong, do not replace the proposer)
+ `mixHash` pbft block shall mixhash specified as 0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365
+ `extraData` header.extra calculated by the initial pbft producer
+ `alloc` pbft does not have a block reward currently, so tokens need to be allocated in advance.

### 2. Sub-chain initialization process

#### Method 1. Use sipe to initialize

1.Create or import a producer account

```shell script
sipe --datadir=pbftdata account new 
``` 
   
2.Use the consensus tool to generate extraData and write it to genesis.json (refer to 1. genesis block)

```shell script
cd cmd/consensus
./init_pbft.sh --numNodes 1 --validator <生产者地址> 
```
3.Initialize sub-chain nodes

```shell script
sipe --datadir=pbftdata --role=subchain init genesis.json
```
   
4.Write the node's nodekey to pbftdata/static-nodes.json (the nodekey public key is the producer public key)

#### Method 2. Use the consensus tool to initialize the cluster with one click

Run init_pbft.sh in the cmd/consensus Directory

```bash
 cd cmd/consensus
 ./init_pbft.sh --numNodes 3 --ip 127.0.0.1 127.0.0.2 127.0.0.3 --port 21001 21002 21003
```
+ `numNodes` Number of cluster nodes generated
+ `ip` List of ip addresses of the specified node (the default ip address is 127.0.0.1)
+ `port` The list of ports for the specified node. The default port is 21001 ~ 2100x, and x is numNodes.

After initialization is completed`cmd/consensus/pbftdata`Create corresponding node files under the Directory

### 3. Sub-chain startup process

```bash
sipe --datadir=pbftdata --istanbul.requesttimeout=10000 --istanbul.blockperiod=5 --syncmode=full --mine --minerthreads=1 --port=21001 --role=subchain
```

+ `port` Must be consistent with the enode configured in static-nodes.json
+ `istanbul.requesttimeout` The expiration time of each view in milliseconds. The default value is 10000.
+ `istanbul.blockperiod` pbft block output interval, in seconds, the default value is 1

### 4.View consensus status

```shell script
> istanbul.getSnapshot()
```
+ `validators` pbft Block producer list
+ `votes` Votes for adding or removing validators
+ `tally` Total voting situation

## Deploy RAFT consensus sub-chain network

### 1. Genesis block

```json
{
  "config": {
    "chainId": 10,
    "raft": true
  },
  "nonce": "0x0",
  "timestamp": "0x0",
  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "gasLimit": "0xe0000000",
  "difficulty": "0x0",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "alloc": {
    "1e69ebb349e802e25c7eb3b41adb6d18a4ae8591": {
      "balance": "0x21e19e0c9bab2400000"
    },
    "73ce1d55593827ab5a680e750e347bf57485a511": {
      "balance": "0x21e19e0c9bab2400000"
    },
    "b8564a5657fa7dc51605b58f271b5bafad93b984": {
      "balance": "0x21e19e0c9bab2400000"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

+ `raft` true indicates the use of raft consensus
+ `alloc` The raft consensus packages blocks only when a transaction exists. Therefore, tokens need to be allocated in advance.

### 2. Sub-chain initialization process

#### Method 1. Use sipe to initialize

1.Create or import a producer account

```bash
sipe --datadir=raftdata account new 
``` 
   
2.Initialize sub-chain nodes
    
```bash
sipe --datadir=raftdata --role=subchain init genesis.json
```

4.`nodekey` Write `raftdata/static-nodes.json` (The nodekey public key is the producer public key)

#### Method 2. Use the consensus tool to initialize the cluster with one click

under `cmd/consensu`Run under the Director`init_pbft.sh`

```bash
cd cmd/consensus
./init_raft.sh --numNodes 3 --ip 127.0.0.1 127.0.0.2 127.0.0.3 --port 21001 21002 21003 --raftport 50401 50402 50403
```
+ `numNodes` Number of cluster nodes generated
+ `ip` List of ip addresses of the specified node (the default ip address is 127.0.0.1)
+ `port` The list of ports for the specified node. The default port is 21001 ~ 2100x, and x is numNodes.
+ `raftport` List of raft communication ports for the specified node (the default port is 50401 ~ 5040x, and x is numNodes)

After initialization, the corresponding node file is created in the cmd/consensus/raftdata directory.

### 3. Sub-chain startup process

```bash
sipe --datadir=raftdata --raft --port=21001 --raftport=50401 --role=subchain
```
+ `port` Must be consistent with the enode configured in static-nodes.json
+ `raft` Use raft mode
+ `raftport` raft port number, which must be consistent with the enode configured in static-nodes.json

### 4.View consensus status

```bash
> istanbul.getSnapshot()
```
+ `validators` pbft Block producer list
+ `votes` Votes for adding or removing validators
+ `tally` Total voting situation


