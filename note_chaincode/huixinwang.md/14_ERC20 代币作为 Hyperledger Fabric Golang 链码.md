与以太坊区块链一样，Hyperledger Fabric平台（HLF）可用于创建代币，实现为智能合约（HLF术语中的链码），用于保存用户余额。与以太坊不同，HLF链码不能将用户地址用作持有者密钥，因此我们将使用成员资格服务提供程序（MSP）标识符和用户证书标识符的组合。下面是一个简单的示例，说明如何使用CCKit链码库在Hyperledger Fabric平台上创建代币作为Golang链码。

什么是ERC20 Token标准
ERC20 Token标准是为了在以太坊中标准化通证智能合约，它描述了以太坊Token合约必须实现的功能和事件。以太坊区块链上的大多数主要代币都符合ERC20标准。ERC-20有许多好处，包括统一Token钱包和交换列出更多Token的能力，只提供Token合约的地址。

// ----------------------------------------------------------------------------
// ERC Token Standard #20 Interface
// https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md
// ----------------------------------------------------------------------------
contract ERC20Interface {
    function totalSupply() public constant returns (uint);
    function balanceOf(address tokenOwner) public constant returns (uint balance);
    function allowance(address tokenOwner, address spender) public constant returns (uint remaining);
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);

    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
ERC20实施基础知识
从本质上讲，以太坊Token合约是一份智能合约，其中包含帐户地址及其余额的地址。余额是由合约创建者定义的值，它可以是可替代的物理对象，另一种货币价值。此余额的单位通常称为Token。

ERC20功能：

balanceOf：返回所有者标识符的标记余额（以太坊的帐户地址）。
transfer：将金额转移到我们选择的所有者标识符。
approve：设置一个Token数量，允许指定的所有者标识符代表我们支出。
allowance：检查允许所有者标识符代表我们花多少钱。
transferFrom：如果我们被该所有者标识符允许花费一些Token，则指定要转移的所有者标识符。
Hyperledger Fabric的所有者标识符
在Hyperledger Fabric网络中，所有参与者都具有其他参与者已知的身份。默认的成员资格服务提供程序Membership Service Provider实现使用X.509证书作为身份，采用传统的公钥基础结构（PKI）分层模型。

使用有关提案和资产所有权的创建者的信息，链码应该能够实现链代码级访问控制机制，检查参与者是否可以启动更新资产的交易。相应的链码逻辑必须能够存储与资产相关联的“所有权”信息，并根据提议对其进行评估。

作为HLF网络中的唯一所有者标识符（Token余额持有者），我们可以使用MSP标识符和用户身份标识符的组合。身份标识符，是X.509证书的Subject和Issuer部分的串联。此ID在MSP中保证是唯一的。

func (c *clientIdentityImpl) GetID() (string, error) {
	// The leading "x509::" distinquishes this as an X509 certificate, and
	// the subject and issuer DNs uniquely identify the X509 certificate.
	// The resulting ID will remain the same if the certificate is renewed.
	id := fmt.Sprintf("x509::%s::%s", getDN(&c.cert.Subject), getDN(&c.cert.Issuer))
	return base64.StdEncoding.EncodeToString([]byte(id)), nil
}
客户端身份链代码库Client identity chaincode library允许编写链代码，该代码根据客户端的身份（即链代码的调用者）做出访问控制决策。

特别是，你可以根据与客户端关联的以下任一或两者来做出访问控制决策：

客户身份的MSP（成员资格服务提供程序）ID。
与客户端标识关联的属性。
CCkit包含具有结构和功能的标识包，可用于实现链代码中的访问控制。

开始使用该示例
在我们的示例中，我们使用CCKit Router来管理智能合约功能。在开始之前，请务必获取CCkit：

git clone git@github.com:s7techlab/cckit.git
并使用dep命令获取依赖项：

dep ensure -vendor-only
ERC20示例位于examples/erc20目录中。

定义令牌智能合约功能
首先，我们需要定义链代码函数。在我们的示例中，我们使用来自CCKit的Router包，它允许我们以一致的方式定义链代码方法及其参数。有关链代码方法路由，中间件，链代码调用上下文的详细信息在前一篇文章中有所描述。

首先，我们使用参数symbol，name和totalSupply定义init函数（智能合约构造函数）。之后我们定义链码方法，实现ERC20接口，采用HLF所有者标识符（MSP Id和证书ID对）。从链代码状态查询的方法以query为前缀，写入链代码状态的方法以invoke为前缀。

因此，我们使用默认的链代码结构，它将Init和Invoke处理委托给router。

func NewErc20FixedSupply() *router.Chaincode {
	
	r := router.New(`erc20fixedSupply`).Use(p.StrictKnown).
        // Chaincode init function, initiates token smart contract with token symbol, name and totalSupply
        Init(invokeInitFixedSupply, p.String(`symbol`), p.String(`name`), p.Int(`totalSupply`)).
        // Get token symbol
        Query(`symbol`, querySymbol).
        // Get token name
        Query(`name`, queryName).
        // Get the total token supply
        Query(`totalSupply`, queryTotalSupply).
        //  get account balance
        Query(`balanceOf`, queryBalanceOf, p.String(`mspId`), p.String(`certId`)).
        //Send value amount of tokens
        Invoke(`transfer`, invokeTransfer, p.String(`toMspId`), p.String(`toCertId`), p.Int(`amount`)).
        // Allow spender to withdraw from your account, multiple times, up to the _value amount.
        // If this function is called again it overwrites the current allowance with _valu
        Invoke(`approve`, invokeApprove, p.String(`spenderMspId`), p.String(`spenderCertId`), p.Int(`amount`)).
        //    Returns the amount which _spender is still allowed to withdraw from _owner]
        Query(`allowance`, queryAllowance, p.String(`ownerMspId`), p.String(`ownerCertId`),
            p.String(`spenderMspId`), p.String(`spenderCertId`)).
        // Send amount of tokens from owner account to another
        Invoke(`transferFrom`, invokeTransferFrom, p.String(`fromMspId`), p.String(`fromCertId`),
            p.String(`toMspId`), p.String(`toCertId`), p.Int(`amount`))

	return router.NewChaincode(r)
}
Chaincode初始化（构造函数）
Chaincode init函数（标记构造函数）执行以下操作：

使用来自CCKit的所有者扩展来链接关于链代码所有者的链接代码状态信息。
放入链码状态token配置：代币符号，名称和总供应量。
设置chaincode所有者余额与总供应量。
const SymbolKey = `symbol`
const NameKey = `name`
const TotalSupplyKey = `totalSupply`


func invokeInitFixedSupply(c router.Context) (interface{}, error) {
	ownerIdentity, err := owner.SetFromCreator(c)
	if err != nil {
		return nil, errors.Wrap(err, `set chaincode owner`)
	}

	// save token configuration in state
	if err := c.State().Insert(SymbolKey, c.ArgString(`symbol`)); err != nil {
		return nil, err
	}

	if err := c.State().Insert(NameKey, c.ArgString(`name`)); err != nil {
		return nil, err
	}

	if err := c.State().Insert(TotalSupplyKey, c.ArgInt(`totalSupply`)); err != nil {
		return nil, err
	}

	// set token owner initial balance
	if err := setBalance(c, ownerIdentity.GetMSPID(), ownerIdentity.GetID(), c.ArgInt(`totalSupply`)); err != nil {
		return nil, errors.Wrap(err, `set owner initial balance`)
	}

	return ownerIdentity, nil
}
定义事件结构类型
我们使用identity身份包中的Id结构并定义Transfer和Approve事件的结构：

type (
  Id struct {
    MSP  string
    Cert string
  }

 Transfer struct {
    From   identity.Id
    To     identity.Id
    Amount int
 }

 Approve struct {
    From    identity.Id
    Spender identity.Id
    Amount  int
 }
)
实现token智能合约功能
查询函数非常简单，它只是从链码状态读取值：

const SymbolKey = `symbol`

func querySymbol(c r.Context) (interface{}, error) {
	return c.State().Get(SymbolKey)
}
一些改变状态函数更复杂。例如，在函数invokeTransfer中，我们执行：

接收函数调用者证书（通过tx GetCreator()函数）。
检查转移目的地。
获得当前的调用者（付款人）余额。
检查余额以转移代币金额amount。
获得收件人余额。
以链码状态更新付款人和收款人余额。
func invokeTransfer(c r.Context) (interface{}, error) {
	// transfer target
	toMspId := c.ArgString(`toMspId`)
	toCertId := c.ArgString(`toCertId`)

	//transfer amount
	amount := c.ArgInt(`amount`)

	// get informartion about tx creator
	invoker, err := identity.FromStub(c.Stub())
	if err != nil {
		return nil, err
	}

	// Disallow to transfer token to same account
	if invoker.GetMSPID() == toMspId && invoker.GetID() == toCertId {
		return nil, ErrForbiddenToTransferToSameAccount
	}

	// get information about invoker balance from state
	invokerBalance, err := getBalance(c, invoker.GetMSPID(), invoker.GetID())
	if err != nil {
		return nil, err
	}

	// Check the funds sufficiency
	if invokerBalance-amount < 0 {
		return nil, ErrNotEnoughFunds
	}

	// Get information about recipient balance from state
	recipientBalance, err := getBalance(c, toMspId, toCertId)
	if err != nil {
		return nil, err
	}

	// Update payer and recipient balance
	setBalance(c, invoker.GetMSPID(), invoker.GetID(), invokerBalance-amount)
	setBalance(c, toMspId, toCertId, recipientBalance+amount)

	// Trigger event with name "transfer" and payload - serialized to json Transfer structure
	c.SetEvent(`transfer`, &Transfer{
		From: identity.Id{
			MSP:  invoker.GetMSPID(),
			Cert: invoker.GetID(), 
		},
		To: identity.Id{
			MSP:  toMspId,
			Cert: toCertId,
		},
		Amount: amount,
	})

	// return current invoker balance
	return invokerBalance - amount, nil
}

// setBalance puts balance value to state
func setBalance(c r.Context, mspId, certId string, balance int) error {
	return c.State().Put(balanceKey(mspId, certId), balance)
}

// balanceKey creates composite key for store balance value in state
func balanceKey(ownerMspId, ownerCertId string) []string {
	return []string{BalancePrefix, ownerMspId, ownerCertId}
}
测试
此外，我们可以通过CCKit MockStub快速测试我们的链代码。

要开始测试，我们通过MockStub使用测试参数初始化链码：

var _ = Describe(`ERC-20`, func() {

	const TokenSymbol = `HLF`
	const TokenName = `HLFCoin`
	const TotalSupply = 10000
	const Decimals = 3

	//Create chaincode mock
	erc20fs := testcc.NewMockStub(`erc20`, NewErc20FixedSupply())

	// load actor certificates
	actors, err := identity.ActorsFromPemFile(`SOME_MSP`, map[string]string{
		`token_owner`:     `s7techlab.pem`,
		`account_holder1`: `victor-nosov.pem`,
		//`accoubt_holder2`: `victor-nosov.pem`
	}, examplecert.Content)
	if err != nil {
		panic(err)
	}

	BeforeSuite(func() {
		// init token haincode
		expectcc.ResponseOk(erc20fs.From(actors[`token_owner`]).Init(TokenSymbol, TokenName, TotalSupply, Decimals))
	})
我们可以检查所有代币操作：

Describe("ERC-20 transfer", func() {
    It("Disallow to transfer token to same account", func() {
        expectcc.ResponseError(
            erc20fs.From(actors[`token_owner`]).Invoke(
                `transfer`, actors[`token_owner`].GetMSPID(), actors[`token_owner`].GetID(), 100),
            ErrForbiddenToTransferToSameAccount)
    })

    It("Disallow token holder with zero balance to transfer tokens", func() {
        expectcc.ResponseError(
            erc20fs.From(actors[`account_holder1`]).Invoke(
                `transfer`, actors[`token_owner`].GetMSPID(), actors[`token_owner`].GetID(), 100),
            ErrNotEnoughFunds)
    })

    It("Allow token holder with non zero balance to transfer tokens", func() {
        expectcc.PayloadInt(
            erc20fs.From(actors[`token_owner`]).Invoke(
                `transfer`, actors[`account_holder1`].GetMSPID(), actors[`account_holder1`].GetID(), 100),
            TotalSupply-100)

        expectcc.PayloadInt(
            erc20fs.Query(
                `balanceOf`, actors[`token_owner`].GetMSPID(), actors[`token_owner`].GetID()), TotalSupply-100)

        expectcc.PayloadInt(
            erc20fs.Query(
                `balanceOf`, actors[`account_holder1`].GetMSPID(), actors[`account_holder1`].GetID()), 100)
    })
})