





# stateTree





# Runtime





# 区块struct

FullBlock-github.com/filecoin-project/lotus/chain/types.FullBlock

```go
package types

import "github.com/ipfs/go-cid"

type FullBlock struct {
	Header        *BlockHeader
	BlsMessages   []*Message
	SecpkMessages []*SignedMessage
}

func (fb *FullBlock) Cid() cid.Cid {
	return fb.Header.Cid()
}
```



```go
func (b *BlockHeader) Cid() cid.Cid {
	sb, err := b.ToStorageBlock()

	return sb.Cid()
}

func (b *BlockHeader) ToStorageBlock() (block.Block, error) {
	data, err := b.Serialize()

	return github.com/ipfs/go-block-format.block.NewBlockWithCid(data)
}
```

BlockHeader的经过编辑的摘录显示了如何将其视为IPFS块github.com/ipfs/go-block-format.block.BasicBlock，以其块存储CID进行存储和引用。不合格的块是为Filecoin块保留的，我们通常不会引用IPFS，而是仅通过其CID的概念隐式地引用它。我们将抽象掉IPFS层，仅使用CID作为标识符，对于两个不同的原始字节字符串序列，我们现在将其唯一。

来自同一round的消息会被收集到一个block set。（chain/store/fts.go）

```go
type FullTipSet struct {
	Blocks []*types.FullBlock
	tipset *types.TipSet
	cids   []cid.Cid
}
```







# Chain模块

/chain















