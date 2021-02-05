configtxgen
configtxgen命令用来创建或查看通道配置相关的构件。生成的构件内容取决于 configtx.yaml文件。

命令语法
configtxgen工具没有子命令，使用标志来完成不同的任务：

~$ configtxgen [flag]
可用标志如下：

  -asOrg string
        Performs the config generation as a particular organization (by name), only including values in the write set that org (likely) has privilege to set
  -channelID string
        The channel ID to use in the configtx
  -configPath string
        The path containing the configuration to use (if set)
  -inspectBlock string
        Prints the configuration contained in the block at the specified path
  -inspectChannelCreateTx string
        Prints the configuration contained in the transaction at the specified path
  -outputAnchorPeersUpdate string
        Creates an config update to update an anchor peer (works only with the default channel creation, and only for the first update)
  -outputBlock string
        The path to write the genesis block to (if set)
  -outputCreateChannelTx string
        The path to write a channel creation configtx to (if set)
  -printOrg string
        Prints the definition of an organization as JSON. (useful for adding an org to a channel manually)
  -profile string
        The profile from configtx.yaml to use for generation. (default "SampleInsecureSolo")
  -version
        Show version information