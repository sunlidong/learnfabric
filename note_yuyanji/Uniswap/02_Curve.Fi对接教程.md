Curve.Fi是一组规模庞大的DEFi协议，这个教程的目的是帮助 你关注智能合约开发的现代方法，以及如何在自己的Defi 应用中集成Curve.Fi这种大型协议的关键点。

用自己熟悉的语言学习 以太坊智能合约与DApp开发 ： Java | Php | Python | .Net / C# | Golang | Node.JS | Flutter / Dart

1、Curve.Fi开发教程的组织方式
是的，我已经说过我们将深入研究代码。尽管我们不是在谈论AMM模型的数学运算， 也不是在Curve.Fi合约中使用的其他一些特定技巧，但是我们仍然需要了解 一些内容。当然，这不是在谈论成功的DeFi开发-我没有给出提示。这是我经验的一部分， 在这里我将分享一些有关Curve智能合约的基本架构以及我们如何与这些架构集成的知识。 实际上，下图中包含了本教程的全部内容：

curve.fi协议集成教程

我们研究的是Y-Pool，它以一种简单明了的方式组织。存款（Deposit）合约将你的资金 打包成Y-Token，然后将其发送到兑换（Swap）合约，兑换合约为用户生成LP代币。 如果你要提取流通性份额以及所赚取的收益，则合约会执行相同的操作，但是方式相反。 这一切都与金融合约有关（当然，如果我们对池平衡和股数计算背后的复杂逻辑选择视而不见）。

此外，我们还有Gauge和Minter合同，它们是Curve.Fi DAO的一部分。这些合约使你可以 获得CRV代币奖励并将其抵押，以便获得协议治理的投票权。

在我们充分了解这些需要交互的合约后，就可以开始我们的开发了。

2、从Curve.Fi用户交互界面开始
你可能还记得，我们正在处理一些使用Vyper开发的外部合约。为了与这些合约交互，我们需要 创建一些接口。这并不是最困难的工作：只需要仔细查看原始合约并挑出将要调用的方法即可。

让我们将从存款合约开始。我们需要资金进出的方法，以及一些数据查看界面（例如，查看已 注册的代币及其Y对应代币）。

contract ICurveFi_DepositY { 
    function add_liquidity(uint256[4] calldata uamounts, uint256 min_mint_amount) external;
    function remove_liquidity(uint256 _amount, uint256[4] calldata min_uamounts) external;
    function remove_liquidity_imbalance(uint256[4] calldata uamounts, uint256 max_burn_amount) external;

    function coins(int128 i) external view returns (address);
    function underlying_coins(int128 i) external view returns (address);
    function underlying_coins() external view returns (address[4] memory);
    function curve() external view returns (address);
    function token() external view returns (address);
}
然后用相同的方法处理兑换合约（你可能还记得，该合约为用户铸造LP代币）。

contract ICurveFi_Gauge {
    function lp_token() external view returns(address);
    function crv_token() external view returns(address);
 
    function balanceOf(address addr) external view returns (uint256);
    function deposit(uint256 _value) external;
    function withdraw(uint256 _value) external;

    function claimable_tokens(address addr) external returns (uint256);
    function minter() external view returns(address); //use minter().mint(gauge_addr) to claim CRV

    function integrate_fraction(address _for) external view returns(uint256);
    function user_checkpoint(address _for) external returns(bool);
}
现在该处理DAO的主要合约-流动性计量器合约。它执行复杂的功能，但我们主要需要用于 创建检查点的方法以及查看Minter和CRV代币地址的界面。

contract ICurveFi_Gauge {
    function lp_token() external view returns(address);
    function crv_token() external view returns(address);
 
    function balanceOf(address addr) external view returns (uint256);
    function deposit(uint256 _value) external;
    function withdraw(uint256 _value) external;

    function claimable_tokens(address addr) external returns (uint256);
    function minter() external view returns(address); //use minter().mint(gauge_addr) to claim CRV

    function integrate_fraction(address _for) external view returns(uint256);
    function user_checkpoint(address _for) external returns(bool);
}
还有从用户界面的角度来看其实是最简单的合约— Minter。 实际上，我们只需要minter方法本身，以及一个可以查看有效奖励的方法。

contract ICurveFi_Minter {
    function mint(address gauge_addr) external;
    function minted(address _for, address gauge_addr) external view returns(uint256);

    function toggle_approve_mint(address minting_user) external;
    function token() external view returns(address);
}
最后一个是YToken接口，不过它并非最不重要，因为我们要交互的就是Curve的Y池。 YToken是一个简单的ERC20接口，几乎没有其他额外的方法。

contract IYERC20 { 
    //ERC20 functions
    //
    //

    //Y-token functions
    function deposit(uint256 amount) external;
    function withdraw(uint256 shares) external;
    function getPricePerFullShare() external view returns (uint256);

    function token() external returns(address);
}
现在我们准备创建一些代码来与此协议交互。

3、基于Curve.Fi以正确的顺序转移资金
我们的实验合约的主要目标是展示如何将Curve协议作为资金流终点来组织资金的流动。 因此，我们将有两种主要方法：一种用于完整的存款流程（具有从Curve.Fi可获得的所有好处）， 另一种用于提款流程。

让我们从存款开始。

function multiStepDeposit(uint256[4] memory _amounts) public {
    address[4] memory stablecoins = ICurveFi_DepositY(curveFi_Deposit).underlying_coins();

    for (uint256 i = 0; i < stablecoins.length; i++) {
        IERC20(stablecoins[i]).safeTransferFrom(_msgSender(), address(this), _amounts[i]);
        IERC20(stablecoins[i]).safeApprove(curveFi_Deposit, _amounts[i]);
    }

    //Step 1 - deposit stablecoins and get Curve.Fi LP tokens
    ICurveFi_DepositY(curveFi_Deposit).add_liquidity(_amounts, 0); //0 to mint all Curve has to 

    //Step 2 - stake Curve LP tokens into Gauge and get CRV rewards
    uint256 curveLPBalance = IERC20(curveFi_LPToken).balanceOf(address(this));

    IERC20(curveFi_LPToken).safeApprove(curveFi_LPGauge, curveLPBalance);
    ICurveFi_Gauge(curveFi_LPGauge).deposit(curveLPBalance);

    //Step 3 - get all the rewards (and make whatever you need with them)
    crvTokenClaim();
    uint256 crvAmount = IERC20(curveFi_CRVToken).balanceOf(address(this));
    IERC20(curveFi_CRVToken).safeTransfer(_msgSender(), crvAmount);
}
正像你看见的，我们在multiStepDeposit()方法中执行了几个动作。首先， 将我们的资金（在本例中为选定的稳定币）存入Deposit合约，以便接收Curve LP代币。 第二步是将LP代币存入Gauge合约，以便获得CRV代币奖励。第三步取决于你自己 —— 可以使用CRV奖励做任何想做的事情。

实际上，相同的方案也适用于multiStepWithdraw()方法，但顺序相反。

function multiStepWithdraw(uint256[4] memory _amounts) public {
    address[4] memory stablecoins = ICurveFi_DepositY(curveFi_Deposit).underlying_coins();

    //Step 1 - Calculate amount of Curve LP-tokens to unstake
    uint256 nWithdraw;
    uint256 i;
    for (i = 0; i < stablecoins.length; i++) {
        nWithdraw = nWithdraw.add(normalize(stablecoins[i], _amounts[i]));
    }
    uint256 withdrawShares = calculateShares(nWithdraw);

    //Check if you can re-use unstaked LP tokens
    uint256 notStaked = curveLPTokenUnstaked();
    if (notStaked > 0) {
        withdrawShares = withdrawShares.sub(notStaked);
    }

    //Step 2 - Unstake Curve LP tokens from Gauge
    ICurveFi_Gauge(curveFi_LPGauge).withdraw(withdrawShares);

    //Step 3 - Withdraw stablecoins from CurveDeposit
    IERC20(curveFi_LPToken).safeApprove(curveFi_Deposit, withdrawShares);
    ICurveFi_DepositY(curveFi_Deposit).remove_liquidity_imbalance(_amounts, withdrawShares);
    
    //Step 4 - Send stablecoins to the requestor
    //
}
是的，解决方案非常简单，某种程度上看起来很笨拙，并且存在几个明显的漏洞 —— 虽然这些代码仅用于 演示目的。你可以添加所需的所有检查，将阶段拆分为不同的方法，为方法设置不同的权限，并执行 使你的DeFi项目成功所需的所有其他工作。

4、测试Curve.Fi解决方案
教程的最后一步是添加几个单元测试，以验证解决方案是否有效。虽然在这里我们面临几个问题。 第一个（也是主要的）问题是Curve.Fi协议环境。为了测试解决方案，我们需要在本地环境（在 我们的例子中为Ganache）上模拟完整的Curve协议。由于这是一组复杂的合约，而且是在Vyper上 编写的，因此我们不能仅将它们编译并部署到我们的Ganache节点中。

我已经完成了一些工作，并准备了Curve协议的测试存根，可以将其与你的项目一起编译并部署在 你的测试网上。除了出于测试目的的几种简化，这些测试存根与原始的Curve协议完全相同。你可以 在这里下载。

因此，我们可以跳过将Vyper代码转换为可测试的Solidity合约的无聊部分，直接查看单元测试的 输出：

curve.fi协议集成教程

好了！我们已经获得了集成复杂的DeFi协议的全部经验。在此基础上，尽情开启你的想象力去创造吧！

