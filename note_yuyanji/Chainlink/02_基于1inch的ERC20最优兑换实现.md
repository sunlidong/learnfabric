1、用1inch实现代币兑换的主要步骤
要利用1inch智能合约进行代币兑换很简单，主要步骤如下：

估算如何利用多DEX获得最大收益的兑换方案
授权1inch合约操作你的代币
利用第一步获得的兑换方案进行交易
首先让我们看一下1inch的智能合约，其中最重要的两个方法是：

getExpectedReturn() - 估算最优兑换方案
swap() - 执行多DEX兑换方案
2、getExpectedReturn - 估算最佳兑换方案
getExpectedReturn函数不会影响链上状态，不需要手续费， 因此你可以根据自己的需要随意调用，只要节点在线就可以。

这个函数需要传入兑换参数，然后返回兑换的期望结果数据， 其中包含了1inch将总的兑换数量如何分配到多个DEX上的分布描述。

function getExpectedReturn(
        IERC20 fromToken,
        IERC20 toToken,
        uint256 amount,
        uint256 parts,
        uint256 disableFlags
) public view
returns(
         uint256 returnAmount,
         uint256[] memory distribution
);
getExpectedReturn()函数的5个参数说明如下：

fromToken：要卖出代币的地址
toToken：要买入代币的地址
amount：要卖出代币的数量
parts：卖出数量拆分成多少份进行最优分布的估算。默认值为100
disableFlags：标志位，用于调整1inch的算法，例如可设置是否禁用某个特定的DEX
getExpectedReturn()函数返回2个值：

returnAmount：执行兑换交易后将得到的买入代币的数量
distribution：兑换交易在各DEX上的分布数组，各成员和为parts参数指定的值。例如 假设parts设置为100并且设置只使用Kyber和Uniswap，那么distribution可能看起来就是 这样：[75, 25, 0, 0, …]。
目前1inch支持的交易所和排序（与distribution对应）如下：

[
    "Uniswap",
    "Kyber",
    "Bancor",
    "Oasis",
    "CurveCompound",
    "CurveUsdt",
    "CurveY",
    "Binance",
    "Synthetix",
    "UniswapCompound",
    "UniswapChai",
    "UniswapAave"
]
需要指出的是，如果你希望卖出ETH而不是ERC20代币，那么应当将fromToken 参数设置为特殊的值：0x0 或 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE。

getExpectedReturn函数的返回值非常重要，因为接下来需要利用它 来执行实际的链上兑换操作。

3、swap - 执行多DEX兑换交易
要执行链上代币兑换交易，就需要使用1inch合约提供的另一个函数swap。 调用swap时，需要传入我们之前从getExpectedReturn返回的数据，并且承担 必要的gas开销。如果要卖出的是ERC20代币，那么还需要先授权1inch合约 可以操作你持有的待卖出代币。swap函数的定义如下：

function swap(
        IERC20 fromToken,
        IERC20 toToken,
        uint256 amount,
        uint256 minReturn,
        uint256[] memory distribution,
        uint256 disableFlags
 ) public payable;
swap函数的6个参数说明如下：

fromToken：待卖出代币的地址
toToken：待买入代币的地址
amount：待卖出代币的数量
minReturn：期望得到的待买入代币的最少数量，当实际交易结果低于该值时将回滚交易
distribution：兑换交易拆分分布数组，取自getExpectedReturn返回值
parts：执行估算时的拆分数量，同getExpectedReturn的参数
disableFlags：估算算法参数，同getExpectedReturn的参数
4、1inch实战开发环境搭建
为了进行下面的实战，我们需要一个方便的开发环境 —— 直接在主网进行操作 需要花费真金白银，成本太高了。因此我们使用ganache-cli来分叉主链并 解锁某个已经持有DAI代币的账号，例如 0x78bc49be7bae5e0eec08780c86f0e8278b8b035b。 出于简化，我们也可以在开发过程中把gas上限设置的高一些，而不是在每个交易 前估算gas成本。

下面的命令加载本地分叉链：

ganache-cli  -f https://mainnet.infura.io/v3/[YOUR INFURA KEY] \
             -d -i 66 \
             --unlock 0x78bc49be7bae5e0eec08780c86f0e8278b8b035b \
             -l 8000000
5、实战1inch - 估算最佳兑换方案
首先我们用web3.js来通过1inch估算1 ETH兑换为DAI的最佳方案。 代码如下：

var Web3 = require('web3');
const BigNumber = require('bignumber.js');

const oneSplitABI = require('./abis/onesplit.json');
const onesplitAddress = "0xC586BeF4a0992C495Cf22e1aeEE4E446CECDee0E"; // 1plit contract address on Main net

const fromToken = '0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE'; // ETHEREUM
const fromTokenDecimals = 18;

const toToken = '0x6b175474e89094c44da98b954eedeac495271d0f'; // DAI Token
const toTokenDecimals = 18;

const amountToExchange = 1

const web3 = new Web3('http://127.0.0.1:8545');

const onesplitContract = new web3.eth.Contract(oneSplitABI, onesplitAddress);

const oneSplitDexes = [
    "Uniswap",
    "Kyber",
    "Bancor",
    "Oasis",
    "CurveCompound",
    "CurveUsdt",
    "CurveY",
    "Binance",
    "Synthetix",
    "UniswapCompound",
    "UniswapChai",
    "UniswapAave"
]


onesplitContract.methods.getExpectedReturn(fromToken, toToken, new BigNumber(amountToExchange).shiftedBy(fromTokenDecimals).toString(), 100, 0).call({ from: '0x9759A6Ac90977b93B58547b4A71c78317f391A28' }, function (error, result) {
    if (error) {
        console.log(error)
        return;
    }
    console.log("Trade From: " + fromToken)
    console.log("Trade To: " + toToken);
    console.log("Trade Amount: " + amountToExchange);
    console.log(new BigNumber(result.returnAmount).shiftedBy(-fromTokenDecimals).toString());
    console.log("Using Dexes:");
    for (let index = 0; index < result.distribution.length; index++) {
        console.log(oneSplitDexes[index] + ": " + result.distribution[index] + "%");
    }
});
第4行：加载ABI以便实例化web3.js和1inch合约实例

第19行：该数组指定要使用的DEX

第35行：调用getExpectedReturn函数获取兑换方案

如果你之前不太熟悉BigNumber.js这个库，可以查阅这篇文章。

上述代码执行后返回的结果类似下面这样：

1inch教程

也就是说，这时1inch给出的最佳兑换方案是通过Uniswap兑换96%，通过Bancor 兑换4%，这样可以得到148.47 DAI，这样比单独通过Uniswap或Bancor进行兑换 都划算。

6、实战1inch - 执行多DEX兑换方案
下面我们使用1inch聚合器将1000 DAI兑换为ETH。首先定义一些变量，例如合约地址、ABI等等。

var Web3 = require('web3');
const BigNumber = require('bignumber.js');

const oneSplitABI = require('./abis/onesplit.json');
const onesplitAddress = "0xC586BeF4a0992C495Cf22e1aeEE4E446CECDee0E"; // 1plit contract address on Main net

const erc20ABI = require('./abis/erc20.json');
const daiAddress = "0x6b175474e89094c44da98b954eedeac495271d0f"; // DAI ERC20 contract address on Main net

const fromAddress = "0x4d10ae710Bd8D1C31bd7465c8CBC3add6F279E81";

const fromToken = daiAddress;
const fromTokenDecimals = 18;

const toToken = '0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE'; // ETH
const toTokenDecimals = 18;

const amountToExchange = new BigNumber(1000);

const web3 = new Web3(new Web3.providers.HttpProvider('http://127.0.0.1:8545'));

const onesplitContract = new web3.eth.Contract(oneSplitABI, onesplitAddress);
const daiToken = new web3.eth.Contract(erc20ABI, fromToken);
同时写一个辅助函数来等待交易确认：

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

async function waitTransaction(txHash) {
    let tx = null;
    while (tx == null) {
        tx = await web3.eth.getTransactionReceipt(txHash);
        await sleep(2000);
    }
    console.log("Transaction " + txHash + " was mined.");
    return (tx.status);
}
和前面类似，我们使用getExpectedReturn先估算兑换方案：

async function getQuote(fromToken, toToken, amount, callback) {
    let quote = null;
    try {
        quote = await onesplitContract.methods.getExpectedReturn(fromToken, toToken, amount, 100, 0).call();
    } catch (error) {
        console.log('Impossible to get the quote', error)
    }
    console.log("Trade From: " + fromToken)
    console.log("Trade To: " + toToken);
    console.log("Trade Amount: " + amountToExchange);
    console.log(new BigNumber(quote.returnAmount).shiftedBy(-fromTokenDecimals).toString());
    console.log("Using Dexes:");
    for (let index = 0; index < quote.distribution.length; index++) {
        console.log(oneSplitDexes[index] + ": " + quote.distribution[index] + "%");
    }
    callback(quote);
}
接下来需要授权1inch可以操作我们持有的代币：

function approveToken(tokenInstance, receiver, amount, callback) {
    tokenInstance.methods.approve(receiver, amount).send({ from: fromAddress }, async function(error, txHash) {
        if (error) {
            console.log("ERC20 could not be approved", error);
            return;
        }
        console.log("ERC20 token approved to " + receiver);
        const status = await waitTransaction(txHash);
        if (!status) {
            console.log("Approval transaction failed.");
            return;
        }
        callback();
    })
}
注意，授权额度可以远远高于当前实际需要的数量，这样后续就不再需要执行这个环节的操作了。

接下来就可以调用1inch聚合器的swap函数了。在下面的代码中，我们在调用 swap函数执行交易后，使用前面编写的辅助函数waitTransaction来等待交易 确认，并在交易确认后，显示转出账户的eth余额和dai余额：

let amountWithDecimals = new BigNumber(amountToExchange).shiftedBy(fromTokenDecimals).toFixed()

getQuote(fromToken, toToken, amountWithDecimals, function(quote) {
    approveToken(daiToken, onesplitAddress, amountWithDecimals, async function() {
        // We get the balance before the swap just for logging purpose
        let ethBalanceBefore = await web3.eth.getBalance(fromAddress);
        let daiBalanceBefore = await daiToken.methods.balanceOf(fromAddress).call();
        onesplitContract.methods.swap(fromToken, toToken, amountWithDecimals, quote.returnAmount, quote.distribution, 0).send({ from: fromAddress, gas: 8000000 }, async function(error, txHash) {
            if (error) {
                console.log("Could not complete the swap", error);
                return;
            }
            const status = await waitTransaction(txHash);
            // We check the final balances after the swap for logging purpose
            let ethBalanceAfter = await web3.eth.getBalance(fromAddress);
            let daiBalanceAfter = await daiToken.methods.balanceOf(fromAddress).call();
            console.log("Final balances:")
            console.log("Change in ETH balance", new BigNumber(ethBalanceAfter).minus(ethBalanceBefore).shiftedBy(-fromTokenDecimals).toFixed(2));
            console.log("Change in DAI balance", new BigNumber(daiBalanceAfter).minus(daiBalanceBefore).shiftedBy(-fromTokenDecimals).toFixed(2));
        });
    });
});
最后的执行结果看起来类似下面这样：

1inch教程

正如你看到的，我们用1000 DAI换回来5.85 ETH。

7、1inch Dex聚合器使用小结
1inch提供了出色的链上DEX聚合实现，可以在一个交易内利用多个DEX实现最优的 兑换策略。1inch的API使用也很简单，只需要用getExpectedReturn估算兑换方案， 然后使用swap执行兑换方案，就可以得到最好的兑换结果。无论是钱包、套利机器人 还是其他DApp，都可以利用1inch来快速实现ETH/ERC20的兑换并且得到最佳的收益。1、用1inch实现代币兑换的主要步骤
要利用1inch智能合约进行代币兑换很简单，主要步骤如下：

估算如何利用多DEX获得最大收益的兑换方案
授权1inch合约操作你的代币
利用第一步获得的兑换方案进行交易
首先让我们看一下1inch的智能合约，其中最重要的两个方法是：

getExpectedReturn() - 估算最优兑换方案
swap() - 执行多DEX兑换方案
2、getExpectedReturn - 估算最佳兑换方案
getExpectedReturn函数不会影响链上状态，不需要手续费， 因此你可以根据自己的需要随意调用，只要节点在线就可以。

这个函数需要传入兑换参数，然后返回兑换的期望结果数据， 其中包含了1inch将总的兑换数量如何分配到多个DEX上的分布描述。

function getExpectedReturn(
        IERC20 fromToken,
        IERC20 toToken,
        uint256 amount,
        uint256 parts,
        uint256 disableFlags
) public view
returns(
         uint256 returnAmount,
         uint256[] memory distribution
);
getExpectedReturn()函数的5个参数说明如下：

fromToken：要卖出代币的地址
toToken：要买入代币的地址
amount：要卖出代币的数量
parts：卖出数量拆分成多少份进行最优分布的估算。默认值为100
disableFlags：标志位，用于调整1inch的算法，例如可设置是否禁用某个特定的DEX
getExpectedReturn()函数返回2个值：

returnAmount：执行兑换交易后将得到的买入代币的数量
distribution：兑换交易在各DEX上的分布数组，各成员和为parts参数指定的值。例如 假设parts设置为100并且设置只使用Kyber和Uniswap，那么distribution可能看起来就是 这样：[75, 25, 0, 0, …]。
目前1inch支持的交易所和排序（与distribution对应）如下：

[
    "Uniswap",
    "Kyber",
    "Bancor",
    "Oasis",
    "CurveCompound",
    "CurveUsdt",
    "CurveY",
    "Binance",
    "Synthetix",
    "UniswapCompound",
    "UniswapChai",
    "UniswapAave"
]
需要指出的是，如果你希望卖出ETH而不是ERC20代币，那么应当将fromToken 参数设置为特殊的值：0x0 或 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE。

getExpectedReturn函数的返回值非常重要，因为接下来需要利用它 来执行实际的链上兑换操作。

3、swap - 执行多DEX兑换交易
要执行链上代币兑换交易，就需要使用1inch合约提供的另一个函数swap。 调用swap时，需要传入我们之前从getExpectedReturn返回的数据，并且承担 必要的gas开销。如果要卖出的是ERC20代币，那么还需要先授权1inch合约 可以操作你持有的待卖出代币。swap函数的定义如下：

function swap(
        IERC20 fromToken,
        IERC20 toToken,
        uint256 amount,
        uint256 minReturn,
        uint256[] memory distribution,
        uint256 disableFlags
 ) public payable;
swap函数的6个参数说明如下：

fromToken：待卖出代币的地址
toToken：待买入代币的地址
amount：待卖出代币的数量
minReturn：期望得到的待买入代币的最少数量，当实际交易结果低于该值时将回滚交易
distribution：兑换交易拆分分布数组，取自getExpectedReturn返回值
parts：执行估算时的拆分数量，同getExpectedReturn的参数
disableFlags：估算算法参数，同getExpectedReturn的参数
4、1inch实战开发环境搭建
为了进行下面的实战，我们需要一个方便的开发环境 —— 直接在主网进行操作 需要花费真金白银，成本太高了。因此我们使用ganache-cli来分叉主链并 解锁某个已经持有DAI代币的账号，例如 0x78bc49be7bae5e0eec08780c86f0e8278b8b035b。 出于简化，我们也可以在开发过程中把gas上限设置的高一些，而不是在每个交易 前估算gas成本。

下面的命令加载本地分叉链：

ganache-cli  -f https://mainnet.infura.io/v3/[YOUR INFURA KEY] \
             -d -i 66 \
             --unlock 0x78bc49be7bae5e0eec08780c86f0e8278b8b035b \
             -l 8000000
5、实战1inch - 估算最佳兑换方案
首先我们用web3.js来通过1inch估算1 ETH兑换为DAI的最佳方案。 代码如下：

var Web3 = require('web3');
const BigNumber = require('bignumber.js');

const oneSplitABI = require('./abis/onesplit.json');
const onesplitAddress = "0xC586BeF4a0992C495Cf22e1aeEE4E446CECDee0E"; // 1plit contract address on Main net

const fromToken = '0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE'; // ETHEREUM
const fromTokenDecimals = 18;

const toToken = '0x6b175474e89094c44da98b954eedeac495271d0f'; // DAI Token
const toTokenDecimals = 18;

const amountToExchange = 1

const web3 = new Web3('http://127.0.0.1:8545');

const onesplitContract = new web3.eth.Contract(oneSplitABI, onesplitAddress);

const oneSplitDexes = [
    "Uniswap",
    "Kyber",
    "Bancor",
    "Oasis",
    "CurveCompound",
    "CurveUsdt",
    "CurveY",
    "Binance",
    "Synthetix",
    "UniswapCompound",
    "UniswapChai",
    "UniswapAave"
]


onesplitContract.methods.getExpectedReturn(fromToken, toToken, new BigNumber(amountToExchange).shiftedBy(fromTokenDecimals).toString(), 100, 0).call({ from: '0x9759A6Ac90977b93B58547b4A71c78317f391A28' }, function (error, result) {
    if (error) {
        console.log(error)
        return;
    }
    console.log("Trade From: " + fromToken)
    console.log("Trade To: " + toToken);
    console.log("Trade Amount: " + amountToExchange);
    console.log(new BigNumber(result.returnAmount).shiftedBy(-fromTokenDecimals).toString());
    console.log("Using Dexes:");
    for (let index = 0; index < result.distribution.length; index++) {
        console.log(oneSplitDexes[index] + ": " + result.distribution[index] + "%");
    }
});
第4行：加载ABI以便实例化web3.js和1inch合约实例

第19行：该数组指定要使用的DEX

第35行：调用getExpectedReturn函数获取兑换方案

如果你之前不太熟悉BigNumber.js这个库，可以查阅这篇文章。

上述代码执行后返回的结果类似下面这样：

1inch教程

也就是说，这时1inch给出的最佳兑换方案是通过Uniswap兑换96%，通过Bancor 兑换4%，这样可以得到148.47 DAI，这样比单独通过Uniswap或Bancor进行兑换 都划算。

6、实战1inch - 执行多DEX兑换方案
下面我们使用1inch聚合器将1000 DAI兑换为ETH。首先定义一些变量，例如合约地址、ABI等等。

var Web3 = require('web3');
const BigNumber = require('bignumber.js');

const oneSplitABI = require('./abis/onesplit.json');
const onesplitAddress = "0xC586BeF4a0992C495Cf22e1aeEE4E446CECDee0E"; // 1plit contract address on Main net

const erc20ABI = require('./abis/erc20.json');
const daiAddress = "0x6b175474e89094c44da98b954eedeac495271d0f"; // DAI ERC20 contract address on Main net

const fromAddress = "0x4d10ae710Bd8D1C31bd7465c8CBC3add6F279E81";

const fromToken = daiAddress;
const fromTokenDecimals = 18;

const toToken = '0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE'; // ETH
const toTokenDecimals = 18;

const amountToExchange = new BigNumber(1000);

const web3 = new Web3(new Web3.providers.HttpProvider('http://127.0.0.1:8545'));

const onesplitContract = new web3.eth.Contract(oneSplitABI, onesplitAddress);
const daiToken = new web3.eth.Contract(erc20ABI, fromToken);
同时写一个辅助函数来等待交易确认：

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

async function waitTransaction(txHash) {
    let tx = null;
    while (tx == null) {
        tx = await web3.eth.getTransactionReceipt(txHash);
        await sleep(2000);
    }
    console.log("Transaction " + txHash + " was mined.");
    return (tx.status);
}
和前面类似，我们使用getExpectedReturn先估算兑换方案：

async function getQuote(fromToken, toToken, amount, callback) {
    let quote = null;
    try {
        quote = await onesplitContract.methods.getExpectedReturn(fromToken, toToken, amount, 100, 0).call();
    } catch (error) {
        console.log('Impossible to get the quote', error)
    }
    console.log("Trade From: " + fromToken)
    console.log("Trade To: " + toToken);
    console.log("Trade Amount: " + amountToExchange);
    console.log(new BigNumber(quote.returnAmount).shiftedBy(-fromTokenDecimals).toString());
    console.log("Using Dexes:");
    for (let index = 0; index < quote.distribution.length; index++) {
        console.log(oneSplitDexes[index] + ": " + quote.distribution[index] + "%");
    }
    callback(quote);
}
接下来需要授权1inch可以操作我们持有的代币：

function approveToken(tokenInstance, receiver, amount, callback) {
    tokenInstance.methods.approve(receiver, amount).send({ from: fromAddress }, async function(error, txHash) {
        if (error) {
            console.log("ERC20 could not be approved", error);
            return;
        }
        console.log("ERC20 token approved to " + receiver);
        const status = await waitTransaction(txHash);
        if (!status) {
            console.log("Approval transaction failed.");
            return;
        }
        callback();
    })
}
注意，授权额度可以远远高于当前实际需要的数量，这样后续就不再需要执行这个环节的操作了。

接下来就可以调用1inch聚合器的swap函数了。在下面的代码中，我们在调用 swap函数执行交易后，使用前面编写的辅助函数waitTransaction来等待交易 确认，并在交易确认后，显示转出账户的eth余额和dai余额：

let amountWithDecimals = new BigNumber(amountToExchange).shiftedBy(fromTokenDecimals).toFixed()

getQuote(fromToken, toToken, amountWithDecimals, function(quote) {
    approveToken(daiToken, onesplitAddress, amountWithDecimals, async function() {
        // We get the balance before the swap just for logging purpose
        let ethBalanceBefore = await web3.eth.getBalance(fromAddress);
        let daiBalanceBefore = await daiToken.methods.balanceOf(fromAddress).call();
        onesplitContract.methods.swap(fromToken, toToken, amountWithDecimals, quote.returnAmount, quote.distribution, 0).send({ from: fromAddress, gas: 8000000 }, async function(error, txHash) {
            if (error) {
                console.log("Could not complete the swap", error);
                return;
            }
            const status = await waitTransaction(txHash);
            // We check the final balances after the swap for logging purpose
            let ethBalanceAfter = await web3.eth.getBalance(fromAddress);
            let daiBalanceAfter = await daiToken.methods.balanceOf(fromAddress).call();
            console.log("Final balances:")
            console.log("Change in ETH balance", new BigNumber(ethBalanceAfter).minus(ethBalanceBefore).shiftedBy(-fromTokenDecimals).toFixed(2));
            console.log("Change in DAI balance", new BigNumber(daiBalanceAfter).minus(daiBalanceBefore).shiftedBy(-fromTokenDecimals).toFixed(2));
        });
    });
});
最后的执行结果看起来类似下面这样：

1inch教程

正如你看到的，我们用1000 DAI换回来5.85 ETH。

7、1inch Dex聚合器使用小结
1inch提供了出色的链上DEX聚合实现，可以在一个交易内利用多个DEX实现最优的 兑换策略。1inch的API使用也很简单，只需要用getExpectedReturn估算兑换方案， 然后使用swap执行兑换方案，就可以得到最好的兑换结果。无论是钱包、套利机器人 还是其他DApp，都可以利用1inch来快速实现ETH/ERC20的兑换并且得到最佳的收益。