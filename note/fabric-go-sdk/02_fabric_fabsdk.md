### fabsdk包相关操作


# FabricSDK - sdk入口
# fabsdk.New() - 创建FabricSDK实例
```
sdk, err := fabsdk.New(config.FromFile(sdkConfig))
```
# sdk.ChannelContext() - 创建通道上下文实例
```
rcp := sdk.Context(fabsdk.WithOrg(orgName), fabsdk.WithUser(userName))
```

# sdk.Close() - 关闭FabricSDK实例
# sdk.CloseContext() - 关闭指定的上下文实例
# sdk.Config() - 创建配置后端实例
# sdk.Context() - 创建SDK上下文实例
# fabsdk.ContextOption - SDK上下文配置结构定义
# fabsdk.WithIdentity() - 创建身份上下文配置对象
# fabsdk.WithOrg() - 创建机构上下文配置对象
# fabsdk.WithUser() - 创建用户上下文配置对象
# fabsdk.Option - SDK配置结构定义
# fabsdk.WithCorePkg() - 向SDK注入核心包
# fabsdk.WithCryptoSuiteConfig() - 向SDK注入密码学套件接口
# fabsdk.WithEndpointConfig() - 向SDK注入端结点配置接口
# fabsdk.WithErrorHandler() - 设置错误处理程序
# fabsdk.WithIdentityConfig() - 向SDK注入身份配置接口
# fabsdk.WithLoggerPkg() - 向SDK注入日志实现
# fabsdk.WithMSPPkg() - 向SDK注入MSP实现
# fabsdk.WithMetricsConfig() - 向SDK注入监视指标配置接口
# fabsdk.WithProviderOpts() - 向提供器添加额外的选项
# fabsdk.WithServicePkg() - 向SDK注入服务实现