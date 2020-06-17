### Create a genesis block file

Create a new file genesis.json with the following content. 

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

Of which `chainId` the ID of the test network. the ID of the primary network is 1,`difficulty` for the difficulty of mining, to facilitate the operation of the test network, the difficulty setting is low.

#### Start node

**1.Create the storage directory nodedata1 for node one**

    mkdir nodedata1

**2.Use genesis.json to initialize the genesis block of node 1**

    sipe init --datadir nodedata1 genesis.json

**3.Start the node and specify the networkid. Ensure that the specified networkid is the same when communicating with the node**

    sipe --datadir nodedata1 --port 30312 --rpc --rpcaddr 127.0.0.1 --rpcport 8541 --networkid 10001 console

**4. View the node information in the open console to obtain the node enode**

    > admin.nodeInfo 
    {
    enode: "enode://05a9c3bd1f6716a1806e677b8337d4e1eb4b9f57d8f94d11bcf4870fd8d5d943b9591 1a3c51f40714f33a307049d8c0c1a7019a71d099a27c6a939a85a809110@[::]:30312",
    id: "05a9c3bd1f6716a1806e677b8337d4e1eb4b9f57d8f94d11bcf4870fd8d5d943b95911a3c51f4 0714f33a307049d8c0c1a7019a71d099a27c6a939a85a809110",
    ip: "::",
    listenAddr: "[::]:30312",
    name: "Sipe/v1.0.2-stable-0cbf2a41/darwin-amd64/go1.12.1", 
    ports: {
        discovery: 30312,
        listener: 30312
    },
    protocols: { 
        eth: {
          config: { 
            chainId: 100, 
            eip150Hash: "0x0000000000000000000000000000000000000000000000000000000000000000", 
            eip155Block: 0,
            eip158Block: 0,
            homesteadBlock: 0 
          },
          difficulty: 131072,
          genesis: "0x5e1fc79cb4ffa4739177b5408045cd5d51c6cf766133f23f7cd72ee1f8d790e0", 
          head: "0x5e1fc79cb4ffa4739177b5408045cd5d51c6cf766133f23f7cd72ee1f8d790e0", 
          network: 10001
      }}
    }
             
**3.Start node 2**

**1. Create the storage directory nodedata2 for node one**

    mkdir nodedata2
  
**2. Use genesis.json to initialize the genesis block of node 1.**

    sipe init --datadir nodedata2 genesis.json

**3. Start the node to ensure that networdid is the same as node one. Note that when configuring bootnodes, replace the enode [::] obtained by node one with the IP address of node one, that is `127.0.0.1`**

    sipe --datadir nodedata2 --port 30313 --rpc --rpcaddr 127.0.0.1 --rpcport 8542 --networkid 10001 --bootnodes "enode://05a9c3bd1f6716a1806e677b8337d4e1eb4b9f57d8f94d11bcf4870fd8d5d943b9591 1a3c51f40714f33a307049d8c0c1a7019a71d099a27c6a939a85a809110@127.0.0.1:30312"
    console

**4. View the information of the associated node. If the returned result is not empty, it is confirmed that node 2 and node 1 are successfully connected.**
                        
    > admin.peers [{
    caps: ["eth/63"],
    id: "05a9c3bd1f6716a1806e677b8337d4e1eb4b9f57d8f94d11bcf4870fd8d5d943b95911a3c51f4 0714f33a307049d8c0c1a7019a71d099a27c6a939a85a809110",
    name: "Sipe/v1.0.2-stable-0cbf2a41/darwin-amd64/go1.12.1", network: {
    inbound: false,
    localAddress: "127.0.0.1:58388", 
    remoteAddress: "127.0.0.1:30312",             
    static: false,
    trusted: false 
    },
    protocols: { 
          eth: {
              difficulty: 131072,
              head: "0x5e1fc79cb4ffa4739177b5408045cd5d51c6cf766133f23f7cd72ee1f8d790e0", 
              version: 63
          }
    }
    }]

#### 3.Mining in the test network

**1.Create an account on Node 1 and set it as the miner address**

    > personal.newAccount()
    Passphrase:
    Repeat passphrase:="0x7f53309f95559c52d08f18724c0b24aa758d1953"
    > miner.setEtherbase('0x7f53309f95559c52d08f18724c0b24aa758d1953') 
    true

**2.Start mining on Node 1**
 
    > miner.start()
    INFO [06-19|10:53:15.918] Updated mining threads threads=0
    INFO [06-19|10:53:15.918] Transaction pool price threshold updated price=1000000000 
    INFO [06-19|10:53:15.918] Starting mining operation
    null
    > INFO [06-19|10:53:15.918] Commit new mining work      number=1 txs=0 uncles=0
    elapsed=207.516μs
    INFO [06-19|10:53:47.601] Successfully sealed new block   number=1
    hash=755f08...62e560
    INFO [06-19|10:53:47.607] 🔨 mined potential block     number=1
    hash=755f08...62e560

**3.Confirm the synchronization block on Node 2**

    INFO [06-19|10:53:49.246] Block synchronisation started
    INFO [06-19|10:53:49.538] Imported new block headers count=2 elapsed=6.482ms number=2 hash=c7c0a9...79db3e ignored=0
    INFO [06-19|10:53:49.539] Imported new chain segment blocks=2 txs=0 mgas=0.000 elapsed=766.945μs mgasps=0.000 number=2 hash=c7c0a9...79db3e cache=1.20kB
    INFO [06-19|10:53:49.556] Imported new state entries count=3 elapsed=90.308μs processed=3 pending=0 retry=0 duplicate=0 unexpected=0
    INFO [06-19|10:53:49.601] Fast sync complete, auto disabling
    INFO [06-19|10:53:59.119] Imported new chain segment blocks=1 txs=0 mgas=0.000 elapsed=1.212ms mgasps=0.000 number=3 hash=6dd8b2...194509 cache=1.81kB

#### 4.Transfer money in the test network

**1. Use the console to create another account.**

    > personal.newAccount()
    Passphrase:
    Repeat passphrase: "0xf9143e3b7de8ce91e463e30480f5afe84d3067ba"

**2. Use the password to unlock the account of the transferor before transferring money.**

     > personal.unlockAccount('0x7f53309f95559c52d08f18724c0b24aa758d1953') Unlock account 0x7f53309f95559c52d08f18724c0b24aa758d1953
     Passphrase:
     true

**3. Send the transaction for transfer, where from is the transferor, here is the address of the miner, to is the payee, and the value is the transfer amount.**

     > eth.sendTransaction({from:"0x7f53309f95559c52d08f18724c0b24aa758d1953",to:"0xf9143e 3b7de8ce91e463e30480f5afe84d3067ba",value:web3.toWei(10,"ether")}) "0x5a6fbb3161329ca2591b7ecbcaca8a15a94cac5d402fce929f24504c76b8b7bb"

**4. Confirm receipt.**

     > eth.getBalance('0xf9143e3b7de8ce91e463e30480f5afe84d3067ba') 10000000000000000000

