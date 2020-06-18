## Basic synchronization process

Synchronization is a very important function of blockchain nodes. It is `consensus` to provide the consensus with the necessary operating conditions. Synchronization is divided into transaction synchronization and status synchronization. Transaction synchronization ensures that each transaction can reach each node correctly. Status synchronization can ensure that the backward nodes in the block can return to the latest status correctly. Only nodes with the latest block status can participate in the consensus.

## Transaction synchronization

Transaction synchronization allows transactions on the blockchain to reach all nodes as much as possible. Provides the basis for packaging transactions into blocks in consensus.

A transaction (tx1) is sent from the client to a node. After receiving the transaction, the node will put the transaction into its own transaction Pool (Tx Pool) for consensus packaging. At the same time, the node broadcasts the transaction to other nodes. After receiving the transaction, other nodes will also put the transaction into their own transaction pool. The transaction may be lost during the process of sending. In order to make the transaction reach all nodes as much as possible, the node that receives the broadcast transaction will choose other nodes according to a certain strategy, broadcast again.

**Transaction broadcast strategy**

If each node has no restrictions on forwarding or broadcasting received transactions, the bandwidth will be occupied and the transaction broadcast avalanche will occur. In order to avoid the avalanche of transaction Broadcasting, Simplechain chose a relatively delicate transaction broadcasting strategy based on experience. On the premise of ensuring the reachability of the transaction as much as possible, reduce the repeated transaction broadcast as much as possible.

* For transactions from the SDK, broadcast to all nodes
* A transaction is broadcast only once on a node. When a duplicate transaction is received, no secondary broadcast is performed.

Through the above strategy, transactions can reach all nodes as much as possible, but a transaction cannot reach a node with a minimum probability. This situation is allowed. The purpose of reaching as many nodes as possible is to package, reach consensus, and confirm the transaction as soon as possible, so that the transaction can be executed as quickly as possible. When the transaction does not reach a certain node, the transaction execution time will only be longer and the correctness of the transaction will not be affected.

## Status synchronization

Status synchronization is to keep the status of blockchain nodes up to date. The status of a blockchain is the new and old data that a blockchain node currently holds, that is, the height of the current block that a node holds. If the block height of a node is the highest block height of the blockchain, the node has the latest status of the blockchain. Only nodes with the latest status can participate in the consensus and carry out the consensus of the next new district.


When a new node is added to the blockchain, or a node that has been disconnected is restored to the network, the block of this node lags behind other nodes, and the status is not up to date. Status synchronization is required. As shown in the figure, the Node (Node 1) that needs state synchronization initiatively requests other nodes to download the block. During the entire download process, the download load is distributed to multiple nodes.

**Status synchronization and Download Queue**

When a blockchain node is running, it regularly broadcasts its highest block height to other nodes. After a node receives the block height broadcasted by other nodes, it compares it with its own block height. If its own block height falls behind this block height, the block download process is started.

The download of the block is completed by request. The node that enters the download process randomly selects the node that meets the requirements and sends the block interval to be downloaded. The node that receives the download request will reply to the corresponding block based on the request content.

The node that receives the reply block maintains a download queue locally to buffer and sort the downloaded blocks. A download queue is a priority queue in order of block heights. The downloaded blocks are continuously inserted into the Download Queue. When the blocks in the queue can connect to the current local blockchain of the node, the blocks are taken out of the download queue, connect to the current local blockchain.

## Examples of synchronization scenarios

### Transaction synchronization

The process of broadcasting a transaction to all nodes:

1. A transaction is sent to a node through a channel or RPC
2. The node that receives the transaction broadcasts the transaction to other nodes in full.
3. After other nodes receive the transaction, to be on the safe side, select 25% of the nodes to broadcast again
4. The node receives a broadcast transaction and will not broadcast it again

### Status synchronization

Broadcast logic when node blocks

1. Outbound block of a node
2. This section broadcasts its latest status (latest block height, highest block hash, Genesis block hash) to all nodes
3. After other nodes receive the peer status, the peer data managed locally is updated.
