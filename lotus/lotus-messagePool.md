# 目录

[toc] 



# message Pool

消息池（消息池）是Lotus的组件，它处理待处理的消息以包含在链中。消息直接添加到本地发布的消息或通过pubsub传播添加到mpool。每当矿工准备为技巧集创建块时，它都会调用mpool选择算法，该算法选择一组适当的消息，从而优化矿工的奖励和链条容量。

[目录](#目录) 





## API

全节点API定义了以下与mpool进行交互的方法：

```go
MpoolPending(context.Context, types.TipSetKey) ([]*types.SignedMessage, error)
//Returns the list of messages that are pending for a tipset 返回等待提示集的消息列表

MpoolSelect(context.Context, types.TipSetKey, float64) ([]*types.SignedMessage, error)
//Selects and returns a list of pending messages for inclusion in the next block包含在下一块中

MpoolPush(context.Context, *types.SignedMessage) (cid.Cid, error)
//Pushes a signed message to the mpool; returns the CID of the message
//将已签名的消息推送到mpool；返回消息的CID

MpoolPushMessage(ctx context.Context, msg *types.Message, spec *MessageSendSpec) (*types.SignedMessage, error)
//以原子方式分配随机数，符号，然后将消息推送到mpool。仅当在消息中未指定GasFeeCap / GasPremium字段时，
//才使用spec参数的MaxFee字段。当maxFee设置为0时，MpoolPushMessage将根据当前的链条条件猜测适当的费用。

MpoolGetNonce(context.Context, address.Address) (uint64, error)
//返回指定发件人的下一个随机数。请注意，此方法可能不是原子方法。改用MpoolPushMessage

MpoolSub(context.Context) (<-chan MpoolUpdate, error)
//返回一个通道，以接收有关消息池更新的通知。请注意，调用者完成订阅后，必须取消上下文

MpoolGetConfig(context.Context) (*types.MpoolConfig, error)
//Returns (a copy of) the current mpool configuration

MpoolSetConfig(context.Context, *types.MpoolConfig) error
//Sets the mpool configuration to (a copy of) the supplied configuration object

MpoolClear(context.Context, local bool) error
//清除来自mpool的待处理消息；如果local为true，则还将清除本地消息并将其从数据存储中删除。
//应格外小心，仅在更换磁头时发生错误而使mpool处于不一致状态时才应使用此命令。
```

## 命令行接口



```go
lotus mpool pending [--local]
//MpoolPending API call打印消息池中待处理消息,如果指定了--local，则仅显示本地钱包中地址的待处理消息。

lotus mpool sub
//MpoolSub,使用MpoolSub API调用订阅mpool更改并打印mpool更新流

lotus mpool stat [--local]
//打印各种Mpool统计信息

lotus mpool replace [--gas-feecap <feecap>] [--gas-premium <premium>] [--gas-limit <limit>] [from] [nonce]
//替换mpool中的消息

lotus mpool find [--from <address>] [--to <address>] [--method <int>]
//查找消息

lotus mpool config [<configuration>]
//Gets or sets 配置

lotus mpool clear [--local]
//无条件清除来自mpool的挂起消息。警告：仅当换头错误使mpool处于状态时，才应使用此命令
```

[目录](#目录) 



## 配置

只有几个参数能够配置

```go
type MpoolConfig struct {
	PriorityAddrs          []address.Address
    // actors的地址，（消息是有利可图的）pending消息应始终在消息选择期间包含在区块中，miner应配置自己worker地址，以便在生成新块时包括自己的消息。默认为空
    
	SizeLimitHigh          int
    //在消息池中触发丢弃消息之前待处理消息的最大数量，请注意，来自优先级地址的消息永远不会被丢弃。默认为30000

	SizeLimitLow           int
    //消息池丢弃后最小的待处理消息数量，默认20000
    
	ReplaceByFeeRatio      float64
    //这是替换mpool中消息的汽油费比率。每当替换消息时，必须通过这个参数将GasPremium增大，默认1.25
    
	PruneCooldown          time.Duration
    //出发裁剪消息之前需要等待的时间，默认1min
    
	GasLimitOverestimation float64
    //控制gaslimit过高，默认1.25
}
```



## 消息选择

在消息选择中，矿工从待处理消息中选择一组消息以包含到下一个区块中，以最大化其奖励。然而，问题在于NP难（这是背包包装的一个实例），而矿工之间并不相互交流门票，这一事实进一步使他们感到困惑。因此，充其量我们可以在合理的时间内估算出最佳选择。



给定矿工的票证质量，mpool使用复杂的算法选择要包含的消息。票证质量反映了tipset中某个区块的执行顺序的概率。给定票证质量，该算法将计算每个区块的概率，并选择相关的消息链，使reward最大化，同时还优化链的容量。如果票证质量足够高，则使用贪婪选择算法代替，该算法只是选择最大奖励的相关链。

请注意，始终选择来自优先级地址的挂起消息链，无论其盈利能力如何。



消息选择的算法可以阅读源码chain/messagepool/selection.go。





## 其他

cli/mpool.go

命令行使用















