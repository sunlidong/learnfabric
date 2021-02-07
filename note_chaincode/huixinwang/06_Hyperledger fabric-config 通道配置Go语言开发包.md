在这个教程中，我们将介绍如何使用fabric-config库进行通道配置的更新。 我们将提供一个基于fabric-config的示例程序，该程序可以更改Hyperledger Fabric的块切割参数。此外,教程还包括如何开始使用fabric-config库和函数 的指南，以便你可以在项目中快速增加通道配置功能。

用熟悉的语言和OS学习Hyperledger Fabric区块链开发：

Node.js | Java | Golang | Python | BYFN Windows版 | WIZ快速开发工具箱 | 链码Python开发包

1、通道配置概述
Hyperledger Fabric网络由一些数据结构和过程构成，它们以及定义了如何与区块链 网络进行交互。其中，数据结构包括组织、对等节点、身份凭证、排序节点和CA等。

标识数据结构及其相应过程（即用于网络交互的管理指令）的数据包含在通道配置中。 这些配置又可以在已提交给通道帐本的区块中找到。因此，用于修改通道配置的过程 称为配置更新事务。下面列举了有关通道配置更新的一些常见需求：

更新一个区块中可以包含的最大交易数量。
更新在第一个交易到达之后切割区块之前需要继续等待交易的时间。
更新Raft排序服务参数。
更新区块签名的有效性要求。
将新组织添加到已有的通道。
将新组织添加到已经建立的联盟。
2、更新通道配置
迄今为止，官方推荐的更新通道配置的方法还是使用configtxlator 和jq工具。使用这些工具更新通道配置时的步骤可以概括如下：

获取通道的最新配置区块。
将最新的配置区块从protobuf格式解码为JSON。
使用jq从JSON配置区块中删除不必要的元数据。
创建JSON配置区块的副本。
对复制的JSON配置区块进行相应的更新（例如，更新一个块允许的最大交易数）。
将JSON更新的配置区块重新编码为protobuf格式。
计算两个protobuf配置（即原始配置块和更新的配置块）之间的差异。这会生成包含增量配置的protobuf数据。
将增量数据解码回JSON。
将必要的头数据添加到JSON增量中，以便使用jq将其包装在信封消息中。
将JSON增量编码为protobuf。
签名配置更新交易。
通过对等节点将配置更新交易提交给排序服务。
尽管上述方法可行，但它非常繁琐且容易出错。例如，很容易忘记在解码最新的配置块后 剥离头数据（步骤3）或将头数据添加到增量块（步骤9）。另外，第5步很容易搞砸。在 使用文本编辑器（例如Visual Studio或Atom）或使用jq手动编辑JSON块时，可能会在 JSON文档的错误部分进行修改从而引入错误。另外，理想情况下，在对JSON块进行任何更改之前， 你应该对JSON模式有透彻的了解，但现实情况是，并不是每个人都拥有这一知识。因此， 我们希望有一种不易出错并且更加简单的更新信道配置的机制。更好的方案井盖使用类型 安全且经过编译的语言，例如Go。在下一节中，我们将介绍这种新机制。

值得一提的是，我们鼓励用户使用下面详细介绍的config更新过程来开发自己的工具， 并最终弃用configtxlator工具。这也是为什么你应该开始熟悉用于更新通道配置的 最新机制的另一个原因。

注意：提供使用configtxlator和jq工具的底层详细信息和说明超出了本文的范围。 有关此操作的完整详细信息，请参见更新通道配置。

3、使用fabric-config库编辑通道配置
3.1 引入fabric-config库
Hyperledger fabric-config库引入了一种用于生成配置交易更新的替代方法，该方法消除了 前面描述的手动JSON解析过程中所需的许多繁琐且容易出错的步骤。fabric-config库被设计为独立的库， 它支持生成诸如应用程序和系统信道创建、通道配置更新操作等，并将背书签名附加到交易信封消息。 fabric-config库是用Go编写的，并提供了丰富的API 用于修改所提供的配置交易，以及计算现有配置和所需更新之间的增量（类似于configtxlator工具的功能）。

3.2 使用fabric-config库更新通道配置
请注意，fabric-config库不包含获取配置块的功能，你可以自己决定如何获取配置块以及如何向 网络提交配置更新交易。例如，你可以选择使用Fabric Peer CLI（Hyperledger Fabric二进制文件的一部分） 从通道中获取最新的配置块。如果你正在利用IBM区块链平台，那么还可以利用Ansible的IBM区块链 平台集合从通道中获取最新的配置块。

从相应的通道中获取最新的配置块后，可以使用Go语言编写如下代码，将该配置块读入内存：

import (
...
    cb "github.com/hyperledger/fabric-protos-go/common"
...
)

func getConfigFromBlock(blockPath string) *cb.Config {
    blockBin, err := ioutil.ReadFile(blockPath)
    if err != nil {
        panic(err)
    }    
    
    block := &cb.Block {}
    err = proto.Unmarshal(blockBin, block)
    if err != nil {
        panic(err)
    }    
    
    blockDataEnvelope := &cb.Envelope {}
    err = proto.Unmarshal(block.Data.Data[0], blockDataEnvelope)
    if err != nil {
        panic(err)
    }    
    
    blockDataPayload := &cb.Payload {}
    err = proto.Unmarshal(blockDataEnvelope.Payload, blockDataPayload)
    if err != nil {
        panic(err)
    }    
    
    config := &cb.ConfigEnvelope {}
    err = proto.Unmarshal(blockDataPayload.Data, config)
    if err != nil {
        panic(err)
    }    return config.Config
}
getConfigFromBlock()函数从指定的路径读取先前获取的块，并返回指向该Config结构实例的指针。 上面的函数是完全通用的，这意味着无论你打算进行什么配置更新，都可以在代码中使用此函数 来读取配置块。注意，Config实例封装了配置块中包含的数据，格式为配置交易protobuf类型。 另外，请注意，这个Config不是fabric-config库中定义的结构。Configprotobuf是在 fabric-protos-go模块中定义的。

将配置块读入内存后，就可以创建ConfigTx实例了，如下所示：

import (
...
    "github.com/hyperledger/fabric-config/configtx"
...
)
...
    baseConfig := getConfigFromBlock(blockPath)
    configTx := configtx.New(baseConfig)
...
configtx.New()函数返回ConfigTx结构的实例，该实例在fabric-config库中定义。 ConfigTx实例是应用程序代码用于对配置块进行必要更新的主要入口点。请注意， 要获取ConfigTx实例，你需要提供使用getConfigFromBlock()方法读取的配置交易protobuf结构 作为参数。现在让我们展示如何利用ConfigTx结构来对配置块进行一些更新。 具体来说，我们将更改以下块切割参数：

absolute_max_bytes：块最大字节数，即任何区块都不应大于absolute_max_bytes。
max_message_count：区块可以包含的最大交易数，即一个区块的交易数不应超过max_message_count。
preferred_max_bytes：块的首选大小，即如果可以在preferred_max_bytes下构造一个块，则将尽早切割一个块，大于该尺寸的交易将出现在另一个块中。
batch_timeout：在第一个交易到达之后，在切割区块之前需要等待其他交易的时间。
注意：如果需要有关上述块切割参数的更多详细信息，请参阅Hyperledger Fabric官方文档中的 更新通道配置部分。

现在让我们定义一组变量来捕获上述参数：

var (
    batchSizeMaxMessage uint32
    batchSizeAbsoluteMax uint32
    batchSizePreferredMax uint32
    batchTimeout uint32
)
在代码中，你可以为这些变量分别赋值。例如，可以从属性文件中读取它们，也可以将这些值作为 运行时参数传递给程序。无论如何向应用程序提供此类值，读取后就可以将它们分配给以下变量：

batchSizeMaxMessage = ...
batchSizeAbsoluteMax = ...
batchSizePreferredMax = ...
batchTimeout = ...
完成此操作后，就可以继续使用fabric-config库中的以下API方法来更新配置块：

// Obtain OrdererGroup instance from ConfigTx instance
ordererGrp := configTx.Orderer()

// Use setter methods in the OrdererGroup instance to make configuration changes
ordererGrp.SetBatchTimeout(time.Second * time.Duration(batchTimeout))

ordererGrp.BatchSize().SetAbsoluteMaxBytes(batchSizeAbsoluteMax)

ordererGrp.BatchSize().SetMaxMessageCount(batchSizeMaxMessage)

ordererGrp.BatchSize().SetPreferredMaxBytes(batchSizePreferredMax)
对块进行配置更新后，即可计算这些更改的增量：

var (
    channelName string
)
...
configUpdateBytes, err := configTx.ComputeMarshaledUpdate(channelName)
...
在计算完增量之后，下一个可选的步骤是签名要进行的区块更新。在这样做之前， 让我们介绍下如何使用getSigningIdentity()函数来解析从本地MSP图区的身份信息：

func getSigningIdentity(sigIDPath string) *configtx.SigningIdentity {
    // Read certificate, private key and MSP ID from sigIDPath
    var (
        certificate *x509.Certificate
        privKey     crypto.PrivateKey
        mspID       string
        err         error
    )    
    
    mspUser := filepath.Base(sigIDPath)

    certificate, err = readCertificate(filepath.Join(sigIDPath, "msp", "signcerts", fmt.Sprintf("%s-cert.pem", mspUser)))
    if err != nil {
        panic(err)
    }    
    
    privKey, err = readPrivKey(filepath.Join(sigIDPath, "msp", "keystore", "priv_sk"))
    if err != nil {
        panic(err)
    }    
    
    mspID = strings.Split(mspUser, "@")[1]    return &configtx.SigningIdentity{
        Certificate: certificate,
        PrivateKey: privKey,
        MSPID: mspID,
    }
}
以下是上述功能中使用的辅助方法。首先，让我们定义readCertificate()方法：

func readCertificate(certPath string) (*x509.Certificate, error) {
    certBytes, err := ioutil.ReadFile(certPath)
    if err != nil {
        return nil, err
    }    
    
    pemBlock, _ := pem.Decode(certBytes)
    if pemBlock == nil {
        return nil, fmt.Errorf("no PEM data found in cert[% x]", certBytes)
    }    
    
    return x509.ParseCertificate(pemBlock.Bytes)
}
然后定义readPrivKey()方法：

func readPrivKey(keyPath string) (crypto.PrivateKey, error) {
    privKeyBytes, err := ioutil.ReadFile(keyPath)
    if err != nil {
        return nil, err
    }
    
    pemBlock, _ := pem.Decode(privKeyBytes)
    if pemBlock == nil {
        return nil, fmt.Errorf("no PEM data found in private key[% x]", privKeyBytes)
    }    
    
    return x509.ParsePKCS8PrivateKey(pemBlock.Bytes)
}
就像getConfigFromBlock()方法一样，getSigningIdentity()也可用于任何类型的方案。 因此，你可以将这个getSigningIdentity()功能添加到你的应用程序代码中并加以利用，而无需 进行任何配置更新。getSigningIdentity()方法的唯一参数是指向MSP根文件夹的路径， 该文件夹应具有一组子文件夹，其中包含MSP组织用户或管理员的相应证书和密钥。MSP根文件夹的 名称应遵循以下命名约定：<enrollment_id>@。例如，由OrdererMSP标识的Orderer组织的 Admin用户的身份材料应位于名为Admin@OrdererMSP的文件夹下。在子文件夹下找到的证书和密钥的 名称应如下所示：

<enrollment_id>@<MSP ID>  // sigIDPath
└── msp   
    ├── admincerts
    │   │   // The public cert for the org administrator
    │   └── admin-cert.pem    
    ├── cacerts
    │   │   // The public cert for the root CA
    │   └── ca-cert.pem
    ├── tlscacerts
    │   │   // The public cert for the root TLS CA
    │   └── tlsca-cert.pem 
    ├── keystore
    │   │   // The private key for the identity
    │   └── priv_sk     
    └── signcerts
        │   // The public cert for the identity
        └── <enrollment_id>@<MSP ID>-cert.pem
注意：如果你使用过cryptogen工具， 那么上面显示的文件夹结构应该看起来很熟悉。

请注意，getSigningIdentity()方法返回指向configtx.SigningIdentity结构实例的指针， 该实例也在fabric-config库中定义。

对于每个应该签名通道配置更新的身份，都应该调用getSigningIdentity()方法。 可以将调用此方法返回的身份标识存储在数组中。一旦拥有用于签名配置更新的所有必需身份， 就可以使用configtx.SigningIdentity结构的CreateConfigSignature()方法来创建相应的签名：

configSignatures := []*cb.ConfigSignature{}
...
signingIdentity := getSigningIdentity(pathToSigningIdentity)
...
configSignature, err := signingIdentity.CreateConfigSignature(configUpdateBytes)
...
configSignatures = append(configSignatures, configSignature)
CreateConfigSignature方法将我们之前通过调用ComputeMarshaledUpdate()函数计算出的 增量作为参数，即configUpdateBytes。

生成必要的签名后，可以使用以下方法创建信封消息，其中包含配置更新以及签名：

env, err := configtx.NewEnvelope(configUpdateBytes, configSignatures...)
就像CreateConfigSignature, configUpdateBytes从ComputeMarshaledUpdate()函数调用中返回的 配置增量一样，configSignatures数组则包含所有必要的签名（即，指向ConfigSignature结构实例的指针）。 对于我们在本文中讨论的示例情况（即，切割参数的更改），只需要订购服务组织的管理员的签名。

你可能还希望使用将交易提交到排序节点的身份对从NewEvelope()函数返回的信封消息进行签名。 在我们的示例中，此身份也是排序服务机构的管理员。你还可以通过调用getSigningIdentity() 方法来获得此标识实例，正如我们已经提到的，该方法返回该configtx.SigningIdentity结构的实例：

envelopeSigningIdentity := getSigningIdentity(pathToEnvelopeSigningIdentity)

err = envelopeSigningIdentity.SignEnvelope(env)
最后，我们将签名的信封写入文件系统：

envelopeBytes, err := proto.Marshal(env)
if err != nil {
    panic(err)
}
err = ioutil.WriteFile(outputPath, envelopeBytes, 0640)
现在，你有了一个配置更新交易，其中包含更改和[签名]，可以将 其提交给网络进行处理！作为参考，下面是示例程序的完整源代码：

package main

import (
	"crypto"
	"crypto/x509"
	"encoding/pem"
	"fmt"
	"io/ioutil"
	"path/filepath"
	"strings"
	"time"

	"github.com/golang/protobuf/proto"
	"github.com/hyperledger/fabric-config/configtx"
	cb "github.com/hyperledger/fabric-protos-go/common"
)

func main() {

	var (
		batchSizeMaxMessage           uint32
		batchSizeAbsoluteMax          uint32
		batchSizePreferredMax         uint32
		batchTimeout                  uint32
		blockPath                     string
		channelName                   string
		pathToSigningIdentity         string
		pathToEnvelopeSigningIdentity string
		outputPath                    string
	)

	// Update variables as needed for your use case
	batchSizeMaxMessage = 10
	batchSizeAbsoluteMax = 103809024
	batchSizePreferredMax = 524288
	batchTimeout = 4
	blockPath = "<blockPath>"
	channelName = "<channelName>"
	pathToSigningIdentity = "<pathToSigningIdentity>"
	pathToEnvelopeSigningIdentity = "<pathToEnvelopeSigningIdentity>"
	outputPath = "<outputPath>"

	// Read configuration block into memory
	baseConfig := getConfigFromBlock(blockPath)
	configTx := configtx.New(baseConfig)

	// Obtain OrdererGroup instance from ConfigTx instance
	ordererGrp := configTx.Orderer()
	// Use setter methods in the OrdererGroup instance to make configuration changes
	ordererGrp.SetBatchTimeout(time.Second * time.Duration(batchTimeout))
	ordererGrp.BatchSize().SetAbsoluteMaxBytes(batchSizeAbsoluteMax)
	ordererGrp.BatchSize().SetMaxMessageCount(batchSizeMaxMessage)
	ordererGrp.BatchSize().SetPreferredMaxBytes(batchSizePreferredMax)

	// Compute delta
	configUpdateBytes, err := configTx.ComputeMarshaledUpdate(channelName)
	if err != nil {
		panic(err)
	}

	// Attach signature
	signingIdentity := getSigningIdentity(pathToSigningIdentity)
	configSignature, err := signingIdentity.CreateConfigSignature(configUpdateBytes)
	if err != nil {
		panic(err)
	}

	// Create envelope
	env, err := configtx.NewEnvelope(configUpdateBytes, configSignature)

	// Sign envelope
	envelopeSigningIdentity := getSigningIdentity(pathToEnvelopeSigningIdentity)
	err = envelopeSigningIdentity.SignEnvelope(env)
	envelopeBytes, err := proto.Marshal(env)
	if err != nil {
		panic(err)
	}

	// Write envelope to file system
	err = ioutil.WriteFile(outputPath, envelopeBytes, 0640)
}

func getConfigFromBlock(blockPath string) *cb.Config {
	blockBin, err := ioutil.ReadFile(blockPath)
	if err != nil {
		panic(err)
	}

	block := &cb.Block{}
	err = proto.Unmarshal(blockBin, block)
	if err != nil {
		panic(err)
	}

	blockDataEnvelope := &cb.Envelope{}
	err = proto.Unmarshal(block.Data.Data[0], blockDataEnvelope)
	if err != nil {
		panic(err)
	}

	blockDataPayload := &cb.Payload{}
	err = proto.Unmarshal(blockDataEnvelope.Payload, blockDataPayload)
	if err != nil {
		panic(err)
	}

	config := &cb.ConfigEnvelope{}
	err = proto.Unmarshal(blockDataPayload.Data, config)
	if err != nil {
		panic(err)
	}

	return config.Config
}

func getSigningIdentity(sigIDPath string) *configtx.SigningIdentity {
	// Read certificate, private key and MSP ID from sigIDPath
	var (
		certificate *x509.Certificate
		privKey     crypto.PrivateKey
		mspID       string
		err         error
	)
	mspUser := filepath.Base(sigIDPath)

	certificate, err = readCertificate(filepath.Join(sigIDPath, "msp", "signcerts", fmt.Sprintf("%s-cert.pem", mspUser)))
	if err != nil {
		panic(err)
	}
	privKey, err = readPrivKey(filepath.Join(sigIDPath, "msp", "keystore", "priv_sk"))
	if err != nil {
		panic(err)
	}
	mspID = strings.Split(mspUser, "@")[1]
	return &configtx.SigningIdentity{
		Certificate: certificate,
		PrivateKey:  privKey,
		MSPID:       mspID,
	}
}

func readCertificate(certPath string) (*x509.Certificate, error) {
	certBytes, err := ioutil.ReadFile(certPath)
	if err != nil {
		return nil, err
	}
	pemBlock, _ := pem.Decode(certBytes)
	if pemBlock == nil {
		return nil, fmt.Errorf("no PEM data found in cert[% x]", certBytes)
	}
	return x509.ParseCertificate(pemBlock.Bytes)
}

func readPrivKey(keyPath string) (crypto.PrivateKey, error) {
	privKeyBytes, err := ioutil.ReadFile(keyPath)
	if err != nil {
		return nil, err
	}
	pemBlock, _ := pem.Decode(privKeyBytes)
	if pemBlock == nil {
		return nil, fmt.Errorf("no PEM data found in private key[% x]", privKeyBytes)
	}
	return x509.ParsePKCS8PrivateKey(pemBlock.Bytes)
}
4、注意事项
尽管fabric-config库极大地改进了更新通道配置的过程，但是仍然需要考虑一些注意事项。

尽管您可以使用这个库来更新任何版本的Hyperledger Fabric的通道配置，但由于 不支持某些被弃用的配置项，因此强烈建议你将其用于更新Hyperledger Fabric v2通道或 迁移到Hyperledger Fabric v2。

由于fabric-config库是用Go语言编写的，因此该库只能由使用Go语言编写工具的开发者使用。 可以通过创建通用工具（例如命令行界面）来避免这种情况，这些通用工具不直接嵌入目标 应用程序中。这种方法将允许在外部使用该库来生成配置更新交易。或者，可以设置用Go语言编写 的服务器，该服务器用于可从输入配置块生成配置交易更新的端点。

Hyperledger Fabric的现有用户可能已经有稳定的方法来更新配置交易。因此，他们可能不愿意 修改现有的自动化过程。但是，随着在通道配置周围添加新功能，最终将难以维护当前的通道配置 方法。无需手动更新零散的解决方法代码，使用此库将成为通过简单扩展来采用新功能的一致方法。

5、结论
如本文所示，Hyperledger fabric-config库提供了一种可靠的机制来生成配置更新交易， 同时消除了手动进行此类更改时出现的机械步骤和易于出错的步骤。因此，fabric-config库 使你能够以可靠且一致的方式自动执行配置更新交易。

尽管未在本教程中显示，fabric-config库还支持生成用于应用程序和系统通道创建的配置 包络以及修改应用程序和通道功能。用Go语言编写的fabric-config库为此类操作提供了类型 安全且经过编译的选项。

我们鼓励你查看fabric-config库的官方GoDoc 文档，以便熟悉其直观且易于使用的API， 并查看其他示例和代码段。利用本文中共享的指导和功能，你可以立即在下一个项目中利用fabric-config库！