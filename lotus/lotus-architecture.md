目录

[toc] 

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

---

## filecoin关键概念

主要是为了区分已有的区块链，入ethereum，详细可参考[spec](https://filecoin-project.github.io/specs/) 



### Tipsets

与以太坊不同，filecoin中一个区块可以有多个parents，因此，我们指的是块的父集，而不是单亲。tipset是共享相同父集的任何块集。

filecoin中没有块难度，相反，tipset的重量（权重）是指该tipset中终止（结束）链中的区块数。请注意，较长链的重量比较短链的重量小，每个tipset包含更多的块。



允许使用“ null” tipsets（包括零块）。这允许矿工可以“跳过”一轮，并根据需要在空tipset的基础上构建。

将链条中最重的tipset称为链条的“头”

---

### Actors and message

一个[actor](https://filecoin-project.github.io/specs/#systems__filecoin_vm__actor)类似于以太坊中的智能合约，filecoin中不允许用户定义自己的actors，但带有多个[builtin actors](https://github.com/filecoin-project/specs-actors)，可以将其视为预编译的合同。

[消息](https://filecoin-project.github.io/specs/#systems__filecoin_vm__message)类似于以太坊中的交易。

（合约账户不能自己启动运行自己的智能合约。要运行一个智能合约，需要由外部账户对合约账户发起交易，从而启动其中的代码的执行。）

---

### Sync

Gossip sub spec 

指lotus全节点被其他peers告知同步最重的链。总的来说，lotus的同步和其他区块链的同步方式类似。

Lotus节点侦听在各个链中的peers并选择最重的链，请求所选链中的块并验证该链中的每个块，并在此过程中运行所有状态转换。

**主要函数：chain/sync.go,内部管理在chain/sync_manager.go** 。[syncer](https://github.com/filecoin-project/lotus/blob/master/chain/sync.go) ，[syncManager](https://github.com/filecoin-project/lotus/blob/master/chain/sync_manager.go) 

同步各个阶段：

#### sync setup

lotus全节点连接一个新的peer，通过[hello协议](https://github.com/filecoin-project/lotus/blob/master/node/hello/hello.go)与new peer交换链的头部，如果这个peer的头比我们的重，我们尝试去同步它。请注意，我们现阶段不更新链头。

#### Fetching and Persisting Block Headers

提取并保留块头。API将这些阶段称为StageHeaders和StagePersistHeaders

在同步过程中，我们从peer请求块头，然后从它们的头移回，直到达到我们共有的tipset（这样的通用tipset必须存在，尽管可能是创世块）。该功能在**Syncer :: collectHeaders（）**中。

如果common tipset是我们的head，则将此同步称为“fast-forward”，否则我们必须丢弃一部分链以连接到peer的head（称为“fork”，分叉）

最好用指向验证文档的链接替换下一个para，此阶段失败的一些可能原因包括：

- 该链链接到我们先前标记的bad block，并存储在[BadBlockCache](https://github.com/filecoin-project/lotus/blob/master/chain/badtscache.go)中。
- 块中的beacon entries不一致。
- 切换到该新链将涉及链重组，超出允许阈值（SPECK-CHECK）。

#### Fetching and Validating Blocks

拉取和校验块。API将此阶段称为StageMessages。

**Syncer :: ValidateTipset（）**；**Syncer :: checkBlockMessages（）**执行消息的语法验证。

获取heads并找到一个common tipset，我们继续向前请求full blocks，包括消息。

对于每个块，我们首先确认该块的syntactic有效性（SPECK-CHECK），其中包括该块中消息的syntactic有效性。然后在这些消息中，执行所有(stat transition)状态转换，并将我们计算出的(stat root)状态根与提供的(stat root)状态根进行比较。

最好用指向验证文档的链接替换下一个para，此阶段失败的一些可能原因包括：

- 一个块syntactic无效（包括消息的syntactic无效）
- 应用该块后计算出的(stat root)状态根与该块的(stat root)状态根不匹配
- 检查语法(syntactic)有效性涵盖的内容，并添加不重要的内容（例如证明有效性，future checks等）



#### Setting the head

API将此阶段称为StageSyncComplete

如果所有校验通过了，我们就将其设置为[ChainStore](https://github.com/filecoin-project/lotus/blob/master/chain/store/store.go)中最重的tipset。我们已经有了full stat，因为在同步处理时已经计算过了。

不理解下面的两段，但是很重要，[IPFS相关问题](https://github.com/ipfs/ipfs-docs/issues/264) 

类似于IPFS体系结构，按内容而不是按位置/地址寻址（IPFS相关文档），在节点库（node repo）中的“真实”（actual）链于我们寻找的CID有关。我们始终存储了一系列Filecoin块，这些块指向其他块，每个块通过遵循其父代的引用以及其父代的父代，本身就是一条潜在的链，依此类推，直到创世块为止（filecoin博客有类似图）。这仅取决于我们开始寻找的位置。我们持有的链中唯一的地址置参考（相对参考）是最重的指针。这反映在以下事实上：我们不会通过反映其内容的固定的绝对CID将其存储在Blockstore中，因为每次我们同步到新的head时，它都会发生变化。

我们通过key，location存储到数据存储（dataStore）中，允许其内容在每次同步时都进行更改。这反映在（* ChainStore）writeHead（）函数（由上面的takeHeaviestTipSet（）调用）中，在该函数中，我们通过显式chainHeadKey地址（字符串'head'，而不是CID中嵌入的散列）引用指针，并且在（* ChainStore）.Load（），当我们启动节点并创建ChainStore时。将其与不可变的Filecoin块或消息进行比较，这些消息或消息由CID存储在Blockstore中，一旦创建，它们就永远不会改变。

#### Keeping up with the chain

追赶chain——同步chain

Lotus全节点还通过gossipsub通道侦听peers广播的新块。如果我们已经验证了一个块的父tipset，并且将这个块添加到我们的tipset的高度时会导致head变重，那么我们将验证并添加此块。描述的验证与同步过程中调用的验证相同（实际上，它是相同的代码路径）。

---



### State

HAMT，创世区块创建根节点。

在Filecoin中，任何给定point的链state是数据的集合，存储在root CID下，root CID封装在[StateTree](https://github.com/filecoin-project/lotus/blob/master/chain/store/store.go)中，通过[StateManager](https://github.com/filecoin-project/lotus/blob/master/chain/stmgr/stmgr.go)进行访问。因此，可以在状态 root CID中很容易跟踪和更新链头的状态。 （在某处谈论CID，我们可能要在这里解释一些modify / flush / update-root机制。）

---

#### Calculating a Tipset State

计算一个tipset state

一个tipset是一组有相同parents的块（即，在相同tipset之上构建的块）。创世tipset包括创世区块，并具有与之对应的某种state。

[StateManager](https://github.com/filecoin-project/lotus/blob/master/chain/stmgr/stmgr.go) 中的TipSetState（）和computeTipSetState（）方法负责计算由于应用tipset而产生的状态。这涉及应用tipset中包含的所有消息，并执行隐式操作，例如授予区块奖励（block reward）。

建立在tipset ts之上的任何有效区块的Parent State Root应等于计算tipset ts的state结果。这意味着一个tipset中的所有块都必须具有相同的Parent State Root（这是可以预期的，因为它们具有相同的parent tipset）。

##### Preparing to apply a tipset

准备apply tipset

​		当一个tipset、ts调用**StateManager :: computeTipsetState**（）时，它将检索区块的父状态根（parent state root）。它还创建一个BlockMessages列表，该列表将BLS和SecP消息与产生该块的矿工一起包装在一个块中。

​		控制流向**StateManager :: ApplyBlocks（）**，将构建一个VM，以应用提供给它的消息。使用ts中块的父状态根（parent state root）来初始化VM。我们按ts顺序应用块（有关一个tipset中blocks的排序，请参见FIXME）





##### Applying a block

​		对于每个区块，我们准备应用有序消息（首先是BLS，然后SecP）。在应用一个消息之前，我们检查是否已经在此方法范围内应用了具有该CID的消息，如果是这样的话，我们只是简单跳过该消息；这就是跳过在同一个tipset中重复消息的方式（只有“first”区块的矿工包含获得奖励的消息）。有关消息应用程序的实际过程，请参阅FIXME。现在我们仅假设VM应用消息的结果是error或[MessageReceipt](https://github.com/filecoin-project/lotus/blob/master/chain/types/message_receipt.go)和其他信息。

我们将来自VM的错误视为“失败”，没有恢复，也无法为ts计算有意义的状态。给定成功的收据，我们会将奖励和惩罚添加到该矿工迄今已赚取的收入中。一旦应用了块中包含的所有消息（如果它们是重复消息，则跳过该消息），我们将使用隐式消息来调用Reward Actor。这会奖励矿工赢得block的奖励，并根据我们跟踪的消息奖励和惩罚来奖励/惩罚他们。

然后，我们使用相同的VM在ts中应用下一个块。这意味着，应用所有后续消息时，即使它们包含在不同的块中，也可以看到由于应用消息而导致的状态更改。

##### Finishing up

应用完所有块后，我们再向Cron Actor发送一条隐式消息，该消息处理必须在每个epoch结束时执行的操作。调用Cron Actor之后的结果状态是tipset的计算状态。

### Virtual Machine

VM负责执行消息，[lotus VM](https://github.com/filecoin-project/lotus/blob/master/chain/vm/vm.go)会调用内置（builtin）actor中的适当方法，并为[builtin actor](https://github.com/filecoin-project/specs-actors)提供[runtime](https://github.com/filecoin-project/specs-actors/blob/master/actors/runtime/runtime.go)接口，以暴露其状态，允许他们采取某些措施并计量其gas使用。VM也执行balance transfer、根据需要创建新的帐户参与者（actors）、跟踪gas奖励、惩罚、返回值和exit code。

#### Applying a Message

VM的主要入口点是ApplyMessage（）方法。除非出现无法恢复的错误，否则此方法不应返回错误。

此方法要做的第一件事是评估所提供的消息是否符合任何惩罚标准。如果符合的话，则发出罚款，并且该方法返回。接下来，该消息的全部gas cost将转移到一个临时gas holder account（临时gas持有者账户），后面从这临时账户中扣除gas。如果gas用完，则消息失败。消息执行结束后这个临时账户中未使用的gas将退还给消息发送者。

VM增加一个sender随机数，拍摄一个状态快照，然后条用VM::send()。

send（）方法为随后的消息执行创建[Runtime](https://github.com/filecoin-project/lotus/blob/master/chain/vm/runtime.go) ，然后，它将消息值转移给接收者，并在需要时创建一个新的帐户actor。

#### Method Invocation

方法调用

我们依赖于VM的[invoker](https://github.com/filecoin-project/lotus/blob/master/chain/vm/invoker.go)结构，使用反射将VM的Filecoin消息转换为实际的Go函数。每个actor在specs-actors/actors/builtin/methods.go中定义了自己的一组代码。invoker structure将 builtin actors的CIDs映射到invokerFunc列表（每个导出方法一个），每个都使用运行时（用于状态操作）和序列化的输入参数。

**（* invoker）.transform（）**的基本布局（不包含反射细节）如下。

从在`NewInvoker（）`中注册的每个actor中，我们用其`Exports（）`方法将它们转换为`invokeFuncs`。实际的方法包装在另一个函数中，该函数负责对序列化的参数和runtime进行解码，该函数传递给`shimCall（）`，该函数将封装在defer函数中运行的actor代码，以从panic中`recover()`（actors panic则会解开堆栈）。然后返回值将marshaled（CBOR）并返回到VM。

#### Returning from the VM

一旦方法调用完成（包括所有子调用）,将返回`ApplyMessage()` ,将收到序列号响应和[ActorError](https://github.com/filecoin-project/lotus/blob/master/chain/actors/aerrors/error.go) 。发送者将被收取适当数量的gas，用于返回的响应，该响应将被放入[MessageReceipt](https://github.com/filecoin-project/lotus/blob/master/chain/types/message_receipt.go)中。

然后，该方法将任何未使用的气体退还给发送方，为矿工设置气体奖励，并将所有这些都包装到`ApplyRet`中，然后将其返回。



### Building a Lotus node

建立一个lotus节点

./lotus daemon（[here](https://github.com/filecoin-project/lotus/blob/master/cmd/lotus/daemon.go)），将通过[依赖注入](https://godoc.org/go.uber.org/fx)来创建该节点。这依赖于反射，这使得某些参考文献难以理解。节点设置它需要运行的所有子系统，例如存储库，网络连接，链同步服务等。此设置通过对`node.Override`函数的调用进行编排。每个调用的结构指示将要设置的组件的类型（许多定义在[node / modules / dtypes /](https://github.com/filecoin-project/lotus/tree/master/node/modules/dtypes)中）以及将提供该组件的功能。依赖关系在提供程序函数的参数中是隐式的。

例如，考虑提供[ChainStore](https://github.com/filecoin-project/lotus/blob/master/chain/store/store.go)结构的modules.ChainStore（）函数。它采用[ChainBlockstore](https://github.com/filecoin-project/lotus/blob/master/node/modules/dtypes/storage.go)类型作为其参数之一，该类型成为其依赖项之一。为了成功构建节点，需要在ChainStore之前提供ChainBlockstore，此要求在另一个Override（）调用中明确提出，该调用将该类型的提供程序设置为ChainBlockstore（）函数。



`node.Repo（）`函数（node / builder.go）包含正确设置节点存储库所需的大多数依赖项（指定为`Override（）`调用）。我们在这里列出最突出的

##### Datastore

`Datastore`和`ChainBlockstore`：与节点状态相关的数据保存在存储库的`Datastore`（此处定义了IPFS接口）中。 Lotus从[FsRepo](https://github.com/filecoin-project/lotus/blob/master/node/repo/fsrepo.go)中的[Badger DB](https://github.com/dgraph-io/badger)创建此接口。从根本上说，每条数据都是仓库的数据存储目录中的键值对。根据我们访问它的方式，在代码之上会出现一些抽象，但是要记住，我们总是从同一位置访问它，这一点很重要。

##### Blocks

[Blockstore interface](https://github.com/filecoin-project/lotus/blob/master/documentation/en/architecture/%60github.com/filecoin-project/lotus/lib/blockstore.go%60)将键值对构造为键的CID格式和值的Block接口。 Block值只是其散列所寻址的原始字节串，该散列包含在CID密钥中。

`ChainBlockstore`在/ blocks命名空间下创建一个`Blockstore`。那里存储的每个密钥都将带有block pfrefix，这样它就不会与使用相同repo的其他stores发生冲突。

FIXME：链接到有关DAG，CID和相关文件的IPFS文档，特别是我们需要一个图来显示如何将每个数据存储包装在下一层（数据存储，批处理，块存储，gc等）中。

##### Metadata

modules.Datastore（）创建一个dtypes.MetadataDS，它是基本Datastore接口的别名。元数据存储在/ metadata前缀下。 

##### LockedRepo

`LockedRepo()`：此方法不会创建或初始化任何新结构，而是注册一个OnStop挂钩，该挂钩将在关闭时关闭与其关联的锁定存储库。

### Online

许多部分包含在p2p部分中

`node.Online（）`配置函数（`node / builder.go`）初始化涉及连接到Filecoin网络或与Filecoin网络交互的组件。这些连接通过libp2p协议栈进行管理。我们讨论了在完整节点类型中找到的一些组件（即，包含在ApplyIf（isType（repo.FullNode））。

#### Chainstore

`modules.ChainStore()` creates the [`store.ChainStore`](https://github.com/filecoin-project/lotus/blob/master/chain/store/store.go) 

它还具有至关重要的最重指针，该指针指示链的当前头部。

#### ChainExchange and ChainBlockservice

ChainExchange（）和ChainBlockservice（）建立一个BitSwap连接（FIXME libp2p链接）,以块形式交换链信息。



#### Incoming handlers

HandleIncomingBlocks（）和HandleIncomingMessages（）启动服务，负责处理来自网络的新Filecoin块和消息。属于libp2p部分,gossipsub。



#### Hello

`RunHello()`:启动服务以发送（（* Service）.SayHello（））和接收（（* Service）.HandleStream（），node / hello / hello.go）hello消息。当节点之间建立新连接时，它们交换这些消息以共享与链相关的信息（即，它们的创世区块和最重的tipset）。



#### Syncer

NewSyncer（）创建Syncer结构并启动与链同步过程相关的服务。



#### Ordering the dependencies

依赖顺序

例如，同步机制取决于节点能够与网络交换不同的IPFS块，以便能够请求构建链所需的“缺失部分”。此依赖关系通过具有`blocksync.BlockSync`参数的`NewSyncer（）`来反映，该参数又取决于ChainBlockservice（）和ChainExchange（）。链交换服务进一步依赖于链存储来保存和检索链数据，这反映在以ChainGCBlockstore作为参数的ChainExchange（）中（这只是能够进行garbage collection的ChainBlockstore的包装）。



block store和chain store基本相同，是NewSyncer（）的间接依赖关系（通过StateManager）。 



## Genesis block

创世区块

在哪里加载创世块，并设置 state root，遵循守护程序命令选项chain.LoadGenesis（）将CAR文件的所有块保存到ChainBlockstore提供的存储中。这些块的CAR root（MT root？）被解码为BlockHeader，它将成为链中的Filecoin（genesis）块。

名称为0的SetGenesis块。（ChainStore）.SetGenesis（）将其存储在那里。

`MakeInitialStateTree` (`chain/gen/genesis/genesis.go`, 构建创世区块MakeGenesisBlock()，构建状态树state tree (`NewStateTree`) 只是指向不同actors的一个pointer（HAMT的根节点）。它会在（Address）（在HAMT中）某个类型的（* StateTree）.SetActor（）中的type.Actor结构中连续使用。

- SetupInitActor(): see the AddressMap

- SetupStoragePowerActor： initial (zero) power state of the chain, most important attributes

- Account actors in the `template.Accounts`: `SetActor`



## 查看miner相关

按照lotus-miner命令查看如何创建矿工，从命令到消息再到存储power逻辑。



## 需要看的主要目录

chain，最重要的结构StateTree、Runtime等

BitSwap协议是指什么?

Filecoin block：github.com/filecoin-project/lotus/chain/types.FullBlock

除了filecoin消息外，还有与协议相关的信息（例如Height）的header。







## CLI/API

CLI，编程方式获取自己的工具。



### API

json-rpc，暴露的接口在FullNode接口（定义在api/api_full.go）

Lotus sync status的命令来查询节点同步的进度时，我们不访问节点的内部，而是在单独的守护进程中将它们解耦，我们通过RPC调用SyncState函数（FullNode API接口的）客户端由我们自己的命令启动（请参阅api / client / client.go中的NewFullNodeRPC）。

因为我们在这部分代码中严重依赖反射，所以仅通过对IDE的符号分析来跟踪引用，就很难轻易看到调用链。如果从`Lotus sync`命令定义（在`cli / sync.go`中）开始，我们最终将到达方法接口`SyncState`，当我们寻找其实现时，我们将找到两个函数。

- `(*SyncAPI).SyncState()` (in `node/impl/full/sync.go`): 它显示了节点（在此充当RPC服务器）在收到从充当客户端的CLI发出的RPC请求时将执行的操作。

- `(*FullNodeStruct).SyncState()`:这是一个“空的占位符”结构，以后将连接到JSON-RPC客户端逻辑（lib / jsonrpc / client.go中的NewMergeClient，由NewFullNodeRPC调用）。 CLI（JSON-RPC客户端）将实际执行此功能，该功能将连接到服务器并发送相应的JSON请求，该请求将触发节点实现对（* SyncAPI）.SyncState（）的调用。

当我们跟踪CLI命令的逻辑时，我们最终将发现这种分歧，并需要研究node / impl / full（主要在common /和full /目录中）的服务器端实现的代码。了解了这种代码体系，则会抽象出json-rpc客户端/服务器逻辑，并且我们可以认为CLI实际上正在运行节点的逻辑。

说明“节点”实际上是一个像`impl.FullNodeAPI`这样的API结构，具有不同的API子组件，例如`full.SyncAPI`。我们不会看到一个单一的节点结构，每个API（完整节点，监视者等）都将收集为其调用服务所需的必要子组件。



### CLI

可以通过lotus CLI连接到改服务器。

```shell
cat  ~/.lotus/api && echo
# With `lotus daemon` running in another terminal.
nc -v -z 127.0.0.1 1234

# Start daemon and turn off the logs to not clutter the command line.
bash -c "lotus daemon &" &&
  lotus wait-api &&
  lotus log set-level error # Or a env.var in the daemon command.

nc -v -z 127.0.0.1 1234
# Connection to 127.0.0.1 1234 port [tcp/*] succeeded!

killall lotus
# FIXME: We need a lotus stop command:
#  https://github.com/filecoin-project/lotus/issues/1827
```











































