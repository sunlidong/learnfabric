在DEFI数据分析应用中通常需要获取大量区块链数据进行显示或分析，而这往往会多次调用合约， 从而导致总查询时间过长。并且，如果我们使用的是Infura这样的第三方提供程序，短时间内的多次 请求还有可能触发服务商的限流管控。MakeDAO的Multicall就是解决这一问题的最佳方案，它包含 链上合约与链下NPM包两个部分，可以将多个针对以太坊区块链查询请求合并为一个，从而有效缩短响应时间并降低eth_call调用的消耗。

为了帮助你了解这种机制的工作原理以及其相对于传统方法是否确实有所改进（每个函数调用n次）， 我们将举一个示例，分别在不使用/使用Multicall的情况下进行操作，然后分析结果。为此，我们 将通过调用Compound协议的getAccountLiquidity()函数来查询1,000个不同的地址的流动性数据。

用自己熟悉的语言学习 以太坊DApp开发 ： Java | Php | Python | .Net / C# | Golang | Node.JS | Flutter / Dart

1、multicall对比测试：安装NPM依赖包
为了进行此测试，我们将创建一个Node项目，并将安装如下依赖：

ethers.js：和以太坊区块链进行交互
money-legos：以更简单的方式访问DeFi协议
ethers-multicall：multicall调用的封装包
首先使用以下命令创建项目：

npm init -y
然后安装上述依赖包：

npm install -S @studydefi/money-legos ethers ethers-multicall
2、multicall对比测试：导入NPM依赖包
无论是否使用multicall，我们都要使用一些公共的依赖包并实例化提供程序以便连接区块链。 我们通过以下代码实现这一点：

const { ethers } = require("ethers");
const { ALCHEMY_URL } = require('./config')
const compound = require("@ studydefi/money-legos/compound");
const { accounts } = require("./accounts");
const { Contract, Provider } = require('ethers-multicall');
const provider = new ethers.providers.JsonRpcProvider(ALCHEMY_URL);
接下来创建一个函数，以便显示测试结果和执行时间，如下所示：

const calculateTime = async () => {
  const startDate = new Date();
  const result = await getLiquidity()
  const endDate = new Date();
  const milliseconds = (endDate.getTime() - startDate.getTime());
  console.log(`Time to process in milliseconds: $ {milliseconds}`)
  console.log(`Time to process in seconds: $ {milliseconds / 1000}`)
  const callsCount = Object.keys(result).length;
  console.log(`Number of entries in the result: $ {callsCount}`);
}
3、multicall对比测试：传统方式循环调用合约
为了使用传统方法进行测试，我们将遍历包含1000个地址的数组，然后在一个map循环中，逐个返回每个地址的查询结果。为此，可以实现如下函数代码：

const getLiquidity = () => {
  const compoundContract = new ethers.Contract(
  compound.comptroller.address,
  compound.comptroller.abi,
  provider
  )
  
  return Promise.all(accounts.map(account => {
  let data
  try {
     data = compoundContract.getAccountLiquidity(account.id)
  } catch (error) {
     console.log(`Error getting the data $ {error}`)
  }
     return data
  }))
}
在上面的代码中，我们实例化compound协议的comptroller合约，然后为每个地址调用 帐户流动性查询函数。

4、multicall对比测试：使用multicall调用的优化实现
使用Multicall调用时，上述代码需要稍作更改：

const getLiquidity = async () => {
  const ethcallProvider = new Provider(provider);
  await ethcallProvider.init();
  
  const compoundContract = new Contract(
    compound.comptroller.address,
    compound.comptroller.abi,
  )
  
  const contractCalls = accounts.map(account => compoundContract.getAccountLiquidity(account.id))
  const results = await ethcallProvider.all(contractCalls);
  return results
}
在上面的代码中，我们利用了Multicall包中的Provider和Contract类。

首先，利用compound协议的comptroller合约的地址和abi初始化提供程序。 然后将地址数组映射为合约调用数组，并在最后将得到的合约调用数组传入 提供器的all函数，这时请求才真正发送到网络中。

5、结果分析
让我们看一下执行时间的差异。

普通方法（不使用multicall）：

处理时间：124.653 秒
结果条目：1000
改进方法（使用mutlicall）：

处理时间：9.591 秒
结果条目：1000
6、结论
正如我们所看到的，使用multicall时执行时间的减少是相当可观的，从124秒减少到9.5秒， 花费的时间减少到原来的大约十分之一。

此外，如果我们比较eth_call请求数量，也会看到非常明显的减少，从上千个减少到 只有一个。因此，如果我们依赖以太坊的访问提供商，而在该提供商中对API的调用 受到限制，则需要考虑这一点。