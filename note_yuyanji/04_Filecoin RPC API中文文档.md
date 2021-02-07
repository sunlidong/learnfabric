JSON-RPC是一种标准的远程调用方式，在以太坊和比特币等区块链实现中都得到了广泛的应用。 Filecoin Lotus也是使用JSON-RPC实现服务进程与客户端的通信，默认监听端口为1234， 支持HTTP和Websocket。Filecoin RPC API中文文档由汇智网提供，访问地址：http://cw.hubwiz.com/card/c/filecoin-lotus-rpc/。

各种常见开发语言或命令行工具都可以访问Filecoin的JSON RPC API。例如使用CURL查询版本信息：

curl -X POST \
  -H "Content-Type: application/json" \
  --data '{ 
      "jsonrpc": "2.0", 
      "method": "Filecoin.Version", 
      "params": [], 
      "id": 1 
    }' \
  http://127.0.0.1:1234/rpc/v0
响应结果看起来会像这样：

{
  "jsonrpc":"2.0",
  "result":{
    "Version":"0.3.0+gite4b5f1df.dirty",
    "APIVersion":512,
    "BlockDelay":6
  },
  "id":1
}
根据用途不同，Filecoin Lutos RPC API可分为如下12个不同的组：

1、Auth/授权令牌相关API
Filecoin.AuthNew - 生成新的授权令牌
Filecoin.AuthVerify - 列举全部授权令牌
2、Chain/链相关API
Filecoin.ChainDeleteObj - 删除链上对象
Filecoin.ChainExport - 导出CAR文件
Filecoin.ChainGetBlock - 读取区块
Filecoin.ChainGetBlockMessages - 读取区块内消息
Filecoin.ChainGetGenesis - 读取创世TipSet
Filecoin.ChainGetMessage - 读取指定消息
Filecoin.ChainGetParentMessages - 读取父TipSet中的消息
Filecoin.ChainGetParentReceipts - 读取父TipSet中的收据
Filecoin.ChainGetPath - 返回TipSet变换路径
Filecoin.ChainGetTipSetByHeight - 读取指定高度的TipSet
Filecoin.ChainHasObj - 检查链库中是否存在指定对象
Filecoin.ChainHead - 返回当前TipSet
Filecoin.ChainNotify - 注册TipSet更新回调
Filecoin.ChainReadObj - 读取指定对象
Filecoin.ChainSetHead - 设置当前TipSet
Filecoin.ChainStatObj - 返回指定对象的统计信息
Filecoin.ChainTipSetWeight - 计算指定TipSet的权重
3、Client/节点相关API
Filecoin.ClientCalcCommP - 计算指定文件的作品承诺
Filecoin.ClientCancelDataTransfer - 取消指定的数据传输
Filecoin.ClientDataTransferUpdates - 注册数据传输回调
Filecoin.ClientDealPieceCID - 返回指定交易作品大小
Filecoin.ClientDealSize - 返回指定交易大小
Filecoin.ClientFindData - 查找指定文件
Filecoin.ClientGenCar - 生成指定文件的CAR转储
Filecoin.ClientGetDealInfo - 返回指定交易的信息
Filecoin.ClientGetDealStatus - 返回指定代码对应的状态
Filecoin.ClientGetDealUpdates - 注册交易状态更新回调
Filecoin.ClientHasLocal - 检查本地是否存有指定数据
Filecoin.ClientImport - 导入本地文件
Filecoin.ClientListDataTransfers - 列举数据传输状态
Filecoin.ClientListDeals - 列举交易信息
Filecoin.ClientListImports - 列举导入的文件
Filecoin.ClientMinerQueryOffer - 查询矿工Offer
Filecoin.ClientQueryAsk - 查询StorageAsk
Filecoin.ClientRemoveImport - 删除指定的导入文件
Filecoin.ClientRestartDataTransfer - 重启指定的数据传输
Filecoin.ClientRetrieve - 提取文件
Filecoin.ClientRetrieveTryRestartInsufficientFunds - 重启资金不足通道并提取文件
Filecoin.ClientRetrieveWithEvents - 提取文件并注册进度回调
Filecoin.ClientStartDeal - 启动交易
4、公共API
Filecoin.ID - 返回节点ID
Filecoin.Version - 返回版本信息
Filecoin.LogList - 返回节点日志清单
Filecoin.LogSetLevel - 设置日志等级
Filecoin.Shutdown - 优雅关闭节点
5、Gas/手续费相关API
Filecoin.GasEstimateFeeCap - 估算手续费封顶值
Filecoin.GasEstimateGasLimit - 估算Gas上限
Filecoin.GasEstimateGasPremium - 估算Gas用量
Filecoin.GasEstimateMessageGas - 估算消息Gas用量
6、Miner/挖矿相关API
Filecoin.MinerCreateBlock - 创建区块
Filecoin.MinerGetBaseInfo - 返回矿工基本信息
7、Mpool/内存池相关API
Filecoin.MpoolBatchPush - 签名消息批量推入内存池
Filecoin.MpoolBatchPushMessage - 未签名消息推入内存池
Filecoin.MpoolBatchPushUntrusted - 非可信源签名消息批量入池
Filecoin.MpoolClear - 清空内存池
Filecoin.MpoolGetConfig - 返回内存池当前配置
Filecoin.MpoolGetNonce - 返回指定地址的下一随机值
Filecoin.MpoolPending - 返回内存池中的待定消息
Filecoin.MpoolPush - 将指定签名消息推入内存池
Filecoin.MpoolPushMessage - 将指定未签名消息推入内存池
Filecoin.MpoolPushUntrusted - 将指定非可信源签名消息推入内存池
Filecoin.MpoolSelect - 选择一组待定交易用于打包区块
Filecoin.MpoolSetConfig - 设置内存池的当前配置
Filecoin.MpoolSub - 注册内存池状态更新回调
8、Msig/多重签名相关API
Filecoin.MsigAddApprove - 批准AddSigner提议
Filecoin.MsigAddCancel - 取消AddSigner提议
Filecoin.MsigAddPropose - 创建AddSigner提议
Filecoin.MsigApprove - 批准多签消息
Filecoin.MsigApproveTxnHash - 批准多签消息
Filecoin.MsigCancel - 取消多签消息
Filecoin.MsigCreate - 创建多签钱包
Filecoin.MsigGetAvailableBalance - 返回多签钱包的有效余额
Filecoin.MsigGetVested - 返回多签钱包的授权数量
Filecoin.MsigGetVestingSchedule - 返回多签钱包的授权详情
Filecoin.MsigPropose - 提议多签消息
Filecoin.MsigRemoveSigner - 从多签钱包移除一个签名方
Filecoin.MsigSwapApprove - 批准SwapSigner消息
Filecoin.MsigSwapCancel - 取消SwapSigner消息
Filecoin.MsigSwapPropose - 创建SwapSigner提议
9、Net/网络相关API
Filecoin.NetAddrsListen - 返回节点地址
Filecoin.NetAutoNatStatus - 返回节点NAT统计
Filecoin.NetBlockAdd
Filecoin.NetConnect - 连接指定节点
Filecoin.NetConnectedness - 返回节点连接状态
Filecoin.NetDisconnect - 断开与指定节点的连接
Filecoin.NetFindPeer - 查找指定节点的地址
Filecoin.NetPeers - 返回已连接节点清单
Filecoin.NetPubsubScores
10、Paych/支付通道相关API
Filecoin.PaychAllocateLane
Filecoin.PaychAvailableFunds
Filecoin.PaychAvailableFundsByFromTo
Filecoin.PaychCollect
Filecoin.PaychGet
Filecoin.PaychGetWaitReady
Filecoin.PaychList
Filecoin.PaychNewPayment
Filecoin.PaychSettle
Filecoin.PaychStatus
Filecoin.PaychVoucherAdd
Filecoin.PaychVoucherCheckSpendable
Filecoin.PaychVoucherCheckValid
Filecoin.PaychVoucherCreate
Filecoin.PaychVoucherList
Filecoin.PaychVoucherSubmit
11、State/状态相关API
Filecoin.StateAccountKey - 查询账号公钥
Filecoin.StateAllMinerFaults - 查询矿工故障
Filecoin.StateCall - 只读执行消息
Filecoin.StateChangedActors - 查询发生变化的ACTOR
Filecoin.StateCirculatingSupply - 查询指定周期的流通供应量
Filecoin.StateCompute - 执行指定的消息
Filecoin.StateDealProviderCollateralBounds - 返回抵押范围
Filecoin.StateGetActor - 查询指定的Actor
Filecoin.StateGetReceipt - 查询指定的收据
Filecoin.StateListActors - 列表返回全部Actor
Filecoin.StateListMessages - 列表返回消息
Filecoin.StateListMiners - 列表返回矿工
Filecoin.StateLookupID - 查找指定的地址
Filecoin.StateMarketBalance - 查询指定地址的存储市场余额
Filecoin.StateMarketDeals - 列表返回市场交易
Filecoin.StateMarketParticipants - 列表返回市场参与各方
Filecoin.StateMarketStorageDeal - 查询指定的存储交易
Filecoin.StateMinerActiveSectors - 查询指定矿工的活动扇区
Filecoin.StateMinerAvailableBalance - 查询指定矿工的有效余额
Filecoin.StateMinerDeadlines - 查询指定矿工的截止时间
Filecoin.StateMinerFaults - 查询指定矿工的故障信息
Filecoin.StateMinerInfo - 查询指定矿工的基本信息
Filecoin.StateMinerInitialPledgeCollateral - 查询指定矿工的初始抵押
Filecoin.StateMinerPartitions - 查询指定矿工的分区信息
Filecoin.StateMinerPower - 查询指定矿工的算力
Filecoin.StateMinerPreCommitDepositForPower - 查询算力预提交存款
Filecoin.StateMinerProvingDeadline - 查询证明截止时间
Filecoin.StateMinerRecoveries - 查询矿工的故障恢复状况
Filecoin.StateMinerSectorAllocated - 检查指定扇区是否分配
Filecoin.StateMinerSectorCount - 查询矿工扇区数量
Filecoin.StateMinerSectors - 查询矿工扇区信息
Filecoin.StateNetworkName - 查询当前网络名称
Filecoin.StateNetworkVersion - 查询当前网络版本
Filecoin.StateReadState - 查询读取状态
Filecoin.StateReplay - 重放指定的消息
Filecoin.StateSearchMsg - 搜索链上消息
Filecoin.StateSectorExpiration - 查询指定扇区的超时
Filecoin.StateSectorGetInfo - 查询指定扇区
Filecoin.StateSectorPartition - 查询扇区分区信息
Filecoin.StateSectorPreCommitInfo - 查询扇区预提交信息
Filecoin.StateVMCirculatingSupplyInternal - 估算流通供应量
Filecoin.StateVerifiedClientStatus - 返回已验证状态
Filecoin.StateVerifiedRegistryRootKey - 返回已验证注册器根键
Filecoin.StateVerifierStatus - 返回验证器状态
Filecoin.StateWaitMsg - 查找并等待指定消息上链
Filecoin.StateWaitMsgLimited - 有效范围查找并等待消息上链
12、Sync/同步相关API
Filecoin.SyncCheckBad - 检查坏块
Filecoin.SyncCheckpoint - 标记检查点
Filecoin.SyncIncomingBlocks - 注册区块头接收回调
Filecoin.SyncMarkBad - 标记坏块
Filecoin.SyncState - 返回同步状态
Filecoin.SyncSubmitBlock - 提交区块
Filecoin.SyncUnmarkAllBad - 取消全部坏块标记
Filecoin.SyncUnmarkBad - 取消指定坏块标记
Filecoin.SyncValidateTipset - 校验指定的TipSet
13、Wallet/钱包相关API
Filecoin.WalletBalance - 查询钱包余额
Filecoin.WalletDefaultAddress - 查询钱包默认地址
Filecoin.WalletDelete - 删除钱包中的指定地址
Filecoin.WalletExport - 导出钱包中指定地址的私钥
Filecoin.WalletHas - 检查钱包是否包含指定地址
Filecoin.WalletImport - 将指定私钥导入钱包
Filecoin.WalletList - 列表返回钱包中的全部地址
Filecoin.WalletNew - 在钱包中创建一个新地址
Filecoin.WalletSetDefault - 设置钱包的默认地址
Filecoin.WalletSign - 使用指定地址签名数据
Filecoin.WalletSignMessage - 使用指定地址签名消息
Filecoin.WalletValidateAddress - 验证指定的地址字符串
Filecoin.WalletVerify - 验证签名有效性