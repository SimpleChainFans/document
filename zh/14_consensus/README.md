共识机制是区块链事务达成分布式共识的算法。由于点对点网络下存在着或高或低的网络延迟，所以各个节点接收到的事务的先后顺序可能不一样，因此区块链系统需要设计一种机制让节点对在差不多时间内发生的事务的先后顺序实现共识，这就是共识机制。

## PoW共识算法

PoW 即通过工作结果来证明你完成了相应的工作哈希函数的特征：
1. 免碰撞，即不存在输入值不同，经过散列变换，而散列值相同的情况。
2. 隐匿性，即给定一个散列值，想要反向逆推出输入值，在计算上是不可行的。
3. 不存在比穷举更好的方法，以使得散列值落在特定的范围。POW算法原理：节点通过不断地更换随机数来探寻合适的哈希值，当节点最先计算出合适的哈希值，它所打包的块如果通过其他共识节点的验证，则会被加入到区块链中。

### Scrypt算法简介

Simplechain 采用的挖矿算法是Scrypt算法。Scrypt是内存依赖型的POW算法，它是一种符合PoW共识机制的算法。Scrypt算法过程中也需要计算哈希值，但是，Scrypt计算过程中需要使用较多的内存资源。

### Scrypt算法过程

Scrypt算法使用的几个函数环环相扣，本节按函数从里到外的调用顺序描述。

### Salsa20/8

```c++
#define R(a,b) (((a) << (b)) | ((a) >> (32 - (b))))
void salsa20_word_specification(uint32 out[16],uint32 in[16])
{
    int i;
    uint32 x[16];
    for (i = 0;i < 16;++i) x[i] = in[i];
    for (i = 8;i > 0;i -= 2) {
        x[ 4] ^= R(x[ 0]+x[12], 7); x[ 8] ^= R(x[ 4]+x[ 0], 9);
        x[12] ^= R(x[ 8]+x[ 4],13); x[ 0] ^= R(x[12]+x[ 8],18);
        x[ 9] ^= R(x[ 5]+x[ 1], 7); x[13] ^= R(x[ 9]+x[ 5], 9);
        x[ 1] ^= R(x[13]+x[ 9],13); x[ 5] ^= R(x[ 1]+x[13],18);
        x[14] ^= R(x[10]+x[ 6], 7); x[ 2] ^= R(x[14]+x[10], 9);
        x[ 6] ^= R(x[ 2]+x[14],13); x[10] ^= R(x[ 6]+x[ 2],18);
        x[ 3] ^= R(x[15]+x[11], 7); x[ 7] ^= R(x[ 3]+x[15], 9);
        x[11] ^= R(x[ 7]+x[ 3],13); x[15] ^= R(x[11]+x[ 7],18);
        x[ 1] ^= R(x[ 0]+x[ 3], 7); x[ 2] ^= R(x[ 1]+x[ 0], 9);
        x[ 3] ^= R(x[ 2]+x[ 1],13); x[ 0] ^= R(x[ 3]+x[ 2],18);
        x[ 6] ^= R(x[ 5]+x[ 4], 7); x[ 7] ^= R(x[ 6]+x[ 5], 9);
        x[ 4] ^= R(x[ 7]+x[ 6],13); x[ 5] ^= R(x[ 4]+x[ 7],18);
        x[11] ^= R(x[10]+x[ 9], 7); x[ 8] ^= R(x[11]+x[10], 9);
        x[ 9] ^= R(x[ 8]+x[11],13); x[10] ^= R(x[ 9]+x[ 8],18);
        x[12] ^= R(x[15]+x[14], 7); x[13] ^= R(x[12]+x[15], 9);
        x[14] ^= R(x[13]+x[12],13); x[15] ^= R(x[14]+x[13],18);
    }
    for (i = 0;i < 16;++i) out[i] = x[i] + in[i];
}

```

###  scryptBlockMix

```c++
 Parameters:
            r Block size parameter.
   Input:
            B[0] || B[1] || ... || B[2 * r - 1]
                   Input octet string (of size 128 * r octets),
                   treated as 2 * r 64-octet blocks,
                   where each element in B is a 64-octet block.
   Output:
            B'[0] || B'[1] || ... || B'[2 * r - 1]
                   Output octet string.
   Steps:
     1. X = B[2 * r - 1]
     2. for i = 0 to 2 * r - 1 do
          T = X xor B[i]
          X = Salsa (T)
          Y[i] = X
        end for
     3. B' = (Y[0], Y[2], ..., Y[2 * r - 2],
              Y[1], Y[3], ..., Y[2 * r - 1])

```
### scryptROMix

```c++
 Input:
            r Block size parameter.
            B Input octet vector of length 128 * r octets.
            N CPU/Memory cost parameter, must be larger than 1,
                    a power of 2, and less than 2^(128 * r / 8).
   Output:
            B' Output octet vector of length 128 * r octets.
   Steps:
     1. X = B
     2. for i = 0 to N - 1 do
          V[i] = X
          X = scryptBlockMix (X)
        end for
     3. for i = 0 to N - 1 do
          j = Integerify (X) mod N
                 where Integerify (B[0] ... B[2 * r - 1]) is defined
                 as the result of interpreting B[2 * r - 1] as a
                 little-endian integer.
          T = X xor V[j]
          X = scryptBlockMix (T)
        end for
     4. B' = X

```
### scrypt

```c++
Input:
            P Passphrase, an octet string.
            S Salt, an octet string.
            N CPU/Memory cost parameter, must be larger than 1,
                    a power of 2, and less than 2^(128 * r / 8).
            r Block size parameter.
            p Parallelization parameter, a positive integer
                    less than or equal to ((2^32-1) * hLen) / MFLen
                    where hLen is 32 and MFlen is 128 * r.
            dkLen Intended output length in octets of the derived
                    key; a positive integer less than or equal to
                    (2^32 - 1) * hLen where hLen is 32.
   Output:
            DK Derived key, of length dkLen octets.
   Steps:
    1. Initialize an array B consisting of p blocks of 128 * r octets
       each:
        B[0] || B[1] || ... || B[p - 1] =
          PBKDF2-HMAC-SHA256 (P, S, 1, p * 128 * r)
    2. for i = 0 to p - 1 do
          B[i] = scryptROMix (r, B[i], N)
        end for
    3. DK = PBKDF2-HMAC-SHA256 (P, B[0] || B[1] || ... || B[p - 1], 
                                        1, dkLen)
```


挖矿算法中，选取的参数为：

- P：区块头；
- S：区块头；
- N：固定为1024；
- r：固定为1；
- p：固定为1；
- dkLen：固定为32，即输出长度为32个字节。

因此，莱特币的区块头哈希值为 powhash = scrypt(blockheader, blockheader, 1024, 1, 1, 32)。可以参考莱特币获取区块头哈希的Go语言版本实现。

```go
// Copyright (c) 2013-2016 The btcsuite developers
// Use of this source code is governed by an ISC
// license that can be found in the LICENSE file.

package wire

import (
	"bytes"
	"io"
	"time"

	"golang.org/x/crypto/scrypt"

	"github.com/ltcsuite/ltcd/chaincfg/chainhash"
)

// MaxBlockHeaderPayload is the maximum number of bytes a block header can be.
// Version 4 bytes + Timestamp 4 bytes + Bits 4 bytes + Nonce 4 bytes +
// PrevBlock and MerkleRoot hashes.
const MaxBlockHeaderPayload = 16 + (chainhash.HashSize * 2)

// BlockHeader defines information about a block and is used in the bitcoin
// block (MsgBlock) and headers (MsgHeaders) messages.
type BlockHeader struct {
	// Version of the block.  This is not the same as the protocol version.
	Version int32

	// Hash of the previous block header in the block chain.
	PrevBlock chainhash.Hash

	// Merkle tree reference to hash of all transactions for the block.
	MerkleRoot chainhash.Hash

	// Time the block was created.  This is, unfortunately, encoded as a
	// uint32 on the wire and therefore is limited to 2106.
	Timestamp time.Time

	// Difficulty target for the block.
	Bits uint32

	// Nonce used to generate the block.
	Nonce uint32
}

// blockHeaderLen is a constant that represents the number of bytes for a block
// header.
const blockHeaderLen = 80

// BlockHash computes the block identifier hash for the given block header.
func (h *BlockHeader) BlockHash() chainhash.Hash {
	// Encode the header and double sha256 everything prior to the number of
	// transactions.  Ignore the error returns since there is no way the
	// encode could fail except being out of memory which would cause a
	// run-time panic.
	buf := bytes.NewBuffer(make([]byte, 0, MaxBlockHeaderPayload))
	_ = writeBlockHeader(buf, 0, h)

	return chainhash.DoubleHashH(buf.Bytes())
}

// PowHash returns the litecoin scrypt hash of this block header. This value is
// used to check the PoW on blocks advertised on the network.
func (h *BlockHeader) PowHash() (*chainhash.Hash, error) {
	var powHash chainhash.Hash

	buf := bytes.NewBuffer(make([]byte, 0, MaxBlockHeaderPayload))
	_ = writeBlockHeader(buf, 0, h)

	scryptHash, err := scrypt.Key(buf.Bytes(), buf.Bytes(), 1024, 1, 1, 32)
	if err != nil {
		return nil, err
	}
	copy(powHash[:], scryptHash)

	return &powHash, nil
}

// BtcDecode decodes r using the bitcoin protocol encoding into the receiver.
// This is part of the Message interface implementation.
// See Deserialize for decoding block headers stored to disk, such as in a
// database, as opposed to decoding block headers from the wire.
func (h *BlockHeader) BtcDecode(r io.Reader, pver uint32, enc MessageEncoding) error {
	return readBlockHeader(r, pver, h)
}

// BtcEncode encodes the receiver to w using the bitcoin protocol encoding.
// This is part of the Message interface implementation.
// See Serialize for encoding block headers to be stored to disk, such as in a
// database, as opposed to encoding block headers for the wire.
func (h *BlockHeader) BtcEncode(w io.Writer, pver uint32, enc MessageEncoding) error {
	return writeBlockHeader(w, pver, h)
}

// Deserialize decodes a block header from r into the receiver using a format
// that is suitable for long-term storage such as a database while respecting
// the Version field.
func (h *BlockHeader) Deserialize(r io.Reader) error {
	// At the current time, there is no difference between the wire encoding
	// at protocol version 0 and the stable long-term storage format.  As
	// a result, make use of readBlockHeader.
	return readBlockHeader(r, 0, h)
}

// Serialize encodes a block header from r into the receiver using a format
// that is suitable for long-term storage such as a database while respecting
// the Version field.
func (h *BlockHeader) Serialize(w io.Writer) error {
	// At the current time, there is no difference between the wire encoding
	// at protocol version 0 and the stable long-term storage format.  As
	// a result, make use of writeBlockHeader.
	return writeBlockHeader(w, 0, h)
}

// NewBlockHeader returns a new BlockHeader using the provided version, previous
// block hash, merkle root hash, difficulty bits, and nonce used to generate the
// block with defaults for the remaining fields.
func NewBlockHeader(version int32, prevHash, merkleRootHash *chainhash.Hash,
	bits uint32, nonce uint32) *BlockHeader {

	// Limit the timestamp to one second precision since the protocol
	// doesn't support better.
	return &BlockHeader{
		Version:    version,
		PrevBlock:  *prevHash,
		MerkleRoot: *merkleRootHash,
		Timestamp:  time.Unix(time.Now().Unix(), 0),
		Bits:       bits,
		Nonce:      nonce,
	}
}

// readBlockHeader reads a bitcoin block header from r.  See Deserialize for
// decoding block headers stored to disk, such as in a database, as opposed to
// decoding from the wire.
func readBlockHeader(r io.Reader, pver uint32, bh *BlockHeader) error {
	return readElements(r, &bh.Version, &bh.PrevBlock, &bh.MerkleRoot,
		(*uint32Time)(&bh.Timestamp), &bh.Bits, &bh.Nonce)
}

// writeBlockHeader writes a bitcoin block header to w.  See Serialize for
// encoding block headers to be stored to disk, such as in a database, as
// opposed to encoding for the wire.
func writeBlockHeader(w io.Writer, pver uint32, bh *BlockHeader) error {
	sec := uint32(bh.Timestamp.Unix())
	return writeElements(w, bh.Version, &bh.PrevBlock, &bh.MerkleRoot,
		sec, bh.Bits, bh.Nonce)
}
```

## POA共识算法

### POA产生的背景

如果你想使用Simplechain搭建一个联盟/私有链, 并要求该链交易成本更低甚至没有, 交易延时更低,并发更高, 还拥有完全的控制权(意味着被攻击概率更低). 目前Simplechain采用PoW是不能满足要求的。

- 首先, pow存在51%攻击问题, 恶意挖矿者超过全网算力的51%后基本上就能完全控制整个网络. 由于链无法被更改, 已上链的数据也无法更改, 但恶意挖矿者也可以做一些DoS攻击阻止合法交易上链,考虑到具有相同创世块的旷工都能加入你的网络, 潜在的安全隐患会长期存在.

- 其次, PoW大量的电力资源消耗也是需要作为后续成本考虑. PoS可以解决部分Pow问题, 比如节约电力,在一定程度上保护了51％的攻击(恶意旷工会被惩罚), 但从控制权和安全考虑还有欠缺, 因为PoS还是允许任何符合条件的旷工加入。

### POA的特点

- PoA是依靠预设好的授权节点(signers)，负责产生block.
- 可以由已授权的signer选举(投票超过50%)加入新的signer。
- 即使存在恶意signer,他最多只能攻击连续块(数量是 (SIGNER_COUNT / 2) + 1) 中的1个,期间可以由其他signer投票踢出该恶意signer。
- 可指定产生block的时间。


### POA的工作流程如下

1. 在创世块中指定一组初始授权的signers, 所有地址 保存在创世块Extra字段中
2. 启动挖矿后, 该组signers开始对生成的block进行 签名并广播.
3. 签名结果 保存在区块头的Extra字段中
4. Extra中更新当前高度已授权的 所有signers的地址 ,因为有新加入或踢出的signer
5. 每一高度都有一个signer处于IN-TURN状态, 其他signer处于OUT-OF-TURN状态, IN-TURN的signer签名的block会 立即广播 , OUT-OF-TURN的signer签名的block会 延时 一点随机时间后再广播, 保证IN-TURN的签名block有更高的优先级上链
6. 如果需要加入一个新的signer, signer通过API接口发起一个proposal, 该proposal通过复用区块头 Coinbase(新signer地址)和Nonce("0xffffffffffffffff") 字段广播给其他节点. 所有已授权的signers对该新的signer进行"加入"投票, 如果赞成票超过signers总数的50%, 表示同意加入
7. 如果需要踢出一个旧的signer, 所有已授权的signers对该旧的signer进行"踢出"投票, 如果赞成票超过signers总数的50%, 表示同意踢出

### Simplechain中如何选择POA共识算法

下载生成新的genesis区块文件，[下载地址](https://github.com/huangxinglong/picture/raw/master/puppeth)

下载完成以后，通过命令执行`生成创世区块`的文件,在生成创世区块文件的过程中，可以选择自己想要的共识算法。如下图：

![W4f.png](1.png)
