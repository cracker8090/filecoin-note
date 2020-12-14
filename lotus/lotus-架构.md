



## component组件

与schomatis's miner 文档进行了交叉引用

- Syncer：处理链同步
- State Manager：计算链中任何给定点point的状态
- VM（virtual Machine）：执行消息
- Repository：所有数据被存储
- P2P：（libp2p）允许hello，blocksync，检索(retrieval)，存储
- API/CLI：（在便签中）
- 其他filecoin依赖：specs actors,proofs,storage等 
- 构建者值得拥有自己的组件部分吗
- 其他PL依赖：IPFS、libp2p、IPLD等
- 在lotus中和其他依赖使用的库



## filecoin关键概念

主要是为了区分已有的区块链，入ethereum，详细可参考[spec](https://filecoin-project.github.io/specs/) 



### Tipsets

与以太坊不同，filecoin中一个区块可以有多个parents，因此，我们指的是块的父集，而不是单亲。tipset是共享相同父集的任何块集。

filecoin中没有块难度，相反，tipset的重量（权重）是指该tipset中终止（结束）链中的区块数。请注意，较长链的重量比较短链的重量小，每个tipset包含更多的块。



允许使用“ null” tipsets（包括零块）。这允许矿工可以“跳过”一轮，并根据需要在空tipset的基础上构建。

将链条中最重的tipset称为链条的“头”



### Actors and message

一个actor类似于以太坊中的智能合约，filecoin中不允许用户定义自己的actors，但带有多个内置角色，可以将其视为预编译的合同。

消息类似于以太坊中的交易。



### Sync

指lotus全节点被其他peers告知同步最重的链。总的来说，lotus的同步和其他区块链的同步方式类似。

Lotus节点侦听在各个链中的peers并选择最重的链，请求所选链中的块并验证该链中的每个块，并在此过程中运行所有状态转换。

**主要函数：chain/sync.go,内部管理在chain/sync_manager.go** 

同步各个阶段：

#### sync setup

lotus全节点连接一个新的peer，通过hello协议与new peer交换链的头部，如果这个peer的头比我们的重，我们尝试去同步它。请注意，我们现阶段不更新链头。







































































