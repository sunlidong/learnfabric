### ledger包 


# client/ledger包
# ledger.Client - 账本客户端结构定义
# ledger.New() - 创建账本客户端实例
# lc.QueryBlock() - 按编号查询区块
# lc.QueryBlockByHash() - 按哈希查询区块
# lc.QueryBlockByTxID() - 查询包含指定交易的区块
# lc.QueryConfig() - 查询通道配置
# lc.QueryConfigBlock() - 查询指定通道的当前配置区块
# lc.QueryInfo() - 查询指定通道的相关信息
# lc.QueryTransaction() - 查询指定的交易
# ClientOption - 账本客户端选项结构定义
# ledger.WithDefaultTargetFilter - 使用默认的节点过滤器
# RequestOption - 请求选项函数
# ledger.WithMaxTargets - 声明每个请求最多可以选择的节点
# ledger.WithMinTargets - 声明每个请求最少需要的响应
# ledger.WithParentContext - 使用父级上下文
# ledger.WithTargetEndpoints - 使用指定的访问端节点
# ledger.WithTargetFilter - 声明节点选择过滤器
# ledger.WithTargets - 为特定请求指定目标节点
# ledger.WithTimeout - 指定账本客户端的超时参数