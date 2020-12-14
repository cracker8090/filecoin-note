





# 代码文件

## 主要文件夹

```go
.
├── api		//主要是rpc调用的方法，类似于函数调用
├── build	//主要是编译需要的配置或者参数
├── chain	//链相关，同步相关消息
├── cli		//命令行相关函数
├── cmd		//主要是编译二进制文件，内包含lotus、miner、worker，还有其他工具的二进制
├── conformance
├── documentation
├── extern
├── gen
├── genesis
├── journal
├── lib
├── lotuspond
├── markets
├── metrics
├── miner
├── node
├── paychmgr
├── scripts
├── storage
├── system
├── testplans
└── tools

CHANGELOG.md、go.mod、go.sum、LICENSE-APACHE、LICENSE-MIT、Makefile、README.md、SECURITY.md
```



## api

### 结构

```go
.
├── apibstore
│   └── apibstore.go
├── api_common.go
├── api_full.go
├── api_gateway.go
├── api_storage.go
├── apistruct
│   ├── permissioned.go
│   ├── struct.go
│   └── struct_test.go
├── api_test.go
├── api_wallet.go
├── api_worker.go
├── cbor_gen.go
├── client
│   └── client.go
├── docgen
│   └── docgen.go
├── test
│   ├── blockminer.go
│   ├── ccupgrade.go
│   ├── deals.go
│   ├── mining.go
│   ├── paych.go
│   ├── tape.go
│   ├── test.go
│   ├── util.go
│   └── window_post.go
├── types.go
└── utils.go
	
```

### api_full.go

```go
type FullNode interface {
   Common

   // MethodGroup: Chain
   // The Chain method group contains methods for interacting with the
   // blockchain, but that do not require any form of state computation.

   // ChainNotify returns channel with chain head updates.
   // First message is guaranteed to be of len == 1, and type == 'current'.
   ChainNotify(context.Context) (<-chan []*HeadChange, error)

   // ChainHead returns the current head of the chain.
   ChainHead(context.Context) (*types.TipSet, error)

   // ChainGetRandomnessFromTickets is used to sample the chain for randomness.
   ChainGetRandomnessFromTickets(ctx context.Context, tsk types.TipSetKey, personalization crypto.DomainSeparationTag, randEpoch abi.ChainEpoch, entropy []byte) (abi.Randomness, error)

   // ChainGetRandomnessFromBeacon is used to sample the beacon for randomness.
   ChainGetRandomnessFromBeacon(ctx context.Context, tsk types.TipSetKey, personalization crypto.DomainSeparationTag, randEpoch abi.ChainEpoch, entropy []byte) (abi.Randomness, error)

   // ChainGetBlock returns the block specified by the given CID.
   ChainGetBlock(context.Context, cid.Cid) (*types.BlockHeader, error)
   // ChainGetTipSet returns the tipset specified by the given TipSetKey.
   ChainGetTipSet(context.Context, types.TipSetKey) (*types.TipSet, error)

   // ChainGetBlockMessages returns messages stored in the specified block.
   ChainGetBlockMessages(ctx context.Context, blockCid cid.Cid) (*BlockMessages, error)

   // ChainGetParentReceipts returns receipts for messages in parent tipset of
   // the specified block.
   ChainGetParentReceipts(ctx context.Context, blockCid cid.Cid) ([]*types.MessageReceipt, error)

   // ChainGetParentMessages returns messages stored in parent tipset of the
   // specified block.
   ChainGetParentMessages(ctx context.Context, blockCid cid.Cid) ([]Message, error)

   // ChainGetTipSetByHeight looks back for a tipset at the specified epoch.
   // If there are no blocks at the specified epoch, a tipset at an earlier epoch
   // will be returned.
   ChainGetTipSetByHeight(context.Context, abi.ChainEpoch, types.TipSetKey) (*types.TipSet, error)

   // ChainReadObj reads ipld nodes referenced by the specified CID from chain
   // blockstore and returns raw bytes.
   ChainReadObj(context.Context, cid.Cid) ([]byte, error)

   // ChainDeleteObj deletes node referenced by the given CID
   ChainDeleteObj(context.Context, cid.Cid) error

   // ChainHasObj checks if a given CID exists in the chain blockstore.
   ChainHasObj(context.Context, cid.Cid) (bool, error)

   // ChainStatObj returns statistics about the graph referenced by 'obj'.
   // If 'base' is also specified, then the returned stat will be a diff
   // between the two objects.
   ChainStatObj(ctx context.Context, obj cid.Cid, base cid.Cid) (ObjStat, error)

   // ChainSetHead forcefully sets current chain head. Use with caution.
   ChainSetHead(context.Context, types.TipSetKey) error

   // ChainGetGenesis returns the genesis tipset.
   ChainGetGenesis(context.Context) (*types.TipSet, error)

   // ChainTipSetWeight computes weight for the specified tipset.
   ChainTipSetWeight(context.Context, types.TipSetKey) (types.BigInt, error)
   ChainGetNode(ctx context.Context, p string) (*IpldObject, error)

   // ChainGetMessage reads a message referenced by the specified CID from the
   // chain blockstore.
   ChainGetMessage(context.Context, cid.Cid) (*types.Message, error)

   // ChainGetPath returns a set of revert/apply operations needed to get from
   // one tipset to another, for example:
   //```
   //        to
   //         ^
   // from   tAA
   //   ^     ^
   // tBA    tAB
   //  ^---*--^
   //      ^
   //     tRR
   //```
   // Would return `[revert(tBA), apply(tAB), apply(tAA)]`
   ChainGetPath(ctx context.Context, from types.TipSetKey, to types.TipSetKey) ([]*HeadChange, error)

   // ChainExport returns a stream of bytes with CAR dump of chain data.
   // The exported chain data includes the header chain from the given tipset
   // back to genesis, the entire genesis state, and the most recent 'nroots'
   // state trees.
   // If oldmsgskip is set, messages from before the requested roots are also not included.
   ChainExport(ctx context.Context, nroots abi.ChainEpoch, oldmsgskip bool, tsk types.TipSetKey) (<-chan []byte, error)

   // MethodGroup: Beacon
   // The Beacon method group contains methods for interacting with the random beacon (DRAND)

   // BeaconGetEntry returns the beacon entry for the given filecoin epoch. If
   // the entry has not yet been produced, the call will block until the entry
   // becomes available
   BeaconGetEntry(ctx context.Context, epoch abi.ChainEpoch) (*types.BeaconEntry, error)

   // GasEstimateFeeCap estimates gas fee cap
   GasEstimateFeeCap(context.Context, *types.Message, int64, types.TipSetKey) (types.BigInt, error)

   // GasEstimateGasLimit estimates gas used by the message and returns it.
   // It fails if message fails to execute.
   GasEstimateGasLimit(context.Context, *types.Message, types.TipSetKey) (int64, error)

   // GasEstimateGasPremium estimates what gas price should be used for a
   // message to have high likelihood of inclusion in `nblocksincl` epochs.

   GasEstimateGasPremium(_ context.Context, nblocksincl uint64,
      sender address.Address, gaslimit int64, tsk types.TipSetKey) (types.BigInt, error)

   // GasEstimateMessageGas estimates gas values for unset message gas fields
   GasEstimateMessageGas(context.Context, *types.Message, *MessageSendSpec, types.TipSetKey) (*types.Message, error)

   // MethodGroup: Sync
   // The Sync method group contains methods for interacting with and
   // observing the lotus sync service.

   // SyncState returns the current status of the lotus sync system.
   SyncState(context.Context) (*SyncState, error)

   // SyncSubmitBlock can be used to submit a newly created block to the.
   // network through this node
   SyncSubmitBlock(ctx context.Context, blk *types.BlockMsg) error

   // SyncIncomingBlocks returns a channel streaming incoming, potentially not
   // yet synced block headers.
   SyncIncomingBlocks(ctx context.Context) (<-chan *types.BlockHeader, error)

   // SyncCheckpoint marks a blocks as checkpointed, meaning that it won't ever fork away from it.
   SyncCheckpoint(ctx context.Context, tsk types.TipSetKey) error

   // SyncMarkBad marks a blocks as bad, meaning that it won't ever by synced.
   // Use with extreme caution.
   SyncMarkBad(ctx context.Context, bcid cid.Cid) error

   // SyncUnmarkBad unmarks a blocks as bad, making it possible to be validated and synced again.
   SyncUnmarkBad(ctx context.Context, bcid cid.Cid) error

   // SyncUnmarkAllBad purges bad block cache, making it possible to sync to chains previously marked as bad
   SyncUnmarkAllBad(ctx context.Context) error

   // SyncCheckBad checks if a block was marked as bad, and if it was, returns
   // the reason.
   SyncCheckBad(ctx context.Context, bcid cid.Cid) (string, error)

   // SyncValidateTipset indicates whether the provided tipset is valid or not
   SyncValidateTipset(ctx context.Context, tsk types.TipSetKey) (bool, error)

   // MethodGroup: Mpool
   // The Mpool methods are for interacting with the message pool. The message pool
   // manages all incoming and outgoing 'messages' going over the network.

   // MpoolPending returns pending mempool messages.
   MpoolPending(context.Context, types.TipSetKey) ([]*types.SignedMessage, error)

   // MpoolSelect returns a list of pending messages for inclusion in the next block
   MpoolSelect(context.Context, types.TipSetKey, float64) ([]*types.SignedMessage, error)

   // MpoolPush pushes a signed message to mempool.
   MpoolPush(context.Context, *types.SignedMessage) (cid.Cid, error)

   // MpoolPushUntrusted pushes a signed message to mempool from untrusted sources.
   MpoolPushUntrusted(context.Context, *types.SignedMessage) (cid.Cid, error)

   // MpoolPushMessage atomically assigns a nonce, signs, and pushes a message
   // to mempool.
   // maxFee is only used when GasFeeCap/GasPremium fields aren't specified
   //
   // When maxFee is set to 0, MpoolPushMessage will guess appropriate fee
   // based on current chain conditions
   MpoolPushMessage(ctx context.Context, msg *types.Message, spec *MessageSendSpec) (*types.SignedMessage, error)

   // MpoolBatchPush batch pushes a signed message to mempool.
   MpoolBatchPush(context.Context, []*types.SignedMessage) ([]cid.Cid, error)

   // MpoolBatchPushUntrusted batch pushes a signed message to mempool from untrusted sources.
   MpoolBatchPushUntrusted(context.Context, []*types.SignedMessage) ([]cid.Cid, error)

   // MpoolBatchPushMessage batch pushes a unsigned message to mempool.
   MpoolBatchPushMessage(context.Context, []*types.Message, *MessageSendSpec) ([]*types.SignedMessage, error)

   // MpoolGetNonce gets next nonce for the specified sender.
   // Note that this method may not be atomic. Use MpoolPushMessage instead.
   MpoolGetNonce(context.Context, address.Address) (uint64, error)
   MpoolSub(context.Context) (<-chan MpoolUpdate, error)

   // MpoolClear clears pending messages from the mpool
   MpoolClear(context.Context, bool) error

   // MpoolGetConfig returns (a copy of) the current mpool config
   MpoolGetConfig(context.Context) (*types.MpoolConfig, error)
   // MpoolSetConfig sets the mpool config to (a copy of) the supplied config
   MpoolSetConfig(context.Context, *types.MpoolConfig) error

   // MethodGroup: Miner

   MinerGetBaseInfo(context.Context, address.Address, abi.ChainEpoch, types.TipSetKey) (*MiningBaseInfo, error)
   MinerCreateBlock(context.Context, *BlockTemplate) (*types.BlockMsg, error)

   // // UX ?

   // MethodGroup: Wallet

   // WalletNew creates a new address in the wallet with the given sigType.
   // Available key types: bls, secp256k1, secp256k1-ledger
   // Support for numerical types: 1 - secp256k1, 2 - BLS is deprecated
   WalletNew(context.Context, types.KeyType) (address.Address, error)
   // WalletHas indicates whether the given address is in the wallet.
   WalletHas(context.Context, address.Address) (bool, error)
   // WalletList lists all the addresses in the wallet.
   WalletList(context.Context) ([]address.Address, error)
   // WalletBalance returns the balance of the given address at the current head of the chain.
   WalletBalance(context.Context, address.Address) (types.BigInt, error)
   // WalletSign signs the given bytes using the given address.
   WalletSign(context.Context, address.Address, []byte) (*crypto.Signature, error)
   // WalletSignMessage signs the given message using the given address.
   WalletSignMessage(context.Context, address.Address, *types.Message) (*types.SignedMessage, error)
   // WalletVerify takes an address, a signature, and some bytes, and indicates whether the signature is valid.
   // The address does not have to be in the wallet.
   WalletVerify(context.Context, address.Address, []byte, *crypto.Signature) (bool, error)
   // WalletDefaultAddress returns the address marked as default in the wallet.
   WalletDefaultAddress(context.Context) (address.Address, error)
   // WalletSetDefault marks the given address as as the default one.
   WalletSetDefault(context.Context, address.Address) error
   // WalletExport returns the private key of an address in the wallet.
   WalletExport(context.Context, address.Address) (*types.KeyInfo, error)
   // WalletImport receives a KeyInfo, which includes a private key, and imports it into the wallet.
   WalletImport(context.Context, *types.KeyInfo) (address.Address, error)
   // WalletDelete deletes an address from the wallet.
   WalletDelete(context.Context, address.Address) error
   // WalletValidateAddress validates whether a given string can be decoded as a well-formed address
   WalletValidateAddress(context.Context, string) (address.Address, error)

   // Other

   // MethodGroup: Client
   // The Client methods all have to do with interacting with the storage and
   // retrieval markets as a client

   // ClientImport imports file under the specified path into filestore.
   ClientImport(ctx context.Context, ref FileRef) (*ImportRes, error)
   // ClientRemoveImport removes file import
   ClientRemoveImport(ctx context.Context, importID multistore.StoreID) error
   // ClientStartDeal proposes a deal with a miner.
   ClientStartDeal(ctx context.Context, params *StartDealParams) (*cid.Cid, error)
   // ClientGetDealInfo returns the latest information about a given deal.
   ClientGetDealInfo(context.Context, cid.Cid) (*DealInfo, error)
   // ClientListDeals returns information about the deals made by the local client.
   ClientListDeals(ctx context.Context) ([]DealInfo, error)
   // ClientGetDealUpdates returns the status of updated deals
   ClientGetDealUpdates(ctx context.Context) (<-chan DealInfo, error)
   // ClientGetDealStatus returns status given a code
   ClientGetDealStatus(ctx context.Context, statusCode uint64) (string, error)
   // ClientHasLocal indicates whether a certain CID is locally stored.
   ClientHasLocal(ctx context.Context, root cid.Cid) (bool, error)
   // ClientFindData identifies peers that have a certain file, and returns QueryOffers (one per peer).
   ClientFindData(ctx context.Context, root cid.Cid, piece *cid.Cid) ([]QueryOffer, error)
   // ClientMinerQueryOffer returns a QueryOffer for the specific miner and file.
   ClientMinerQueryOffer(ctx context.Context, miner address.Address, root cid.Cid, piece *cid.Cid) (QueryOffer, error)
   // ClientRetrieve initiates the retrieval of a file, as specified in the order.
   ClientRetrieve(ctx context.Context, order RetrievalOrder, ref *FileRef) error
   // ClientRetrieveWithEvents initiates the retrieval of a file, as specified in the order, and provides a channel
   // of status updates.
   ClientRetrieveWithEvents(ctx context.Context, order RetrievalOrder, ref *FileRef) (<-chan marketevents.RetrievalEvent, error)
   // ClientQueryAsk returns a signed StorageAsk from the specified miner.
   ClientQueryAsk(ctx context.Context, p peer.ID, miner address.Address) (*storagemarket.StorageAsk, error)
   // ClientCalcCommP calculates the CommP and data size of the specified CID
   ClientDealPieceCID(ctx context.Context, root cid.Cid) (DataCIDSize, error)
   // ClientCalcCommP calculates the CommP for a specified file
   ClientCalcCommP(ctx context.Context, inpath string) (*CommPRet, error)
   // ClientGenCar generates a CAR file for the specified file.
   ClientGenCar(ctx context.Context, ref FileRef, outpath string) error
   // ClientDealSize calculates real deal data size
   ClientDealSize(ctx context.Context, root cid.Cid) (DataSize, error)
   // ClientListTransfers returns the status of all ongoing transfers of data
   ClientListDataTransfers(ctx context.Context) ([]DataTransferChannel, error)
   ClientDataTransferUpdates(ctx context.Context) (<-chan DataTransferChannel, error)
   // ClientRestartDataTransfer attempts to restart a data transfer with the given transfer ID and other peer
   ClientRestartDataTransfer(ctx context.Context, transferID datatransfer.TransferID, otherPeer peer.ID, isInitiator bool) error
   // ClientCancelDataTransfer cancels a data transfer with the given transfer ID and other peer
   ClientCancelDataTransfer(ctx context.Context, transferID datatransfer.TransferID, otherPeer peer.ID, isInitiator bool) error
   // ClientRetrieveTryRestartInsufficientFunds attempts to restart stalled retrievals on a given payment channel
   // which are stuck due to insufficient funds
   ClientRetrieveTryRestartInsufficientFunds(ctx context.Context, paymentChannel address.Address) error

   // ClientUnimport removes references to the specified file from filestore
   //ClientUnimport(path string)

   // ClientListImports lists imported files and their root CIDs
   ClientListImports(ctx context.Context) ([]Import, error)

   //ClientListAsks() []Ask

   // MethodGroup: State
   // The State methods are used to query, inspect, and interact with chain state.
   // Most methods take a TipSetKey as a parameter. The state looked up is the state at that tipset.
   // A nil TipSetKey can be provided as a param, this will cause the heaviest tipset in the chain to be used.

   // StateCall runs the given message and returns its result without any persisted changes.
   StateCall(context.Context, *types.Message, types.TipSetKey) (*InvocResult, error)
   // StateReplay replays a given message, assuming it was included in a block in the specified tipset.
   // If no tipset key is provided, the appropriate tipset is looked up.
   StateReplay(context.Context, types.TipSetKey, cid.Cid) (*InvocResult, error)
   // StateGetActor returns the indicated actor's nonce and balance.
   StateGetActor(ctx context.Context, actor address.Address, tsk types.TipSetKey) (*types.Actor, error)
   // StateReadState returns the indicated actor's state.
   StateReadState(ctx context.Context, actor address.Address, tsk types.TipSetKey) (*ActorState, error)
   // StateListMessages looks back and returns all messages with a matching to or from address, stopping at the given height.
   StateListMessages(ctx context.Context, match *MessageMatch, tsk types.TipSetKey, toht abi.ChainEpoch) ([]cid.Cid, error)
   // StateDecodeParams attempts to decode the provided params, based on the recipient actor address and method number.
   StateDecodeParams(ctx context.Context, toAddr address.Address, method abi.MethodNum, params []byte, tsk types.TipSetKey) (interface{}, error)

   // StateNetworkName returns the name of the network the node is synced to
   StateNetworkName(context.Context) (dtypes.NetworkName, error)
   // StateMinerSectors returns info about the given miner's sectors. If the filter bitfield is nil, all sectors are included.
   StateMinerSectors(context.Context, address.Address, *bitfield.BitField, types.TipSetKey) ([]*miner.SectorOnChainInfo, error)
   // StateMinerActiveSectors returns info about sectors that a given miner is actively proving.
   StateMinerActiveSectors(context.Context, address.Address, types.TipSetKey) ([]*miner.SectorOnChainInfo, error)
   // StateMinerProvingDeadline calculates the deadline at some epoch for a proving period
   // and returns the deadline-related calculations.
   StateMinerProvingDeadline(context.Context, address.Address, types.TipSetKey) (*dline.Info, error)
   // StateMinerPower returns the power of the indicated miner
   StateMinerPower(context.Context, address.Address, types.TipSetKey) (*MinerPower, error)
   // StateMinerInfo returns info about the indicated miner
   StateMinerInfo(context.Context, address.Address, types.TipSetKey) (miner.MinerInfo, error)
   // StateMinerDeadlines returns all the proving deadlines for the given miner
   StateMinerDeadlines(context.Context, address.Address, types.TipSetKey) ([]Deadline, error)
   // StateMinerPartitions returns all partitions in the specified deadline
   StateMinerPartitions(ctx context.Context, m address.Address, dlIdx uint64, tsk types.TipSetKey) ([]Partition, error)
   // StateMinerFaults returns a bitfield indicating the faulty sectors of the given miner
   StateMinerFaults(context.Context, address.Address, types.TipSetKey) (bitfield.BitField, error)
   // StateAllMinerFaults returns all non-expired Faults that occur within lookback epochs of the given tipset
   StateAllMinerFaults(ctx context.Context, lookback abi.ChainEpoch, ts types.TipSetKey) ([]*Fault, error)
   // StateMinerRecoveries returns a bitfield indicating the recovering sectors of the given miner
   StateMinerRecoveries(context.Context, address.Address, types.TipSetKey) (bitfield.BitField, error)
   // StateMinerInitialPledgeCollateral returns the precommit deposit for the specified miner's sector
   StateMinerPreCommitDepositForPower(context.Context, address.Address, miner.SectorPreCommitInfo, types.TipSetKey) (types.BigInt, error)
   // StateMinerInitialPledgeCollateral returns the initial pledge collateral for the specified miner's sector
   StateMinerInitialPledgeCollateral(context.Context, address.Address, miner.SectorPreCommitInfo, types.TipSetKey) (types.BigInt, error)
   // StateMinerAvailableBalance returns the portion of a miner's balance that can be withdrawn or spent
   StateMinerAvailableBalance(context.Context, address.Address, types.TipSetKey) (types.BigInt, error)
   // StateMinerSectorAllocated checks if a sector is allocated
   StateMinerSectorAllocated(context.Context, address.Address, abi.SectorNumber, types.TipSetKey) (bool, error)
   // StateSectorPreCommitInfo returns the PreCommit info for the specified miner's sector
   StateSectorPreCommitInfo(context.Context, address.Address, abi.SectorNumber, types.TipSetKey) (miner.SectorPreCommitOnChainInfo, error)
   // StateSectorGetInfo returns the on-chain info for the specified miner's sector. Returns null in case the sector info isn't found
   // NOTE: returned info.Expiration may not be accurate in some cases, use StateSectorExpiration to get accurate
   // expiration epoch
   StateSectorGetInfo(context.Context, address.Address, abi.SectorNumber, types.TipSetKey) (*miner.SectorOnChainInfo, error)
   // StateSectorExpiration returns epoch at which given sector will expire
   StateSectorExpiration(context.Context, address.Address, abi.SectorNumber, types.TipSetKey) (*miner.SectorExpiration, error)
   // StateSectorPartition finds deadline/partition with the specified sector
   StateSectorPartition(ctx context.Context, maddr address.Address, sectorNumber abi.SectorNumber, tok types.TipSetKey) (*miner.SectorLocation, error)
   // StateSearchMsg searches for a message in the chain, and returns its receipt and the tipset where it was executed
   StateSearchMsg(context.Context, cid.Cid) (*MsgLookup, error)
   // StateWaitMsg looks back in the chain for a message. If not found, it blocks until the
   // message arrives on chain, and gets to the indicated confidence depth.
   StateWaitMsg(ctx context.Context, cid cid.Cid, confidence uint64) (*MsgLookup, error)
   // StateWaitMsgLimited looks back up to limit epochs in the chain for a message.
   // If not found, it blocks until the message arrives on chain, and gets to the
   // indicated confidence depth.
   StateWaitMsgLimited(ctx context.Context, cid cid.Cid, confidence uint64, limit abi.ChainEpoch) (*MsgLookup, error)
   // StateListMiners returns the addresses of every miner that has claimed power in the Power Actor
   StateListMiners(context.Context, types.TipSetKey) ([]address.Address, error)
   // StateListActors returns the addresses of every actor in the state
   StateListActors(context.Context, types.TipSetKey) ([]address.Address, error)
   // StateMarketBalance looks up the Escrow and Locked balances of the given address in the Storage Market
   StateMarketBalance(context.Context, address.Address, types.TipSetKey) (MarketBalance, error)
   // StateMarketParticipants returns the Escrow and Locked balances of every participant in the Storage Market
   StateMarketParticipants(context.Context, types.TipSetKey) (map[string]MarketBalance, error)
   // StateMarketDeals returns information about every deal in the Storage Market
   StateMarketDeals(context.Context, types.TipSetKey) (map[string]MarketDeal, error)
   // StateMarketStorageDeal returns information about the indicated deal
   StateMarketStorageDeal(context.Context, abi.DealID, types.TipSetKey) (*MarketDeal, error)
   // StateLookupID retrieves the ID address of the given address
   StateLookupID(context.Context, address.Address, types.TipSetKey) (address.Address, error)
   // StateAccountKey returns the public key address of the given ID address
   StateAccountKey(context.Context, address.Address, types.TipSetKey) (address.Address, error)
   // StateChangedActors returns all the actors whose states change between the two given state CIDs
   // TODO: Should this take tipset keys instead?
   StateChangedActors(context.Context, cid.Cid, cid.Cid) (map[string]types.Actor, error)
   // StateGetReceipt returns the message receipt for the given message
   StateGetReceipt(context.Context, cid.Cid, types.TipSetKey) (*types.MessageReceipt, error)
   // StateMinerSectorCount returns the number of sectors in a miner's sector set and proving set
   StateMinerSectorCount(context.Context, address.Address, types.TipSetKey) (MinerSectors, error)
   // StateCompute is a flexible command that applies the given messages on the given tipset.
   // The messages are run as though the VM were at the provided height.
   StateCompute(context.Context, abi.ChainEpoch, []*types.Message, types.TipSetKey) (*ComputeStateOutput, error)
   // StateVerifierStatus returns the data cap for the given address.
   // Returns nil if there is no entry in the data cap table for the
   // address.
   StateVerifierStatus(ctx context.Context, addr address.Address, tsk types.TipSetKey) (*abi.StoragePower, error)
   // StateVerifiedClientStatus returns the data cap for the given address.
   // Returns nil if there is no entry in the data cap table for the
   // address.
   StateVerifiedClientStatus(ctx context.Context, addr address.Address, tsk types.TipSetKey) (*abi.StoragePower, error)
   // StateVerifiedClientStatus returns the address of the Verified Registry's root key
   StateVerifiedRegistryRootKey(ctx context.Context, tsk types.TipSetKey) (address.Address, error)
   // StateDealProviderCollateralBounds returns the min and max collateral a storage provider
   // can issue. It takes the deal size and verified status as parameters.
   StateDealProviderCollateralBounds(context.Context, abi.PaddedPieceSize, bool, types.TipSetKey) (DealCollateralBounds, error)

   // StateCirculatingSupply returns the exact circulating supply of Filecoin at the given tipset.
   // This is not used anywhere in the protocol itself, and is only for external consumption.
   StateCirculatingSupply(context.Context, types.TipSetKey) (abi.TokenAmount, error)
   // StateVMCirculatingSupplyInternal returns an approximation of the circulating supply of Filecoin at the given tipset.
   // This is the value reported by the runtime interface to actors code.
   StateVMCirculatingSupplyInternal(context.Context, types.TipSetKey) (CirculatingSupply, error)
   // StateNetworkVersion returns the network version at the given tipset
   StateNetworkVersion(context.Context, types.TipSetKey) (network.Version, error)

   // MethodGroup: Msig
   // The Msig methods are used to interact with multisig wallets on the
   // filecoin network

   // MsigGetAvailableBalance returns the portion of a multisig's balance that can be withdrawn or spent
   MsigGetAvailableBalance(context.Context, address.Address, types.TipSetKey) (types.BigInt, error)
   // MsigGetVestingSchedule returns the vesting details of a given multisig.
   MsigGetVestingSchedule(context.Context, address.Address, types.TipSetKey) (MsigVesting, error)
   // MsigGetVested returns the amount of FIL that vested in a multisig in a certain period.
   // It takes the following params: <multisig address>, <start epoch>, <end epoch>
   MsigGetVested(context.Context, address.Address, types.TipSetKey, types.TipSetKey) (types.BigInt, error)
   // MsigCreate creates a multisig wallet
   // It takes the following params: <required number of senders>, <approving addresses>, <unlock duration>
   //<initial balance>, <sender address of the create msg>, <gas price>
   MsigCreate(context.Context, uint64, []address.Address, abi.ChainEpoch, types.BigInt, address.Address, types.BigInt) (cid.Cid, error)
   // MsigPropose proposes a multisig message
   // It takes the following params: <multisig address>, <recipient address>, <value to transfer>,
   // <sender address of the propose msg>, <method to call in the proposed message>, <params to include in the proposed message>
   MsigPropose(context.Context, address.Address, address.Address, types.BigInt, address.Address, uint64, []byte) (cid.Cid, error)

   // MsigApprove approves a previously-proposed multisig message by transaction ID
   // It takes the following params: <multisig address>, <proposed transaction ID> <signer address>
   MsigApprove(context.Context, address.Address, uint64, address.Address) (cid.Cid, error)

   // MsigApproveTxnHash approves a previously-proposed multisig message, specified
   // using both transaction ID and a hash of the parameters used in the
   // proposal. This method of approval can be used to ensure you only approve
   // exactly the transaction you think you are.
   // It takes the following params: <multisig address>, <proposed message ID>, <proposer address>, <recipient address>, <value to transfer>,
   // <sender address of the approve msg>, <method to call in the proposed message>, <params to include in the proposed message>
   MsigApproveTxnHash(context.Context, address.Address, uint64, address.Address, address.Address, types.BigInt, address.Address, uint64, []byte) (cid.Cid, error)

   // MsigCancel cancels a previously-proposed multisig message
   // It takes the following params: <multisig address>, <proposed transaction ID>, <recipient address>, <value to transfer>,
   // <sender address of the cancel msg>, <method to call in the proposed message>, <params to include in the proposed message>
   MsigCancel(context.Context, address.Address, uint64, address.Address, types.BigInt, address.Address, uint64, []byte) (cid.Cid, error)
   // MsigAddPropose proposes adding a signer in the multisig
   // It takes the following params: <multisig address>, <sender address of the propose msg>,
   // <new signer>, <whether the number of required signers should be increased>
   MsigAddPropose(context.Context, address.Address, address.Address, address.Address, bool) (cid.Cid, error)
   // MsigAddApprove approves a previously proposed AddSigner message
   // It takes the following params: <multisig address>, <sender address of the approve msg>, <proposed message ID>,
   // <proposer address>, <new signer>, <whether the number of required signers should be increased>
   MsigAddApprove(context.Context, address.Address, address.Address, uint64, address.Address, address.Address, bool) (cid.Cid, error)
   // MsigAddCancel cancels a previously proposed AddSigner message
   // It takes the following params: <multisig address>, <sender address of the cancel msg>, <proposed message ID>,
   // <new signer>, <whether the number of required signers should be increased>
   MsigAddCancel(context.Context, address.Address, address.Address, uint64, address.Address, bool) (cid.Cid, error)
   // MsigSwapPropose proposes swapping 2 signers in the multisig
   // It takes the following params: <multisig address>, <sender address of the propose msg>,
   // <old signer>, <new signer>
   MsigSwapPropose(context.Context, address.Address, address.Address, address.Address, address.Address) (cid.Cid, error)
   // MsigSwapApprove approves a previously proposed SwapSigner
   // It takes the following params: <multisig address>, <sender address of the approve msg>, <proposed message ID>,
   // <proposer address>, <old signer>, <new signer>
   MsigSwapApprove(context.Context, address.Address, address.Address, uint64, address.Address, address.Address, address.Address) (cid.Cid, error)
   // MsigSwapCancel cancels a previously proposed SwapSigner message
   // It takes the following params: <multisig address>, <sender address of the cancel msg>, <proposed message ID>,
   // <old signer>, <new signer>
   MsigSwapCancel(context.Context, address.Address, address.Address, uint64, address.Address, address.Address) (cid.Cid, error)

   // MsigRemoveSigner proposes the removal of a signer from the multisig.
   // It accepts the multisig to make the change on, the proposer address to
   // send the message from, the address to be removed, and a boolean
   // indicating whether or not the signing threshold should be lowered by one
   // along with the address removal.
   MsigRemoveSigner(ctx context.Context, msig address.Address, proposer address.Address, toRemove address.Address, decrease bool) (cid.Cid, error)

   // MarketReserveFunds reserves funds for a deal
   MarketReserveFunds(ctx context.Context, wallet address.Address, addr address.Address, amt types.BigInt) (cid.Cid, error)
   // MarketReleaseFunds releases funds reserved by MarketReserveFunds
   MarketReleaseFunds(ctx context.Context, addr address.Address, amt types.BigInt) error
   // MarketWithdraw withdraws unlocked funds from the market actor
   MarketWithdraw(ctx context.Context, wallet, addr address.Address, amt types.BigInt) (cid.Cid, error)

   // MethodGroup: Paych
   // The Paych methods are for interacting with and managing payment channels

   PaychGet(ctx context.Context, from, to address.Address, amt types.BigInt) (*ChannelInfo, error)
   PaychGetWaitReady(context.Context, cid.Cid) (address.Address, error)
   PaychAvailableFunds(ctx context.Context, ch address.Address) (*ChannelAvailableFunds, error)
   PaychAvailableFundsByFromTo(ctx context.Context, from, to address.Address) (*ChannelAvailableFunds, error)
   PaychList(context.Context) ([]address.Address, error)
   PaychStatus(context.Context, address.Address) (*PaychStatus, error)
   PaychSettle(context.Context, address.Address) (cid.Cid, error)
   PaychCollect(context.Context, address.Address) (cid.Cid, error)
   PaychAllocateLane(ctx context.Context, ch address.Address) (uint64, error)
   PaychNewPayment(ctx context.Context, from, to address.Address, vouchers []VoucherSpec) (*PaymentInfo, error)
   PaychVoucherCheckValid(context.Context, address.Address, *paych.SignedVoucher) error
   PaychVoucherCheckSpendable(context.Context, address.Address, *paych.SignedVoucher, []byte, []byte) (bool, error)
   PaychVoucherCreate(context.Context, address.Address, types.BigInt, uint64) (*VoucherCreateResult, error)
   PaychVoucherAdd(context.Context, address.Address, *paych.SignedVoucher, []byte, types.BigInt) (types.BigInt, error)
   PaychVoucherList(context.Context, address.Address) ([]*paych.SignedVoucher, error)
   PaychVoucherSubmit(context.Context, address.Address, *paych.SignedVoucher, []byte, []byte) (cid.Cid, error)

   // CreateBackup creates node backup onder the specified file name. The
   // method requires that the lotus daemon is running with the
   // LOTUS_BACKUP_BASE_PATH environment variable set to some path, and that
   // the path specified when calling CreateBackup is within the base path
   CreateBackup(ctx context.Context, fpath string) error
}
```







### client.go-client包（文件夹）



```go
func NewCommonRPC(ctx context.Context, addr string, requestHeader http.Header) (api.Common, jsonrpc.ClientCloser, error)

func NewFullNodeRPC(ctx context.Context, addr string, requestHeader http.Header) (api.FullNode, jsonrpc.ClientCloser, error)

func NewStorageMinerRPC(ctx context.Context, addr string, requestHeader http.Header, opts ...jsonrpc.Option) (api.StorageMiner, jsonrpc.ClientCloser, error)

func NewWorkerRPC(ctx context.Context, addr string, requestHeader http.Header) (api.WorkerAPI, jsonrpc.ClientCloser, error)

func NewGatewayRPC(ctx context.Context, addr string, requestHeader http.Header, opts ...jsonrpc.Option) (api.GatewayAPI, jsonrpc.ClientCloser, error)

func NewWalletRPC(ctx context.Context, addr string, requestHeader http.Header) (api.WalletAPI, jsonrpc.ClientCloser, error)


```



### api_wallet.go

```go
type WalletAPI interface {
   WalletNew(context.Context, types.KeyType) (address.Address, error)
   WalletHas(context.Context, address.Address) (bool, error)
   WalletList(context.Context) ([]address.Address, error)

   WalletSign(ctx context.Context, signer address.Address, toSign []byte, meta MsgMeta) (*crypto.Signature, error)

   WalletExport(context.Context, address.Address) (*types.KeyInfo, error)
   WalletImport(context.Context, *types.KeyInfo) (address.Address, error)
   WalletDelete(context.Context, address.Address) error
}
```











## cli





CommonCommands
Commands
ErrCmdFailed
Error
ApiConnector
GetStorageMinerOptions
PreferHttp
GetStorageMinerOption
NewCliError
GetAPIInfo
GetRawAPI
GetAPI
GetFullNodeAPI
StorageMinerUseHttp
GetStorageMinerAPI
GetWorkerAPI
GetGatewayAPI
DaemonContext
ReqContext
WithCategory

















## cmd文件夹






















































































































