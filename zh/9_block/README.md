---
id: docs_10
title: 区块
sidebar_label: 区块
---

[Simplechain](https://www.simplechain.com/)可以视为一个数据库，数据库数据的变更由交易催化。为了有效、有序管理交易，必须将一笔或多笔交易组成一个数据块，才能提交到数据库中。这个数据块即区块(Block)。一个区块不但包含了多笔交易，还记录一些额外数据，以便正确提交到数据库中。

下图是Simplechain区块数据结构与关系，讲解区块数据结构时不得不将其他数据一并呈现，只有掌握区块中各项数据来源才能真正了解区块链数据。

![19.png](https://i.loli.net/2020/04/27/C1eqRNahAynJMuv.png)

区块分为两部分：区块头(Header)和区块体(Body)。区块头信息量非常丰富，不但和上一个单元建立联系还记录了一些交易执行情况信息和矿工工作信息。在上图中涉及一个非常重要的概念 Trie，全名是默克尔压缩前缀树。

## 区块头数据解释

#### 各个字段数据如下：

**parentHash**

是一个哈希值，记录此区块直接引用的父区块哈希值。通过此记录，才能完整的将区块有序组织，形成一条区块链。并且可以防止父区块内容被修改，因为数据修改，区块哈希必然发生变化，因此一个区块直接或间接的强化了所有父辈区块，通过加密算法保证历史区块不可能被修改。

![13.png](https://i.loli.net/2020/04/27/Y5LpelbgwrKmyQj.png)

**sha3Uncles**

是一个哈希值，表示区块引用的多个叔辈区块。在区块体中也包含了多个叔辈的区块头信息，而sha3Uncles则是叔块集的 RLPHASH 哈希值。在比特币中只有成功挖出区块并被其他节点接受时才能获得奖励，是所有矿工在争取记账权和连带的奖励。而Simplechain稍有不同，不能成为主链一部分的孤儿区块，如果有幸被后来的区块收留进区块链就变成了叔块。收留了孤块的区块有额外的奖励。孤块一旦成为叔块，该区块统一可获得奖励。通过叔块奖励机制，来降低Simplechain软分叉和平衡网速慢的矿工利益。

**miner**

是一个地址，表示区块是此账户的矿工挖出，挖矿奖励将下发到此账户。

**stateRoot**

是一个哈希值，表示执行完此区块中的所有交易后Simplechain状态快照ID。因为Simplechain描述为一个状态机系统，因此快照ID称之为状态哈希值。又因为状态哈希是由所有账户状态按默克尔前缀树算法生成，因此称为状态默克尔树根值。

**transactionsRoot**

是一个哈希值，表示该区块中所有交易生成一颗默克尔树根节点哈希值。是一个密码学保证交易集合摘要。通过此Root可以直接校验某交易是否包含在此区块中。

**receiptRoot**

是一个哈希值，同样是默克尔树根节点哈希值。由区块交易在执行完成后生成的交易回执信息集合生成。

**logsBloom**

是一个256长度Byte数组。提取自receipt，用于快速定位查找交易回执中的智能合约事件信息。

**difficulty**

是 big.Int 值，表示此区块能被挖出的难度系数。

**number**

是 big.Int 值，表示此区块高度。用于对区块标注序号，在一条区块链上，区块高度必须是连续递增。

**gasLimit**

是 uint64 值，表示此区块所允许消耗的Gas燃料量。此数值根据父区块进行动态调整，调整的目的是调整区块所能包含的交易数量。

**gasUsed**

是uint64 值，表示此区块所有交易执行所实际消耗的Gas燃料量。

**timestamp**

是uint64 值，表示此区块创建的UTC时间戳，单位秒。因为Simplechain平均14.5s出一个区块(白皮书中研究是 12秒)，因此区块时间戳可以充当时间戳服务，但不能完全信任。

**extraData**

是一个长度不固定的Byte数组，最长32位。完全由矿工自定义，矿工一般会写一些公开推广类内容或者作为投票使用。

**mixHash**

是一个哈希值。用于校验区块是否正确挖出。实际上是区块头数据不包含nonce时的一个哈希值。

**nonce**

是一个8长度的Byte，实际是一个uint64 值。用于校验区块是否正确挖出，mixHash只有用一个正确的 nonce 才能进行PoW工作量证明。

## 关键代码

下面是Simplechain代码中定义的区块头和区块体结构定义代码，所有核心代码均在core/types/block.go文件中：

```javascript
//core/types/block.go:70
type Header struct {
   ParentHash  common.Hash    `json:"parentHash"       gencodec:"required"`
   UncleHash   common.Hash    `json:"sha3Uncles"       gencodec:"required"`
   Coinbase    common.Address `json:"miner"            gencodec:"required"`
   Root        common.Hash    `json:"stateRoot"        gencodec:"required"`
   TxHash      common.Hash    `json:"transactionsRoot" gencodec:"required"`
   ReceiptHash common.Hash    `json:"receiptsRoot"     gencodec:"required"`
   Bloom       Bloom          `json:"logsBloom"        gencodec:"required"`
   Difficulty  *big.Int       `json:"difficulty"       gencodec:"required"`
   Number      *big.Int       `json:"number"           gencodec:"required"`
   GasLimit    uint64         `json:"gasLimit"         gencodec:"required"`
   GasUsed     uint64         `json:"gasUsed"          gencodec:"required"`
   Time        uint64         `json:"timestamp"        gencodec:"required"`
   Extra       []byte         `json:"extraData"        gencodec:"required"`
   MixDigest   common.Hash    `json:"mixHash"`
   Nonce       BlockNonce     `json:"nonce"`
}
type Body struct {
	Transactions []*Transaction
	Uncles       []*Header
}
```
创建一个区块需要调用函数 NewBlock：

```javascript
func NewBlock(header *Header, txs []*Transaction, uncles []*Header, receipts []*Receipt) *Block {
   b := &Block{header: CopyHeader(header), td: new(big.Int)}

   // TODO: panic if len(txs) != len(receipts)
   if len(txs) == 0 {
      b.header.TxHash = EmptyRootHash
   } else {
      b.header.TxHash = DeriveSha(Transactions(txs))
      b.transactions = make(Transactions, len(txs))
      copy(b.transactions, txs)
   }

   if len(receipts) == 0 {
      b.header.ReceiptHash = EmptyRootHash
   } else {
      b.header.ReceiptHash = DeriveSha(Receipts(receipts))
      b.header.Bloom = CreateBloom(receipts)
   }

   if len(uncles) == 0 {
      b.header.UncleHash = EmptyUncleHash
   } else {
      b.header.UncleHash = CalcUncleHash(uncles)
      b.uncles = make([]*Header, len(uncles))
      for i := range uncles {
         b.uncles[i] = CopyHeader(uncles[i])
      }
   }

   return b
}
```

