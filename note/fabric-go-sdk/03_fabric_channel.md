### channel 

# client/channel包
# channel.Client - 通道客户端结构定义
# channel.New() - 创建通道客户端
# cc.Execute() - 执行交易
# cc.InvokeHandler() - 调用指定的处理器
# cc.Query() - 查询链码
# cc.RegisterChaincodeEvent() - 监听链码事件
# cc.UnregisterChaincodeEvent() - 取消监听链码事件
# channel.ClientOption - 客户端选项结构定义
# channel.Request - 链码请求结构定义
# channle.RequestOption - 链码请求选项函数
# channel.WithBeforeRetry() - 设置链码请求重试前需调用的函数
# channel.WithChaincodeFilter() - 为链码请求添加链码过滤器
# channel.WithParentContext() - 为链码请求封装父级上下文
# channel.WithRetry() - 为链码请求配置重试参数
# channel.WithTargetEndpoints() - 为链码请求配置访问端结点
# channel.WithTargetFilter() - 为特定链码请求指定节点过滤器
# channel.WithTargetSorter() - 对特定链码请求指定排序器
# channel.WithTargets() - 为链码请求设置目标peer节点
# channel.WithTimeout() - 为链码请求设置超时参数
# channel.Response - 链码响应结构定义