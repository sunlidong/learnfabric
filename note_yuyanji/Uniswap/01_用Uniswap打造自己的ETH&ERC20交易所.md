在这个教程中，我们将简要地谈论Uniswap的工作原理，并利用Uniswap协议构建 一个简单的Ether/ERC20兑换程序，你可以在此基础上继续扩展，从而让用户可以 在你的Dapp或钱包中轻松的实现Ether/ERC20的兑换。

用自己熟悉的语言学习 以太坊DApp开发 ： Java | Php | Python | .Net / C# | Golang | Node.JS | Flutter / Dart

1、Uniswap概述
Uniswap是用于Ether和ERC20代币的去中心化交换协议。整个过程在链上进行，没有网守。 每个ERC20代币都有一个单独的交换智能合约。这些交换合约持有以太币及其相关的ERC20的储备金。 交易者可以随时根据储备进行交易。他们将以太币发送给合同，并获得ERC20代币，反之亦然。 这些准备金由流动性提供者存放，他们在每次交易中收取费用。也可以在ERC20令牌之间进行交换。

交易价格由定价公式自动确定：常量产品做市商模型。计算得出它总是恒定的。 此公式也适用于始终恒定的ERC20代币交换。

假设你有10ETH与1,000 DAI交换的合约。那么常数将是10 1000 = 10,000（x y = k）。 如果你发送1ETH给合约，那么现在就11ETH在合约中。智能合约规定，以太币乘以合同中的Dai 必须保持不变。所以10,000 / 11 ≒ 909DAI应该在合约中。这也意味着合约不必具有1000DAI， 因此它将返回1000 – 909 = 91DAI给买方，这是合约中现有金额与应包含的金额之间的差。

该公式通常也称为x乘以y等于k公式。下图显示了价格如何根据x和y的值而变化：



Uniswap通过其储备系统和自动定价公式降低了协调买卖双方的成本。结果，交易者可以更容易，更快地交换代币。

好吧，让我们开始使用我们的应用程序。我们将构建一个交换Ether和Dai的简单nodejs应用程序。 你可以在github下载最终完成的代码。不过 在着手具体的代码开发之前，我们需要先准备好开发环境。

2、Ether/ERC20 DEX应用开发环境准备
2.1 生成测试私钥和地址
首先，从vanity-eth获取测试用的私钥和地址。点击“生成”按钮即可。 注意，除测试目的外，请勿使用网页生成你的私钥和地址。

2.2 获取测试用的以太币
如果您在Rinkeby中没有任何以太币，可以利用以太币水龙头获得一些。 你可以通过搜索地址在Rinkeby Etherscan中检查余额。0.1ETH就足够了。

2.3 创建一个新的Npm项目
mkdir eht-uniswap-demo
cd eht-uniswap-demo/
npm init --yes
如果没有npm和nodejs，可以从这里下载。

2.4 初始化git仓库
很简单，就一句话：

git init
2.5 安装web3软件包
npm i --save web3@1.2.1
web3是以太坊JavaScript API。它是以太坊网络（包括测试网）的接口。 它与以太坊节点通信或通过JavaScript应用程序与部署在区块链上的智能合约进行交易。

2.6 获取Infura端点
简单起见，我们将使用Infura作为与以太坊网络进行通信 的服务提供商，而不是自己从头搭建一个以太坊节点。创建一个Infura帐户， 并创建一个新项目就可以获得免费的Rinkeby的API端点。它看起来应该像这样：

https://rinkeby.infura.io/v3/YOUR_PROJECT_ID
2.7 安装ethereumjs-tx软件包
npm i --save ethereumjs-tx@1.3.7
我们使用ethereumjs-tx创建并签署交易。

3、用Ether兑换DAI
现在我们将开始编写代码实现用Ether兑换DAI。你可以点击这里查看完成的代码。

3.1 创建新文件
创建一个新文件EthToDaiRinkeby.mjs。mjs是文件扩展名，允许你使用ES6语法编写代码。

3.2 导入包
import Web3 from "web3";
import EthTx from "ethereumjs-tx";
3.3 声明合约地址和Abi
下面是Rinkeby测试网中的交换合约的地址和Dai合约的abi，你可以使用它们与交换智能合约进行交互。

Uniswap DAI交易所合约地址：

const daiExchangeAddress = "0x77dB9C915809e7BE439D2AB21032B1b8B58F6891";
你可以在Etherscan查看合约的详细信息。

交换合约abi：

const daiExchangeAbi = '[{“name”: “TokenPurchase”, “inputs”: [{“type”: “address”, “name”: “buyer”, “indexed”: true}, {“type”: “uint256”, “name”: “eth_sold”, “indexed”: true}, {“type”: “uint256”, “name”: “tokens_bought”, “indexed”: true}], “anonymous”: false, “type”: “event”}, {“name”: “EthPurchase”, “inputs”: [{“type”: “address”, “name”: “buyer”, “indexed”: true}, {“type”: “uint256”, “name”: “tokens_sold”, “indexed”: true}, {“type”: “uint256”, “name”: “eth_bought”, “indexed”: true}], “anonymous”: false, “type”: “event”}, {“name”: “AddLiquidity”, “inputs”: [{“type”: “address”, “name”: “provider”, “indexed”: true}, {“type”: “uint256”, “name”: “eth_amount”, “indexed”: true}, {“type”: “uint256”, “name”: “token_amount”, “indexed”: true}], “anonymous”: false, “type”: “event”}, {“name”: “RemoveLiquidity”, “inputs”: [{“type”: “address”, “name”: “provider”, “indexed”: true}, {“type”: “uint256”, “name”: “eth_amount”, “indexed”: true}, {“type”: “uint256”, “name”: “token_amount”, “indexed”: true}], “anonymous”: false, “type”: “event”}, {“name”: “Transfer”, “inputs”: [{“type”: “address”, “name”: “_from”, “indexed”: true}, {“type”: “address”, “name”: “_to”, “indexed”: true}, {“type”: “uint256”, “name”: “_value”, “indexed”: false}], “anonymous”: false, “type”: “event”}, {“name”: “Approval”, “inputs”: [{“type”: “address”, “name”: “_owner”, “indexed”: true}, {“type”: “address”, “name”: “_spender”, “indexed”: true}, {“type”: “uint256”, “name”: “_value”, “indexed”: false}], “anonymous”: false, “type”: “event”}, {“name”: “setup”, “outputs”: [], “inputs”: [{“type”: “address”, “name”: “token_addr”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 175875}, {“name”: “addLiquidity”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “min_liquidity”}, {“type”: “uint256”, “name”: “max_tokens”}, {“type”: “uint256”, “name”: “deadline”}], “constant”: false, “payable”: true, “type”: “function”, “gas”: 82605}, {“name”: “removeLiquidity”, “outputs”: [{“type”: “uint256”, “name”: “out”}, {“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “amount”}, {“type”: “uint256”, “name”: “min_eth”}, {“type”: “uint256”, “name”: “min_tokens”}, {“type”: “uint256”, “name”: “deadline”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 116814}, {“name”: “__default__”, “outputs”: [], “inputs”: [], “constant”: false, “payable”: true, “type”: “function”}, {“name”: “ethToTokenSwapInput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “min_tokens”}, {“type”: “uint256”, “name”: “deadline”}], “constant”: false, “payable”: true, “type”: “function”, “gas”: 12757}, {“name”: “ethToTokenTransferInput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “min_tokens”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “recipient”}], “constant”: false, “payable”: true, “type”: “function”, “gas”: 12965}, {“name”: “ethToTokenSwapOutput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_bought”}, {“type”: “uint256”, “name”: “deadline”}], “constant”: false, “payable”: true, “type”: “function”, “gas”: 50463}, {“name”: “ethToTokenTransferOutput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_bought”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “recipient”}], “constant”: false, “payable”: true, “type”: “function”, “gas”: 50671}, {“name”: “tokenToEthSwapInput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_sold”}, {“type”: “uint256”, “name”: “min_eth”}, {“type”: “uint256”, “name”: “deadline”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 47503}, {“name”: “tokenToEthTransferInput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_sold”}, {“type”: “uint256”, “name”: “min_eth”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “recipient”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 47712}, {“name”: “tokenToEthSwapOutput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “eth_bought”}, {“type”: “uint256”, “name”: “max_tokens”}, {“type”: “uint256”, “name”: “deadline”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 50175}, {“name”: “tokenToEthTransferOutput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “eth_bought”}, {“type”: “uint256”, “name”: “max_tokens”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “recipient”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 50384}, {“name”: “tokenToTokenSwapInput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_sold”}, {“type”: “uint256”, “name”: “min_tokens_bought”}, {“type”: “uint256”, “name”: “min_eth_bought”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “token_addr”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 51007}, {“name”: “tokenToTokenTransferInput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_sold”}, {“type”: “uint256”, “name”: “min_tokens_bought”}, {“type”: “uint256”, “name”: “min_eth_bought”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “recipient”}, {“type”: “address”, “name”: “token_addr”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 51098}, {“name”: “tokenToTokenSwapOutput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_bought”}, {“type”: “uint256”, “name”: “max_tokens_sold”}, {“type”: “uint256”, “name”: “max_eth_sold”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “token_addr”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 54928}, {“name”: “tokenToTokenTransferOutput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_bought”}, {“type”: “uint256”, “name”: “max_tokens_sold”}, {“type”: “uint256”, “name”: “max_eth_sold”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “recipient”}, {“type”: “address”, “name”: “token_addr”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 55019}, {“name”: “tokenToExchangeSwapInput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_sold”}, {“type”: “uint256”, “name”: “min_tokens_bought”}, {“type”: “uint256”, “name”: “min_eth_bought”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “exchange_addr”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 49342}, {“name”: “tokenToExchangeTransferInput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_sold”}, {“type”: “uint256”, “name”: “min_tokens_bought”}, {“type”: “uint256”, “name”: “min_eth_bought”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “recipient”}, {“type”: “address”, “name”: “exchange_addr”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 49532}, {“name”: “tokenToExchangeSwapOutput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_bought”}, {“type”: “uint256”, “name”: “max_tokens_sold”}, {“type”: “uint256”, “name”: “max_eth_sold”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “exchange_addr”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 53233}, {“name”: “tokenToExchangeTransferOutput”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_bought”}, {“type”: “uint256”, “name”: “max_tokens_sold”}, {“type”: “uint256”, “name”: “max_eth_sold”}, {“type”: “uint256”, “name”: “deadline”}, {“type”: “address”, “name”: “recipient”}, {“type”: “address”, “name”: “exchange_addr”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 53423}, {“name”: “getEthToTokenInputPrice”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “eth_sold”}], “constant”: true, “payable”: false, “type”: “function”, “gas”: 5542}, {“name”: “getEthToTokenOutputPrice”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_bought”}], “constant”: true, “payable”: false, “type”: “function”, “gas”: 6872}, {“name”: “getTokenToEthInputPrice”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “tokens_sold”}], “constant”: true, “payable”: false, “type”: “function”, “gas”: 5637}, {“name”: “getTokenToEthOutputPrice”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “uint256”, “name”: “eth_bought”}], “constant”: true, “payable”: false, “type”: “function”, “gas”: 6897}, {“name”: “tokenAddress”, “outputs”: [{“type”: “address”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 1413}, {“name”: “factoryAddress”, “outputs”: [{“type”: “address”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 1443}, {“name”: “balanceOf”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “_owner”}], “constant”: true, “payable”: false, “type”: “function”, “gas”: 1645}, {“name”: “transfer”, “outputs”: [{“type”: “bool”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “_to”}, {“type”: “uint256”, “name”: “_value”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 75034}, {“name”: “transferFrom”, “outputs”: [{“type”: “bool”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “_from”}, {“type”: “address”, “name”: “_to”}, {“type”: “uint256”, “name”: “_value”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 110907}, {“name”: “approve”, “outputs”: [{“type”: “bool”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “_spender”}, {“type”: “uint256”, “name”: “_value”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 38769}, {“name”: “allowance”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “_owner”}, {“type”: “address”, “name”: “_spender”}], “constant”: true, “payable”: false, “type”: “function”, “gas”: 1925}, {“name”: “name”, “outputs”: [{“type”: “bytes32”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 1623}, {“name”: “symbol”, “outputs”: [{“type”: “bytes32”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 1653}, {“name”: “decimals”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 1683}, {“name”: “totalSupply”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 1713}]';
声明地址和私钥变量

const addressFrom = "[YOUR_ADDRESS]";
const privKey = "[YOUR_PRIVATE_KEY]";
3.4 设置Web3
设置Web3以将Infura用作Web3的提供器：

const web3 = new Web3(
  new Web3.providers.HttpProvider("https://rinkeby.infura.io/v3/[YOUR_PROJECT_ID]"
  )
);
3.5 实例化合约对象
可以使用Contract带有地址和abi的web3方法来实例化合同。将这些值作为参数传递给web3.eth.Contract：

const daiExchangeContract = new web3.eth.Contract(JSON.parse(daiExchangeAbi), daiExchangeAddress);
现在我们准备调用交换合约功能来与Dai交换以太币。

3.6 声明ETH_SOLDConst变量
首先声明一个const变量以指定要与Dai交换的以太币的数量：

const ETH_SOLD = web3.utils.toHex(50000000000000000); // 0.05ETH
3.7 为ethToTokenSwapInput功能编码Abi
我们使用合约的ethToTokenSwapInput方法与Dai交换以太币。

function ethToTokenSwapInput(uint256 min_tokens, uint256 deadline) external payable returns (uint256 tokens_bought);
该方法需要两个参数：min_tokens和deadline。min_tokens参数指定购买的Dai的最小数量。 deadline声明了交易的最后期限，意思是此交易只能在此时间之前执行。

这些选项可以减轻矿工抢占先机的风险。假设你卖出1ETH，当前价格为200 DAI。你可以说： 如果我得到的金额少于199 DAI，那么我就希望交易失败。 恶意的领跑者可能会将价格推高， 但由于你已经制定了交易的执行价位，因此可以不受影响。截止日期可以防止矿工在对其更有利 的时候执行交易，从而帮助你减轻风险。可以访问这里 进一步了解有关交易抢跑的信息。

我们声明一个传递给min_tokens参数的const变量：

const MIN_TOKENS = web3.utils.toHex(0.2 * 10 ** 18); // 0.2 DAI
你应该选择一个不太低的金额，因为太低了就容易被抢跑者利用。但是也不能太高， 因为太高的话，你的交易就很容易失败。

我们声明另一个传递给deadline参数的const变量。下面的数字是的Unix时间戳 10/01/2019 @ 12:00am (UTC)：

const DEADLINE = 1569888000; // 10/01/2019 @ 12:00am (UTC)
你可以使用这个在线工具将时间转换为unix时间戳。

现在，将两个const变量作为函数的参数传入ethToTokenSwapInput并为其编码abi：

const exchangeEncodedABI = daiExchangeContract.methods.ethToTokenSwapInput(MIN_TOKENS, DEADLINE).encodeABI();
3.8 声明sendSignedTx函数
接下来，创建一个函数来签名交易对象，然后将其广播到以太坊网络：

function sendSignedTx(transactionObject, cb) {
  let transaction = new EthTx(transactionObject);
  const privateKey = new Buffer.from(privKey, "hex");
  transaction.sign(privateKey);
  const serializedEthTx = transaction.serialize().toString("hex");
  web3.eth.sendSignedTransaction(`0x${serializedEthTx}`, cb);
}
构造一个事务对象然后执行 sendSignedTx

web3.eth.getTransactionCount(addressFrom).then(transactionNonce => {
  const transactionObject = {
    chainId: 4,
    nonce: web3.utils.toHex(transactionNonce),
    gasLimit: web3.utils.toHex(6000000),
    gasPrice: web3.utils.toHex(10000000000),
    to: daiExchangeAddress,
    from: addressFrom,
    data: exchangeEncodedABI,
    value: ETH_SOLD
  };

sendSignedTx(transactionObject, function(error, result){
    if(error) return console.log("error ===> ", error);
    console.log("sent ===> ", result);
  })
}
);
3.9 运行代码
npm run eth-to-dai-rinkeby
一旦获得了这样的交易哈希：0xbbc617c9…….92bec4839e10， 就可以转到Etherscan查看交易的详细信息。下面以我的交易 为例：



如果交易成功确认，你就可以从Erc20代币的Txns选项卡中看到在你的地址中收到了一些DAI。



4、用dai兑换ether
接下来让我们用DAI兑换ether。这会略微复杂一些。我们将进行两次交易。首先， 我们需要授权Uniswap Dai交易合约可以代表我们的帐户消费Dai。第二，我们调用 合约的tokenToEthSwapInput函数将DAI兑换为Ether。

4.1 授权Uniswap DAI交换合约
我们为此制作另一个文件：approveDaiExchangeRinkeby.mjs。由于大多数代码都是相同的，因此我将跳过细节。 你可以在此处查看完成的代码。

4.2 声明代币合约地址和Abi变量
这是Rinkeby测试网中的Dai代币合约的地址和abi：

代币合约地址：

const daiTokenAddress = “0x2448eE2641d78CC42D7AD76498917359D961A783”;
代币合约abi：

const daiTokenAbi = '[{“name”: “Transfer”, “inputs”: [{“type”: “address”, “name”: “_from”, “indexed”: true}, {“type”: “address”, “name”: “_to”, “indexed”: true}, {“type”: “uint256”, “name”: “_value”, “indexed”: false}], “anonymous”: false, “type”: “event”}, {“name”: “Approval”, “inputs”: [{“type”: “address”, “name”: “_owner”, “indexed”: true}, {“type”: “address”, “name”: “_spender”, “indexed”: true}, {“type”: “uint256”, “name”: “_value”, “indexed”: false}], “anonymous”: false, “type”: “event”}, {“outputs”: [], “inputs”: [{“type”: “string”, “name”: “_name”}, {“type”: “string”, “name”: “_symbol”}, {“type”: “uint256”, “name”: “_decimals”}, {“type”: “uint256”, “name”: “_supply”}], “constant”: false, “payable”: false, “type”: “constructor”}, {“name”: “transfer”, “outputs”: [{“type”: “bool”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “_to”}, {“type”: “uint256”, “name”: “_value”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 74020}, {“name”: “transferFrom”, “outputs”: [{“type”: “bool”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “_from”}, {“type”: “address”, “name”: “_to”}, {“type”: “uint256”, “name”: “_value”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 110371}, {“name”: “approve”, “outputs”: [{“type”: “bool”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “_spender”}, {“type”: “uint256”, “name”: “_value”}], “constant”: false, “payable”: false, “type”: “function”, “gas”: 37755}, {“name”: “name”, “outputs”: [{“type”: “string”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 6402}, {“name”: “symbol”, “outputs”: [{“type”: “string”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 6432}, {“name”: “decimals”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 663}, {“name”: “totalSupply”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [], “constant”: true, “payable”: false, “type”: “function”, “gas”: 693}, {“name”: “balanceOf”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “arg0”}], “constant”: true, “payable”: false, “type”: “function”, “gas”: 877}, {“name”: “allowance”, “outputs”: [{“type”: “uint256”, “name”: “out”}], “inputs”: [{“type”: “address”, “name”: “arg0”}, {“type”: “address”, “name”: “arg1”}], “constant”: true, “payable”: false, “type”: “function”, “gas”: 1061}]';
注意：ERC20代币的ABI都是相同的。

4.3 声明daiExchangeAddress变量
const daiExchangeAddress = "0x77dB9C915809e7BE439D2AB21032B1b8B58F6891";
我们将授权此地址代表我们的帐户进行Dai的转账。

4.4 实例化Dai代币合约
const daiTokenContract = new web3.eth.Contract(
  JSON.parse(daiTokenAbi),
  daiTokenAddress
);
声明常量以传递给approve函数

const ADDRESS_SPENDER = daiExchangeAddress;
const TOKENS = web3.utils.toHex(1 * 10 ** 18); // 1 DAI
我们授权daiExchangeAddress可以操作我们账户的1 DAI。

4.5 Approve函数调用的ABI编码
const approveEncodedABI = daiTokenContract.methods
  .approve(ADDRESS_SPENDER, TOKENS)
  .encodeABI();
4.6 声明sendSignedTx函数
下面代码创建一个函数来签名交易对象，然后将其广播到以太坊网络：

function sendSignedTx(transactionObject, cb) {
  let transaction = new EthTx(transactionObject);
  const privateKey = new Buffer.from(privKey, "hex");
  transaction.sign(privateKey);
  const serializedEthTx = transaction.serialize().toString("hex");
  web3.eth.sendSignedTransaction(`0x${serializedEthTx}`, cb);
}
下面代码构造一个事务对象然后执行 sendSignedTx：

web3.eth.getTransactionCount(addressFrom).then(transactionNonce =&gt; {
  const transactionObject = {
    chainId: 4,
    nonce: web3.utils.toHex(transactionNonce),
    gasLimit: web3.utils.toHex(42000),
    gasPrice: web3.utils.toHex(5000000),
    to: daiTokenAddress,
    from: addressFrom,
    data: approveEncodedABI
  };

sendSignedTx(transactionObject, function(error, result){
    if(error) return console.log("error ===&gt;", error);
    console.log("sent ===&gt;", result);
  })
}
);
4.7 运行代码文件
npm run approve-dai-exchange-rinkeby
一旦获得了这样的交易哈希：0x1d16e0……1f1d0f44e5262d0e8f018a， 就可以转到Etherscan查看交易的详细信息。下面以我的交易 为例：



4.8 用DAI兑换以太币
最后，让我们完成用DAI兑换以太币的代码。我们将跳过细节，因为大多数代码将是相同的。 创建另一个文件DaiToEthRinkeby.mjs，然后调用合约的tokenToEthSwapInput函数。

function tokenToEthSwapInput(uint256 tokens_sold, uint256 min_eth, uint256 deadline) external returns (uint256 eth_bought);
该函数需要三个参数。tokens_sold是要售出的ERC20代币的数量。min_eth是至少应该得到以太币数量。 deadline是交易截止时间戳。

const TOKENS_SOLD = web3.utils.toHex(0.4 * 10 ** 18); // 0.4DAI
const MIN_ETH = web3.utils.toHex(5000000000000000); // 0.005ETH
const DEADLINE = 1569888000; // 10/01/2019 @ 12:00am (UTC)
现在，将三个const变量作为函数的参数传入tokenToEthSwapInput并进行ABI编码：

const tokenToEthEncodedABI = daiExchangeContract.methods
  .tokenToEthSwapInput(TOKENS_SOLD, MIN_ETH, DEADLINE)
  .encodeABI();
4.9 运行代码文件
运行文件：

npm run dai-to-eth-rinkeby
获得交易哈希后，你就可以转到Etherscan查看交易的详细信息。下面以我的交易 为例：



现在，你可以看到你的以太坊地址中DAI减少了：



并且以太币被添加到你的地址：



5、集成其他ERC20代币
如果要集成其他ERC20代币以进行交换，只需要更改代码中的代币地址就可以实现了。

