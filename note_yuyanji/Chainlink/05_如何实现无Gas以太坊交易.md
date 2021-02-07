每个人都在讨论无gas以太坊交易，因为没有人喜欢支付gas费用。 但是以太坊网络能够精准地运转恰恰是因为交易需要手续费。那么 如何实现无gas交易呢？让我们一起学习无gas以太坊交易的魔法！

用自己熟悉的语言学习 以太坊DApp开发 ： Java | Php | Python | .Net / C# | Golang | Node.JS | Flutter / Dart

在这篇文章中，我们将学习如何实现无gas交易模式。你会发现虽然 在以太坊上没有免费的午餐，但可以用有趣的方式来转移gas成本。 利用本文中学到的知识，你的DApp用户就可以省掉gas，获得更好 的用户体验，或者在你的智能合约中构建新颖的代理模式。

不过等一下！还不止这些！为了方便你的使用，我已经将相关工具 放到这个Github仓库了。 因此现在你要实现无gas以太坊交易的门槛已经大大降低了。

现在让我们开始吧！

1、一些背景知识
我不得不承认，虽然我了解如何在智能合约中实现无gas交易，但是 并不太了解背后的密码学知识。不过对我而言这算不上大的障碍，因此 如果你也不太熟悉密码学，相信也不会影响你实现无gas以太坊交易。

据我所知，我的私钥被用来签名发送到以太坊网络的交易，在这个过程 中运用了一些密码学技术来识别我的身份并存入变量msg.sender，这是 以太坊中访问控制的基石。

无gas交易背后的魔法在于，我们可以用自己的私钥为希望执行的合约交易 制作一个签名。

签名是链下生成的，无需消耗任何gas。一旦签名完成，就可以将交易 发送给其他人替我们执行，同时也替我们支付gas费用。

使用签名的合约函数通常就是一个普通的函数，不过支持传入额外的 签名参数。例如在dai.sol中，我们可以看到如下的approve函数：

function approve(address usr, uint wad) external returns (bool)
同时也可以看到permit函数，它和approve做的事情一样，只是支持 额外的签名参数：

function permit(address holder, address spender, uint256 nonce, uint256 expiry, bool allowed, uint8 v, bytes32 r, bytes32 s) external
不用担心看不懂这些额外的参数，下面会讲解。我们需要注意的是，上面 这两个函数是如何处理allowance映射的：

function approve(address usr, uint wad) external returns (bool)
{
  allowance[msg.sender][usr] = wad;
  …
}

function permit(
  address holder, address spender,
  uint256 nonce, uint256 expiry, bool allowed,
  uint8 v, bytes32 r, bytes32 s
) external {
  …
  allowance[holder][spender] = wad;
  …
}
如果你调用approve方法，那么就意味着允许spender账号操作不超过wad个你持有的代币。
如果你把一个有效签名给了其他人，那么那个人就可以通过调用permit方法 来允许spender账号操作不超过wad个你持有的代币。
是不是一样？

因此基本上来说，无gas交易背后的模式就是制作一个签名，别人用这个 签名就可以用你的身份安全地执行一个特殊的交易，就像你授权别人执行 一个方法。

这其实就是一种代理模式。

2、无gas交易规范
如果你和我一样，那你可能马上就会深入研究代码。我立刻注意到了 一个注释：

// — — EIP712 niceties — -
看起来是一个以太坊规范，因此我就研究了一下， 不过当时并没有理解。现在我已经理解，并且可以用浅显的话语来解释了。

EIP712描述了为 合约方法生成签名的通用方式。其他的EIP则描述如何在特定的用例中运用EIP712。 例如EIP2612描述 如何将EIP712签名用于permit方法，该方法和ERC20代币中的approve方法实现相同的功能， 就像我们在前面看到的。

如果你只是想实现一个已经定义过的签名方法，比如为你的MetaCoin合约添加 支持签名的approve方法，那么阅读EIP2612就够了。更简单的办法就是直接 继承一个已经实现了EIP2612的合约。

在这篇文章中，我们将研究dai.sol中的一种无gas交易实现，这会帮助我们 更清晰地理解其内部机制。dai.sol的无gas实现是在EIP2612之前完成的，因此 有一些区别。不过这不是大问题。

3、签名构成
在dai.sol中可以看到EIP712的一个早期实现，它允许dai持有者在链下计算签名 并交由spender代为执行approve方法，而不是由dai持有者直接调用approve方法。

整个实现包含4个部分：

DOMAIN_SEPARATOR
PERMIT_TYPEHASH
nonces变量
permit函数
下面是DOMAIN_SEPARATOR以及相关的变量：

string  public constant name     = "Dai Stablecoin";
string  public constant version  = "1";
bytes32 public DOMAIN_SEPARATOR;constructor(uint256 chainId_) public {
  ...
  DOMAIN_SEPARATOR = keccak256(abi.encode(
    keccak256(
      "EIP712Domain(string name,string version," + 
      "uint256 chainId,address verifyingContract)"
    ),
    keccak256(bytes(name)),
    keccak256(bytes(version)),
    chainId_,
    address(this)
  ));
}
DOMAIN_SEPARATOR就是一个用来唯一标识智能合约的哈希，它是利用 一个标记EIP712域（合约名称、版本、链ID、部署地址）的字符串构造的。

所有这些信息在构造函数中进行哈希并存入DOMAIN_SEPARATOR变量，dai 持有者在生成签名时需要使用这个变量值，并且在执行permit方法时需要 匹配。DOMAIN_SEPARATOR可以确保一个签名仅对单一合约有效。

下图是PERMIT_TYPEHASH：

gas-less ethereum transactions

PERMIT_TYPEHASH是函数名（首字母大写）以及全部参数（包括类型和参数名） 的哈希，其目的是清晰界定签名的适用方法。

在permit方法中需要处理签名，如果适用的PERMIT_TYPEHASH并不是针对 这个方法的，交易就会回滚。这样就确保了一个签名仅可以用于特定的方法。

下面是nonces映射：

mapping (address => uint) public nonces;
nonces应用用来注册一个特定的dai持有者已经使用的签名数量。当创建 签名时，需要包含一个nonces值，当执行permit方法时，nonce必须匹配 该持有者已经使用的签名数量。这一措施用来确保签名仅使用一次。

这三者结合在一起，PERMIT_TYPEHASH、DOMAIN_SEPARATOR以及nonce，就 可以确保一个签名仅可以用于特定的合约、特定的方法，并且只可以使用 一次。

现在让我们看看在智能合约中是如何处理签名的。

4、permit方法
permit方法是dai.sol中实现的一个函数，它允许使用签名来实现approve 相同的功能。

// --- Approve by signature ---
function permit(
  address holder, address spender,
  uint256 nonce, uint256 expiry, bool allowed,
  uint8 v, bytes32 r, bytes32 s
) external
正如你看到的，permit方法包含很多参数。这些参数是计算签名需要的 数据，以及签名数据v、r和s。

传入创建签名的参数看起来很傻，但是这是必须的。因为从签名中能够 恢复出来的只有签名创建者的地址。我们需要所有这些参数以及恢复出来 的创建者地址来确保签名的有效性。

首先我们利用这些参数计算一个摘要数据。dai持有者需要在链下进行同样 的计算，这是生成签名的必要环节：

bytes32 digest =
  keccak256(abi.encodePacked(
    "\x19\x01",
    DOMAIN_SEPARATOR,
    keccak256(abi.encode(
      PERMIT_TYPEHASH,
      holder,
      spender,
      nonce,
      expiry,
      allowed
    ))
  ));
使用ecrecover方法以及v、r和s，我们可以从签名中恢复出地址。 如果这就是dai持有者的地址，那么我们就知道参数对上了，也就是 说DOMAIN_SEPARATOR、PERMIT_TYPEHASH、nonce、holder、spender、 expiry以及allowed都对。如何对不上，就拒绝这个签名：

require(holder == ecrecover(digest, v, r, s), "Dai/invalid-permit");
这个地方需要注意。签名涉及很多参数，其中有些参数比较晦涩，例如 链ID（DOMAIN_SEPARATOR的一部分）。其中任何参数对不上都会导致签名 被拒绝，这使得链下签名的调试非常困难。

现在我们指导持有者已经授权了这个方法调用。接下来我们需要确认 签名没有被滥用。

首先检查当前时间是否在expiry之前，这样可以让授权仅在特定时间点 之前有效。

require(expiry == 0 || now <= expiry, "Dai/permit-expired");
我们也可以检查具有这个nonce的签名还没有使用过，这样就可以确保 一个签名只能使用一次。

require(nonce == nonces[holder]++, "Dai/invalid-nonce");
现在通过了！dai.sol更新allowance，触发事件，就这些简单的工作了。

uint wad = allowed ? uint(-1) : 0;
allowance[holder][spender] = wad;
emit Approval(holder, spender, wad);
dai.sol合约使用二进制方式处理allowance，在我们提供的代码 中则使用了更传统的方式来处理allowance。

5、创建链下签名
创建签名不适合胆小的人，不过只需要一点练习和耐心，其实也容易掌握。 我们用三个步骤来复制智能合约的permit方法中的逻辑：

生成DOMAIN_SEPARATOR
生成摘要
生成交易签名
下面的函数将创建DOMAIN_SEPARATOR。它和dai.sol构造函数中的代码 功能一样，不过使用的是javascript，以及ethers.js中的keccak256、 defaultAbiCoder和toUtfBytes。这个函数需要代币名称、部署地址以及 链ID，并假设代币版本为”1”：

gas-less ethereum transaction

下面的函数将为特定的permit调用创建摘要。注意holder、spender、nonce 和expiry都作为参数传入。同时传入一个approve.allowed参数，虽然你可以 始终将其设置为true。注意这里的PERMIT_TYPEHASH我们是直接从dai.sol拷贝 过来的。

gas-less ethereum transaction

一旦我们得到摘要，那么进行签名就相对容易多了。我们使用ethereumjs-util中 的ecsign对移除0x前缀的摘要数据进行签名。注意这个步骤我们需要用户私钥。

gas-less ethereum transaction

上述js函数的调用方法如下：

gas-less ethereum transaction

注意我们在调用permit时是如何使用之前创建摘要的那些参数的。只有这样签名才会 有效。

另一点需要注意的是，在这个代码片段中user2只调用两个交易。user1 表示dai持有者，他是创建摘要并进行签名的账号。然而user1并不需要 消耗任何gas。

user1将签名给user2，user2使用这个签名来执行permit方法以及 transferFrom方法。

在user1看来，这就是一个无gas交易，他不需要消耗任何wei。

6、结论
本文展示了如何使用无gas交易，澄清了无gas实际上意味着将gas成本 转嫁给了其他人。为此我们需要智能合约中的方法能够处理预签名交易。

不过使用这一模式有显著的好处，因此无gas交易已经被广泛使用。 签名允许交易的gas成本从用户转移到服务提供商，从而消除了很多 场景中的用户进入障碍。无gas交易也支持更加高级的代码模式实现， 通常都会带来显著的用户体验改善。