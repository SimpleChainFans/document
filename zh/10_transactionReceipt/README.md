不同于比特币，Simplechain作为智能合约平台。每一笔交易作为消息在Simplechain虚拟机中执行时，均会获得一个交易回执信息(Receipt)。这份交易回执记录了关于此笔交易的处理结果信息：

![1.png](1.png)

回执信息分为三部分：共识信息、交易信息、区块信息。下面分别介绍各类信息。

![2.png](2.png)

## 交易回执内容介绍

**交易回执共识信息**

共识意味在在校验区块合法性时，这部分信息也参与校验。这些信息参与校验的原因是确保交易必须在区块中的固定顺序中执行，且记录了交易执行后的状态信息。这样可强化交易顺序。

```javascript
//core/state_processor.go:104
var root []byte
if config.IsByzantium(header.Number) {
    statedb.Finalise(true)
} else {
    root = statedb.IntermediateRoot(config.IsEIP158(header.Number)).Bytes()
}
//...
receipt := types.NewReceipt(root, failed, *usedGas)
```

- CumulativeGasUsed： 区块中已执行的交易累计消耗的Gas，包含当前交易。
- Logs: 当前交易执行所产生的智能合约事件列表。
- Bloom：是从 Logs 中提取的事件布隆过滤器，用于快速检测某主题的事件是否存在于Logs中。

这些信息是如何参与共识校验的呢？实际上参与校验的仅仅是回执哈希，而回执哈希计算只包含这些信息。 首先，在校验时获取整个区块回执信息的默克尔树的根哈希值。再判断此哈希值是否同区块头定义内容相同。

```javascript
//core/block_validator.go:92
receiptSha := types.DeriveSha(receipts)
if receiptSha != header.ReceiptHash {
   return fmt.Errorf("invalid receipt root hash (remote: %x local: %x)",
   header.ReceiptHash, receiptSha)
}
```

而函数types.DeriveSha中生成根哈希值，是将列表元素（这里是交易回执）的RLP编码信息构成默克树，最终获得列表的哈希值。

```javascript
//core/types/derive_sha.go:32
func DeriveSha(list DerivableList) common.Hash {
   keybuf := new(bytes.Buffer)
   trie := new(trie.Trie)
   for i := 0; i < list.Len(); i++ {
      keybuf.Reset()
      rlp.Encode(keybuf, uint(i))
      trie.Update(keybuf.Bytes(), list.GetRlp(i))
   }
   return trie.Hash()
}
// core/types/receipt.go:237
func (r Receipts) GetRlp(i int) []byte {
   bytes, err := rlp.EncodeToBytes(r[i])
   if err != nil {
      panic(err)
   }
   return bytes
}
```
交易回执实现了 RLP 编码接口。在方法EncodeRLP中是构建了一个私有的receiptRLP。

```javascript
//core/types/receipt.go:119
func (r *Receipt) EncodeRLP(w io.Writer) error {
	return rlp.Encode(w, 
	&receiptRLP{r.statusEncoding(), r.CumulativeGasUsed, r.Bloom, r.Logs})
}

```

## 交易回执交易信息

这部分信息记录的是关于回执所对应的交易信息，有：

- TxHash ： 交易回执所对应的交易哈希。
- ContractAddress： 当这笔交易是部署新合约时，记录新合约的地址。

```javascript
//core/state_processor.go:118
if msg.To() == nil {
 receipt.ContractAddress = crypto.CreateAddress(vmenv.Context.Origin, tx.Nonce())
}

```
- GasUsed: 这笔交易执行所消耗的Gas手续费

## 交易回执区块信息

这部分信息完全是为了方便外部读取交易回执，不但知道交易执行情况，还能方便的指定该交易属于哪个区块中第几笔交易。

- BlockHash: 交易所在区块哈希。
- BlockNumber: 交易所在区块高度。
- TransactionIndex： 交易在区块中的序号。

这三项信息，主要是在数据库 Leveldb 中读取交易回执时，实时指定。

```javascript
//core/rawdb/accessors_chain.go:315
receipts := make(types.Receipts, len(storageReceipts))
logIndex := uint(0)
for i, receipt := range storageReceipts {
   //...
   receipts[i] = (*types.Receipt)(receipt)
   receipts[i].BlockHash = hash
   receipts[i].BlockNumber = big.NewInt(0).SetUint64(number)
   receipts[i].TransactionIndex = uint(i)
}
```

## 交易回执构造

交易回执是在Simplechain虚拟机处理完交易后，根据结果整理出的交易执行结果信息。反映了交易执行前后Simplechain变化以及交易执行状态。

构造细节，已经在前面提及，不再细说。这里给出的完整的交易回执构造代码。

```javascript
// core/state_processor.go:94
context := NewEVMContext(msg, header, bc, author) 
vmenv := vm.NewEVM(context, statedb, config, cfg) 
_, gas, failed, err := ApplyMessage(vmenv, msg, gp)
if err != nil {
   return nil, 0, err
} 
var root []byte
if config.IsByzantium(header.Number) {
   statedb.Finalise(true)
} else {
   root = statedb.IntermediateRoot(config.IsEIP158(header.Number)).Bytes()
}
*usedGas += gas
 
receipt := types.NewReceipt(root, failed, *usedGas)
receipt.TxHash = tx.Hash()
receipt.GasUsed = gas 
if msg.To() == nil {
   receipt.ContractAddress = crypto.CreateAddress(vmenv.Context.Origin, tx.Nonce())
} 
receipt.Logs = statedb.GetLogs(tx.Hash())
receipt.Bloom = types.CreateBloom(types.Receipts{receipt})
receipt.BlockHash = statedb.BlockHash()
receipt.BlockNumber = header.Number
receipt.TransactionIndex = uint(statedb.TxIndex())

return receipt, gas, err
```
## 交易回执存储

交易回执作为交易执行中间产物，为了方便快速获取某笔交易的执行明细。Simplechain中有跟随区块存储时实时存储交易回执。但为了降低存储量，只存储了必要内容。

首先，在存储时，将交易回执对象转换为精简内容。

```javascript
//core/rawdb/accessors_chain.go:338
storageReceipts := make([]*types.ReceiptForStorage, len(receipts))
for i, receipt := range receipts {
   storageReceipts[i] = (*types.ReceiptForStorage)(receipt)
}
```
精简内容是专门为存储定义的一个结构ReceiptForStorage。存储时将交易回执集进行RLP编码存储。

```javascript
//core/rawdb/accessors_chain.go:342
bytes, err := rlp.EncodeToBytes(storageReceipts)
if err != nil {
   log.Crit("Failed to encode block receipts", "err", err)
} 
if err := db.Put(blockReceiptsKey(number, hash), bytes); err != nil {
   log.Crit("Failed to store block receipts", "err", err)
}
```
看ReceiptForStorage的EncodeRLP方法就知道存储了哪些内容

```javascript
//core/types/receipt.go:179
func (r *ReceiptForStorage) EncodeRLP(w io.Writer) error {
   enc := &receiptStorageRLP{
      PostStateOrStatus: (*Receipt)(r).statusEncoding(),
      CumulativeGasUsed: r.CumulativeGasUsed,
      TxHash:            r.TxHash,
      ContractAddress:   r.ContractAddress,
      Logs:              make([]*LogForStorage, len(r.Logs)),
      GasUsed:           r.GasUsed,
   }
   for i, log := range r.Logs {
      enc.Logs[i] = (*LogForStorage)(log)
   }
   return rlp.Encode(w, enc)
}
```
根据EncodeRLP方法实现，可以得出在存储时仅仅存储了部分内容，且 Logs 内容同样进行了特殊处理LogForStorage

![3.png](3.png)