Simplechain的区块、交易等数据最终都是存储在`Level DB`数据库中。`Level DB` 数据库是一个键值对（ key-value ）数据库， key一般与散列相关，value则是存储内容的RLP编码。

## 一、KV存储LevelDB

### MPT树

MPT(Merkle Patricia Trie)，是一种用hash索引数据的前缀树。

从宏观上来说，MPT树是一棵前缀树，用key查询value。通过key去查询value，就是用key去在MPT树上进行索引，在经过多个中间节点后，最终到达存储数据的叶子节点。

从细节上来说，MPT树，是一棵Merkle树，每个树上节点的索引，都是这个节点的hash值。在用key查找value的时候，是根据key在某节点内部，获取下一个需要跳转的节点的hash值，拿到下一个节点的hash值，才能从底层的数据库中取出下一个节点的数据，之后，再用key，去下一个节点中查询下下个节点的hash值，直至到达value所在的叶子节点。

当MPT树上某个叶子节点的数据更新后，此叶子节点的hash也会更新，随之而来的，是这个叶子节点回溯到根节点的所有中间节点的hash都会更新。最终，MPT根节点的hash也会更新。当要索引这个新的数据时，用MPT新的根节点hash，从底层数据库查出新的根节点，再往后一层层遍历，最终找到新的数据。而如果要查询历史数据，则可用老的树根hash，从底层数据库取出老的根节点，再往下遍历，就可查询到历史的数据。

MPT树的实现图:

![图片](1.png)

### 账户State

在Simplechain上，数据是以account为单位存储的，每个account内，保存着这个合约(用户)的代码、参数、nonce等数据。account的数据，通过account的地址（address）进行索引。

随着account数据的改变，account的hash也进行改变。于此同时，MPT的根的hash也会改变。不同的时候，account的数据不同，对应的MPT的根就不同。此处，以太坊把这层含义进行了具体化，提出了“状态”的概念。把MPT根的hash，叫state root。不同的state root，对应着不同的“状态”，对应查询到不同的MPT根节点，再用account的address从不同的MPT根节点查询到此状态下的account数据。不同的state，拿到的MPT根节点不同，查询的account也许会有不同。

state root是区块中的一个字段，每个区块对应着不同的“状态”。区块中的交易会对account进行操作，进而改变account中的数据。不同的区块下，account的数据有所不同，即此区块的状态有所不同，具体的，是state root不同。从某个区块中取出这个区块的state root，查询到MPT的根节点，就能索引到这个区块当时account的数据历史。



![图片](2.png)

## 二、索引存储StormDB

StormDB(a simple and powerful toolkit for [BoltDB](https://github.com/coreos/bbolt))是一种基于boltDB实现的数据库，它支持索引与搜索查询，相对于传统关系数据库MySQL、Oracle等，StormDB使用单一文件存储，没有单独的服务，更加轻量便于系统移植，SImplechain使用它来存储跨链交易。

### 名词解释

**Bucket**

自定义对象的存储单元

**Id**

存储对象的主键索引，唯一不可重复

**Index**

排序索引，目前只支持数值类型的字段排序(int8, int16, int32, int64, uint8, uint16, uint32, uint64)，其他类型皆使用编码后字节比较的方式来进行排序。

**Unique**

唯一哈希索引

**Options**

查询选项，支持Limit(限制条数)、OrderBy(根据字段排序)、Skip(跳过的条数)等

**Matcher**

查询的传入条件，支持Eq、Gt(greater than)、Lt(less than)、And、Or、Not、Re(正则匹配)等

### 举例

以跨链交易存储结构为例，解释上述名词。

```go
type CrossTransactionIndexed struct {
   PK       uint64         `storm:"id,increment"`
   CtxId    common.Hash    `storm:"unique"`
   From     common.Address `storm:"index"`
   To       common.Address `storm:"index"`
   TxHash   common.Hash    `storm:"index"`
   Price    *big.Float     `storm:"index"`
   
   Status uint8 `storm:"index"`
   Value            *big.Int
   BlockNum         uint64 `storm:"index"`
   BlockHash        common.Hash
   DestinationId    *big.Int
   DestinationValue *big.Int `storm:"index"`
   Input            []byte
   
   V []*big.Int
   R []*big.Int
   S []*big.Int
}
```
解释如下：
* storm:"id,increment"：表示此字段为主键，increment表示在插入时自动递增
* storm:"unique"：表示此字段使用不可重复的哈希索引
* storm:"index"：表示此字段使用排序索引

### CRUD与事务

**插入对象**

```go
user := User{
  ID: 10,
  Group: "staff",
  Email: "john@provider.com",
  Name: "John",
  Age: 21,
  CreatedAt: time.Now(),
}
err := db.Save(&user)
// err == nil
user.ID++
err = db.Save(&user)
// err == storm.ErrAlreadyExists
```
**更新对象**
```go
// Update multiple fields
err := db.Update(&User{ID: 10, Name: "Jack", Age: 45})
// Update a single field
err := db.UpdateField(&User{ID: 10}, "Age", 0)
```
**删除对象**
```go
err := db.DeleteStruct(&user)
```
**删除Bucket**
```go
err := db.Drop(&User)
//OR
err := db.Drop("User")
```
**查询对象**
```go
var users []User
err := db.Find("Group", "staff", &users, storm.Skip(10))
err = db.Find("Group", "staff", &users, storm.Limit(10))
err = db.Find("Group", "staff", &users, storm.Reverse())
err = db.Find("Group", "staff", &users, storm.Limit(10), storm.Skip(10), storm.Reverse())
err = db.All(&users, storm.Limit(10), storm.Skip(10), storm.Reverse())
err = db.AllByIndex("CreatedAt", &users, storm.Limit(10), storm.Skip(10), storm.Reverse())
err = db.Range("Age", 10, 21, &users, storm.Limit(10), storm.Skip(10), storm.Reverse())
```
**条件查询**
```go
query := db.Select(q.Gte("Age", 7), q.Lte("Age", 77))
var users []User
err = query.Find(&users)
```
**使用事务**
```go
tx, err := db.Begin(true)
if err != nil {
  return err
}
defer tx.Rollback()
accountA.Amount -= 100
accountB.Amount += 100
err = tx.Save(accountA)
if err != nil {
  return err
}
err = tx.Save(accountB)
if err != nil {
  return err
}
return tx.Commit()
```
## 三、高效内存多索引存储

为了适应不同要求的索引类型以及效率需求，单独使用stormDB已难以满足，因此在内存中引入多索引结构来存储高读写频次的数据。

### 哈希索引

Simplechain中使用的哈希索引皆为键值唯一的哈希索引，因此直接使用map存储。

**插入与更新**

```go
idx := make(map[Key]Value)
idx[key]=value
```
**删除**
```go
delete(idx,key)
```
**读取**
```go
if v,ok := idx[key]; ok {//...}
```
### 排序索引

Simplechain使用红黑树作为排序索引，用于存储任意有顺序要求的数据类型。

**初始化**

```go
//comparator:key的比较方法
//isMulti:是否允许key值重复
redblacktree.NewWith(container.UInt64Comparator, true)
```
**插入与更新**
```go
idx.Put(key,value)
```
**查询**
```go
//返回迭代器，在multi模式下，返回lowerbound
itr := idx.Get(key)
```
**删除key**
```go
//在multi模式下，删除此key关联的所有value
idx.Remove(key)
```
**删除特定对象**
```go
//获得key的lowerBound与upperBound迭代器，遍历删除满足条件的对象
for it:=idx.LowerBound(key); it!=idx.UpperBound(key); it.Next() {
   if ... {
      idx.RemoveOne(itr)
      break
   }
}
```

