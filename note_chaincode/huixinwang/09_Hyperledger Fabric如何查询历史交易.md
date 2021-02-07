从以太坊、比特币等其他区块链进入Hyperledger Fabric的新手，常常会问：如何 查询Hyperledger Fabric区块链上的历史交易？的确，既然区块链或账本 上都有交易记录了，怎么找不到一个简单的API来查询历史交易？

在以太坊、比特币等区块链平台中，通常都会提供简单的JSON RPC API接口， 应用程序只需要调用这些RPC API，就可以查询区块或历史交易了。Hyperledger Fabric 也有类似的API，但情况略有不同，根据查询目的区别，可以分为两种方法。

相关教程：

Hyperledger Fabric Java链码与应用开发详解
Hyperledger Fabric Node.js链码与应用开发详解
1、使用系统链码qscc
如果你在寻找像以太坊/比特币那样的区块查询、交易查询API，那就应该 使用系统链码QSCC，该链码提供了如下方法：

GetChainInfo：获取链信息
GetBlockByNumer：按区块号获取区块数据
GetBlockByHash：按区块哈希获取区块数据
GetTransactionById：按交易ID获取交易数据
GetBlockByTxId：按交易ID获取区块数据
调用系统链码和调用自己的链码没什么区别，例如下面是调用 qscc链码的GetChainInfo()方法的go语言测试代码：

response, err := chClient.Query(
  chclient.Request{
    ChaincodeID: "qscc", 
    Fcn: "invoke", 
    Args: integration.ExampleCCQueryArgs("GetChainInfo")
  })
原始代码可参考：go sdk test

2、查询指定键的历史交易
如果要查询特定链码中指定状态键的历史交易，可以在链码中使用 ChaincodeStubInterface接口的GetHistoryForKey()方法来查询 其历史记录。例如：

historyIter, err := stub.GetHistoryForKey(yourKey)

if err != nil {
    fmt.Println(errMsg)
    return shim.Error(errMsg)
}

if historyIter.HasNext() {
    modification, err := historyIter.Next()
    if err != nil {
        fmt.Println(errMsg)
        return shim.Error(errMsg)
    }
    fmt.Println("Returning information related to", string(modification.Value))
}
上面的链码要正常工作，需要在core.yaml中设置enableHistoryDatabase 配置为true:

ledger:
  history:
    enableHistoryDatabase: true
