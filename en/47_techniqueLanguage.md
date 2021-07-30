We need to know some common terms and nouns before we begin to understand Simplechain and learn more about Simplechain in the future.

## Proper noun

> **External account**：EOAs(External Owned Accounts), associated with personal private key. Can be used to send transactions (transfer SIMPLE or send messages), like a savings card with a digital ID.

> **Contract account**：Contracts Accounts, which can store contract codes and contract data on Simplechain. External users cannot directly operate this account. It can only be called directly or indirectly from an external account.

> **Account Status**： account state, which indicates the status of an account in Simplechain. The account status changes when the account data changes. The account status includes four items: nonce, balance, root hash value of account storage content, and hash value of account code. Status data is not directly stored in the block.

> **Account Nonce**: account random number, which is the transaction count of the account. To prevent replay attacks.

> **Smart Contract**：Smart Contract,Simplechain supports writing Smart Contract code through Turing's complete advanced programming language. After being deployed on the chain, you can accept transaction requests and events from outside to trigger the execution of specific contract code logic and further generate new transactions and events. Even call other smart contracts.

> **World State**：state, which manages the mapping relationship between account address and account state. The status of all accounts constitutes the status of the entire blockchain.

> **Trading**：Transaction is the only way for external interaction with Simplechain. It must be signed by an external account. The miner executes the Transaction and finally packages it into the block.

> **Transaction receipt**：Receipt, which is convenient for zero-knowledge proof, index and search of transactions, and codes some specific information during transaction execution as transaction receipts.

> **Block**：block is a data block composed of a set of transactions, some auxiliary information (block header for short), and other block header hashes. Other block header hash indicates the parent block or the back block.

> **uncle block**：Uncle Block，an orphan Block that cannot be part of the main chain. If you are lucky enough to be taken into the Block chain by later blocks, it becomes Uncle Block. Additional rewards will be given to the blocks that have retained the isolated blocks. Once a block becomes a block, the block will be rewarded. Reduce Simplechain soft forking and balance the benefits of miners with slow network speed through the block reward mechanism.

> **Random number**：nonce，recorded in the block header, proof of hard work.

> **Gas**：fuel is a visualized concept of the amount of resources consumed during the operation of EVM when a transaction is packaged into a block. It is a metaphor that fuel is required to run EVM. In Simplechain, CPU resources and storage resources are expressed in accordance with built-in rules and Gas is used as a resource unit. Each time a virtual machine command is executed, a certain amount of Gas is consumed.

> **GasPrice**: fuel price, any transaction needs to include a unit price of fuel that is willing to pay, and finally according to the amount of fuel consumed by the transaction, the handling fee (usedGas * gasPrice) is calculated and paid to the miners.

> **Price forecast**：GPO(Gas Price Oracle),Gas Price forecast, predict the future GasPrice trend according to GasPrice of historical transactions.

## Technical terms

> **ZKP**: Zero Knowledge Proof.

> **EVM**：Ethereum Virtual Machine，which is a lightweight sandbox Virtual Machine for executing transactions.

> **Message**：A message is a virtual object that cannot be serialized and only exists in the running environment of Simplechain. A message mainly includes the sender, receiver, and gasLimit of the message;

> **序列化**：RLP is used to encode data into a set of byte data to facilitate data exchange and storage.

> **RLP**: ecursive length prefix encoding, a data encoding protocol that can compress data, is often used to serialize data in Simplechain.

> **MPT**：Merkel compressed prefix Tree, Merkle Patricia Tree, is a modified data structure that combines the advantages of Merkel Tree and prefix Tree, it is an important data structure used in Simplechain to organize and manage account data and generate transaction set hash.

> **Patricia Trie**: a compressed prefix tree, a tree that saves more space. For each node of trie, if the node is the only son of its parent node, it is combined with the parent node;

> **Merkle Tree**: Merkel Tree, also known as Hash Tree, the value of the leaf node of Merkel Tree is the content of the data item, or the Hash value of the data item; The value of the non-leaf node is based on the information of its child node, then it is calculated according to the Hash algorithm.

> **Whisper**：ciphertext is a communication protocol based on P2P. Through Whisper, nodes can send information to a specific node, achieve dual-node private chat and communication on multiple nodes by topic. It is mainly designed for DApp with large-scale point-to-point data discovery, signal negotiation, minimum transmission communication and complete privacy protection.
 
> **Swarm**： it is a distributed storage platform and content distribution service, and is the local basic Layer Service of Simplechain web3 technology stack; 

> **LLL，Sperpent、Mutan和Solidity**：the programming language used to write intelligent contract code, which can be compiled into EVM code.

> **ERC20**: it can be understood as a Token protocol specification of Simplechain. All Token contracts developed based on Simplechain comply with this specification. Tokens that comply with the ERC20 protocol specifications can be supported by various Simplechain wallets.

> **ERC721**: it is a Token protocol specification established on the ERC20 standard and a smart contract standard for non-fundable tokens (NFTs for short).
