Compared with Bitcoin's "UTXO" balance model, [SimpleChain](https://www.simplechain.com/) similar to Simplechain the account balance model is used.
[SimpleChain](https://www.simplechain.com/) it enriches the account content and can store any amount of data in addition to the balance. And use the maintainability of account data to build an intelligent contract account.

Actually [SimpleChain](https://www.simplechain.com/)it is an account model refined to realize smart contracts.
Isolate data in an account. The information between accounts is independent of each other and does not interfere with each other. Cooperate again [SimpleChain](https://www.simplechain.com/)Virtual machine, let the smart contract sandbox run.

[SimpleChain](https://www.simplechain.com/) as an intelligent contract operation platform, accounts are divided into two types: external account (EOAs) and contract account.

## External account

EOAs-external owned accouts are accounts created by people through private keys.
It is the mapping of real-world financial accounts, and anyone with the private key of the account can control the account.
Just like a bank card, when you withdraw money from an ATM, you only need to enter the correct password to trade.
This is also the only medium for human beings to communicate with Simplechain account books, because[SimpleChain](https://www.simplechain.com/) transaction in needs to be signed,
You can only use a private external account signature.

Summary of external account characteristics:

1. Have sipc balance.
1. Can send transactions, including transfer and execution of contract codes.
1. Controlled by the private key.
1. No relevant executable code.

## Contract account

The account that contains the contract code. Created by an external account or contract, the contract is automatically assigned to an account address when created,
Used to store contract code and stored data generated during contract deployment or execution. The contract account address is generated by the SHA3 hash algorithm, not the private key. Because of the selfless key, no one can use the contract account as an external account. Contract execution code can only be driven through external accounts.

The following is the contract address generation algorithm: `Keccak256(rlp([sender,nonce])[12:]`

```go
// crypto/crypto.go:74
func CreateAddress(b common.Address, nonce uint64) common.Address {
    data, _ := rlp.EncodeToBytes([]interface{}{b, nonce})
    return common.BytesToAddress(Keccak256(data)[12:])
}
```

Because the contract is created by another account, the creator's address and the random number of the transaction are hashed and then the truncated part is generated.

It is particularly important to note that in[EIP1014](http://eips.ethereum.org/EIPS/eip-1014) another algorithm for generating contract addresses proposed in.
Its purpose is to facilitate the status channel by determining a stable contract address for content output. You can know the exact contract address before deploying the contract. The following is the algorithm method:`keccak256( 0xff ++ address ++ salt ++ keccak256(init_code))[12:]`。

```go
// crypto/crypto.go:81
func CreateAddress2(b common.Address, salt [32]byte, inithash []byte) common.Address {
    return common.BytesToAddress(Keccak256([]byte{0xff}, b.Bytes(), salt[:], inithash)[12:])
}
```

Summary of contract account characteristics:

1. Have sipc balance.
2. Relevant executable code (contract code).
3. Contract codes can be called by transactions or other contract messages.
4. Other contract codes can be called when the contract code is executed.
5. When the contract code is executed, it can perform complex operations and permanently change the data storage inside the contract.


## Difference comparison

To sum up, the following table lists the differences between the two types of accounts, and contract accounts are better than external accounts.
However, the external account is the only medium for people to communicate with Simplechain, and it is complementary to the contract account.

|Item|External account|Contract account|
|----|----|----|
|private Key| ✔️ | ✖️|
|balance| ✔️ |✔️|
|code|  ✖️|✔️|
|Multiple signature	| ✖️|✔️|
|Control mode| Private key control | Execute the contract through an external account |

Multiple signatures are listed above because [SimpleChain](https://www.simplechain.com/) an external account is created by only one independent private key and cannot be signed multiple times. However, the contract is programmable, and the logic conforming to multiple signatures can be written to implement an account that supports multiple signatures.

## Account data structure

[SimpleChain](https://www.simplechain.com/)Data is organized by accounts, and changes in account data cause changes in account status. this causes changes in the Simplechain state.

Logically, the data structures of the two types of accounts are the same:

![Simplechain account](https://i.loli.net/2020/05/09/Nrj4wSvnWyPtEkg.png)

The corresponding code is as follows:

```go
// core/state/state_object.go:100
type Account struct {
    Nonce    uint64
    Balance  *big.Int
    Root     common.Hash
    CodeHash []byte
}
```
However, the data storage is slightly different, because the external account does not have internal storage data and contract code, so the external account data `StateRootHash` And `CodeHash` Is an empty default value. Once it belongs to an empty default value, it is not stored in the corresponding physical database. In the program logic, exist `code` The contract account. That `CodeHash` When the value is null, the account is an external account, otherwise it is a contract account.

![14_sipc.png](https://i.loli.net/2020/05/09/J4iywoUuEhBM695.png)

The above figure is [SimpleChain](https://www.simplechain.com/) the account data storage structure. In fact, only key data is stored within the account, while the contract code and the contract data are associated by corresponding hash values. Because each account object will be treated as a [SimpleChain](https://www.simplechain.com/) the data storage of a leaf in the account tree cannot be too large.

In the field of cryptography, Nonce represents a number that is used only once. It is often a random or pseudo random number to avoid duplication.

[SimpleChain](https://www.simplechain.com/) adding Nonce to the account can avoid replay attacks, but not randomly generated.
The initial Nonce value of the account is 0. The Nonce value is added once for each subsequent account execution. The counting logic of one of them is as follows:

```go
// core/state_transition.go:212
st.state.SetNonce(msg.From(), st.state.GetNonce(sender.Address())+1)
```

The additional advantage of this is that Nonce can generally be used as the counter for the number of transactions of the account, especially for the contract account, the number of times the contract is called can be accurately recorded.

and Balance Then record the number of sipcs owned by the account, which is called account balance. Transfer assets (Transfer) are in one account Balance Add up and reduce in another account.

```go
// core/evm.go:94
func Transfer(db vm.StateDB, sender, recipient common.Address, amount *big.Int) {
    db.SubBalance(sender, amount)
    db.AddBalance(recipient, amount)
}
// core/vm/evm.go:191
if !evm.Context.CanTransfer(evm.StateDB, caller.Address(), value) {
    return nil, gas, ErrInsufficientBalance
}
// core/vm/evm.go:214
evm.Transfer(evm.StateDB, caller.Address(), to.Address(), value)
```
Of course, the balance of the transferor must be guaranteed to be sufficient. Before the transfer `CanTransfer` Check,
If the balance is sufficient, execute `Transfer` Transfer `Value` The number of ether.

Account Status hash value `StateRoot`, is the root value of a Merkle Patricia Tree composed of methods and field information owned by the contract. In short, it is the root node value of a binary Tree. Any slight change in the contract status will eventually cause `StateRoot` Change, so the change of contract status will be reflected in the account `StateRoot` Up.

At the same time, you can directly use `StateRoot` Quickly read a specific state data from Leveldb, such as the contract creator.
Pass [SimpleChain](https://www.simplechain.com/) API [web3.eth.getStorageAt]() data at any position in the contract can be read.

Next, let's use a sample code to feel [SimpleChain](https://www.simplechain.com/)data storage.

```go
import(...)
var toAddr =common.HexToAddress
var toHash =common.BytesToHash

func main()  {
    statadb, _ := state.New(common.Hash{},
        state.NewDatabase(rawdb.NewMemoryDatabase()))// 1

    acct1:=toAddr("0x0bB141C2F7d4d12B1D27E62F86254e6ccEd5FF9a")// 2
    acct2:=toAddr("0x77de172A492C40217e48Ebb7EEFf9b2d7dF8151B")

    statadb.AddBalance(acct1,big.NewInt(100))
    statadb.AddBalance(acct2,big.NewInt(888))

    contract:=crypto.CreateAddress(acct1,statadb.GetNonce(acct1))//3
    statadb.CreateAccount(contract)
    statadb.SetCode(contract,[]byte("contract code bytes"))//4

    statadb.SetNonce(contract,1)
    statadb.SetState(contract,toHash([]byte("owner")),toHash(acct1.Bytes()))//5
    statadb.SetState(contract,toHash([]byte("name")),toHash([]byte("ysqi")))

    statadb.SetState(contract,toHash([]byte("online")),toHash([]byte{1})
    statadb.SetState(contract,toHash([]byte("online")),toHash([]byte{}))//6

    statadb.Commit(true)//7
    fmt.Println(string(statadb.Dump()))//8
}

```
In the above code, we created three accounts and submitted them to the database. Finally, print out the data information of all accounts in the current data:

+ A line of code involves multiple operations. First, create a memory KV database and package it as a stata database instance,
Finally, use an empty DB-level `StateRoot` , initialize a Simplechain statadb.
+ Define two accounts acct1 and acct2, and add 100 and 888 to the account balance respectively.
+ Simulate the creation process of a contract account, create a contract account address from the external account acct1, and load the address into statadb.
+ Add the contract code to the newly created contract account, while writing the contract code,
  Can use `crypto.Keccak256Hash(code)` Calculate the contract code hash and keep it in the account data.
+ Simulate the contract execution process, involving modifying the contract status and adding three new status data owner , `name` And `online` , corresponding to different   values.
+ What is different from the previous is that it is given status online The assignment is empty []byte{} Because the default value for all states is []byte{} ,
  When submitted to the database, Leveldb deletes this record from the database file if it considers these states to have no valid values. Therefore, this operation is actually a delete State online Operation.
+ All the above operations only occur in the statdb memory and are not actually written to database files.
  Implementation `Commit` , then all changes about statadb will be updated to the database file.
+ Once the data is submitted, you can use `Dump` Command to find all data related to this stata from the database, including all accounts.
And return it in JSON format. Here, we will directly print out the returned results.

The code execution output is as follows:

```json
{
    "root": "3a25b0816cf007c0b878ca7a62ba35ee0337fa53703f281c41a791a137519f00",
    "accounts": {
        "0bb141c2f7d4d12b1d27e62f86254e6cced5ff9a": {
            "balance": "100",
            "nonce": 0,
            "root": "56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
            "codeHash": "c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470",
            "code": "",
            "storage": {}
        },
        "77de172a492c40217e48ebb7eeff9b2d7df8151b": {
            "balance": "888",
            "nonce": 0,
            "root": "56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
            "codeHash": "c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470",
            "code": "",
            "storage": {}
        },
        "80580f576731dc1e1dcc53d80b261e228c447cdd": {
            "balance": "0",
            "nonce": 1,
            "root": "1f6d937817f2ac217d8b123c4983c45141e50bd0c358c07f3c19c7b526dd4267",
            "codeHash": "c668dac8131a99c411450ba912234439ace20d1cc1084f8e198fee0a334bc592",
            "code": "636f6e747261637420636f6465206279746573",
            "storage": {
                "000000000000000000000000000000000000000000000000000000006e616d65": "8479737169",
                "0000000000000000000000000000000000000000000000000000006f776e6572": "940bb141c2f7d4d12b1d27e62f86254e6cced5ff9a"
            }
        }
    }
}
```

We can see that these displayed data directly correspond to all the operations we just performed.
Only contract accounts `storage` And `code` . And external account `codeHash` And `root` The same value is a default value.