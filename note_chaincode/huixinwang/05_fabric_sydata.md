在Hyperledger Fabric中有两个相关的概念：私有数据（Private Data） 和暂态数据（Transient Data）。本文提供四个示例程序，分别对应 私有数据和暂态数据的四种组合使用方式，并通过观察账本的交易以及 世界状态数据库，理解为什么在使用私有数据时应当采用暂态数据作为输入。

从技术上讲，私有数据和暂态数据是两个不同的概念。私有数据 是考虑如何在通道的部分机构之间共享数据，而暂态数据则是使用私有数据 时的一种输入方法。有趣的是，这两者并没有直接的关系，虽然在现实中， 当我们需要安全的使用私有数据时，通常都应当使用暂态数据作为输入。

Hyperledger Fabric区块链开发教程： Fabric Node.js开发详解 | Fabric Java开发详解 | Fabric Golang开发详解

1、基本概念
先让我们重温一下在演示程序中将要用到的一些核心概念。

账本：在Hyperledger Fabric中，当一个peer节点加入通道后，就会维护一个 账本的副本。账本包含一个用于保存区块的区块链数据结构，以及一个用于 保存最新状态的世界状态数据库。当peer节点从排序服务收到一个新的区块 并且验证成功后，peer节点就将区块提交进账本，并根据区块中每个交易的 RWSet来更新相应的世界状态。

基于共识机制，在一个通道中，不同的peer节点上的账本的大部分都是一致的。 不过有一个例外，Fabric支持在部分通道之间存在的私有数据。

私有数据：在一个通道内有时会需要仅在部分机构之间共享数据。Hyperledger Fabric引入私有数据的目的就是满足这一需求。通过定义数据集，我们可以声明 私有数据实现的机构子集，同时所有节点（包括在机构子集之外的其他节点） 都会保存私有数据哈希的记录以便作为数据存在的证据或者用于审计目的。

可以利用链码API来使用私有数据。我们在示例代码中使用PutPrivateData和 GetPrivateData这两个API。作为对比，我们使用PutState和GetState完成对 公共状态的读写。

在Fabric 2.0中，会为每个机构准备一个隐含的数据集。本教程将利用这个 隐含的数据集，因此我们不需要单独的数据集定义文件。

Fabric的账本结构以及私有数据的位置如下图所示，在后面的演示代码中， 我们将查看交易以及世界状态数据库的内容：

fabric私有数据与暂态数据

暂态数据：许多链码函数在被调用时需要额外的输入数据。在大多数情况下我们会在 调用函数时传入一组参数，而链码参数，包括函数名和函数参数，都会作为有效 交易的一部分保存在区块内，因此将永久性的存在于账本中。如果出于某种原因 我们不希望在链上永久保存参数列表，我们就可以使用暂态数据。暂态数据是一种 可以向链码函数传参但不需要将其保存在交易记录中的输入方法。当使用暂态数据 时，需要一个特殊的链码API即GetRansient方法来读取暂态数据。我们接下来将在 演示代码中看到。

因此，链码的设计需要根据业务需求设计链码，决定哪些数据应当作为正常参数 输入并记录在交易中，哪些数据应当作为暂态数据输入而不必记录在链上。

私有数据与暂态数据的关系：私有数据与暂态数据并不是直接相关的，我们可以 只使用私有数据而不利用暂态数据作为输入，也可以在非私有数据中使用暂态数据。 因此我们可以得到示例中的四种应用场景并观察每种场景下的结果。

私有数据与暂态数据的关系如下图所示：

fabric私有数据与暂态数据

输入方法的选择以及是否是否私有数据依赖于具体的业务需求，因为链码函数反应 了真实的业务交易。我们可以选择普通的参数列表、暂态数据或者同时使用两种 方式的输入，也可以向公共状态或者私有数据集写入数据。我们需要的是正确地选择 需要使用的链码API。

2、应用场景概述
如前所述，我们将私有数据和暂态数据进行2X2的组合，概述如下：

fabric私有数据与暂态数据

场景1：不使用私有数据，不输入暂态数据

在这种场景下，数据被写入账本中的公开状态部分，所有的peer节点将保存相同的账本 数据。在链码中使用PutState和GetState来访问这部分数据。当我们调用链码时， 使用普通的参数列表指定输入数据。

当通道中的所有机构都需要相同的数据时，适合采用这种方式。

场景2：使用私有数据，不输入暂态数据

在这种场景下，数据被写入账本中的私有数据部分，并且只有在私有数据集定义的 机构的peer节点上会保存数据。在链码中我们使用PutPrivateData和GetPrivateData 来访问私有数据集。当我们调用链码时，使用普通的参数列表指定输入数据。

当应用中存在对部分数据的隐私需求，而对于输入数据不敏感时，可以采用这种方式。

场景3：使用私有数据，输入暂态数据

类似于场景2，数据被写入账本中的私有数据部分，只有在私有数据集定义中的 那些peer节点会保存这部分私有数据。在链码中，我们使用PutPrivateData和 GetPrivateData来访问数据集。当我们调用链码时，采用暂态数据作为输入， 因此在链码中我们需要使用GetTransient来处理输入数据。

场景4：不使用私有数据，输入暂态数据

这是一个想象的场景，只是用来表明在不使用私有数据时，也可以使用暂态数据作为输入。 我们在链码中使用PutState和GetState来将数据保存到账本的公共状态，而采用暂态数据 作为链码调用的输入参数。和之前一样，我们在链码中使用GetTransient方法来处理 输入数据。

3、演示环境搭建
在这些演示中我们使用Fabric 2.0，使用First Network作为Fabric网络。我们 采用CouchDB选项启动网络，以便于查看世界状态数据库的内容。我们将重点关注 peer0.org1.example.com (couchdb port 5984) 和 peer0.org2.example.com (couchdb port 7984) 以便查看两个机构中的节点的行为。

在私有数据部分，我们使用Org1内置的隐含私有数据集(_implicit_org_Org1MSP)。只有Org1 中的peer节点可以保存私有数据，而Org1和Org2中的节点都可以保存数据哈希。

我们修改了fabric-samples中的SACC链码。SACC链码有两个函数set和get。为了展示私有 数据和暂态数据，我们创建以下函数：

setPrivate：使用相同的参数列表，数据保存在Org1隐含的私有数据集
setPrivateTransient：使用暂态数据输入，数据保存在Org1隐含的私有数据集
setTransient：使用暂态数据输入，数据保存在公共状态
getPrivate：提取保存在Org1隐含的私有数据集中的数据
修改后的SACC链码如下：

/*
 * Copyright IBM Corp All Rights Reserved
 *
 * SPDX-License-Identifier: Apache-2.0
 */

package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-chaincode-go/shim"
	"github.com/hyperledger/fabric-protos-go/peer"
)

// SimpleAsset implements a simple chaincode to manage an asset
type SimpleAsset struct {
}

// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
	// Get the args from the transaction proposal
	args := stub.GetStringArgs()
	if len(args) != 2 {
		return shim.Error("Incorrect arguments. Expecting a key and a value")
	}

	// Set up any variables or assets here by calling stub.PutState()

	// We store the key and the value on the ledger
	err := stub.PutState(args[0], []byte(args[1]))
	if err != nil {
		return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
	}
	return shim.Success(nil)
}

// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
	// Extract the function and args from the transaction proposal
	fn, args := stub.GetFunctionAndParameters()

	var result string
	var err error
	if fn == "set" {
		result, err = set(stub, args)
	} else if fn == "setPrivate" {
		result, err = setPrivate(stub, args)
	} else if fn == "setTransient" {
		result, err = setTransient(stub, args)
	} else if fn == "setPrivateTransient" {
		result, err = setPrivateTransient(stub, args)
	} else if fn == "getPrivate" {
		result, err = getPrivate(stub, args)
	} else { // assume 'get' even if fn is nil
		result, err = get(stub, args)
	}
	if err != nil {
		return shim.Error(err.Error())
	}

	// Return the result as success payload
	return shim.Success([]byte(result))
}

// Set stores the asset (both key and value) on the ledger. If the key exists,
// it will override the value with the new one
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
	if len(args) != 2 {
		return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
	}

	err := stub.PutState(args[0], []byte(args[1]))
	if err != nil {
		return "", fmt.Errorf("Failed to set asset: %s", args[0])
	}
	return args[1], nil
}

func setPrivate(stub shim.ChaincodeStubInterface, args []string) (string, error) {
	if len(args) != 2 {
		return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
	}

	err := stub.PutPrivateData("_implicit_org_Org1MSP", args[0], []byte(args[1]))
	if err != nil {
		return "", fmt.Errorf("Failed to set asset: %s", args[0])
	}
	return args[1], nil
}

func setTransient(stub shim.ChaincodeStubInterface, args []string) (string, error) {

	type keyValueTransientInput struct {
		Key   string `json:"key"`
		Value string `json:"value"`
	}

	if len(args) != 0 {
		return "", fmt.Errorf("Incorrect arguments. Expecting no data when using transient")
	}

	transMap, err := stub.GetTransient()
	if err != nil {
		return "", fmt.Errorf("Failed to get transient")
	}

	// assuming only "name" is processed
	keyValueAsBytes, ok := transMap["keyvalue"]
	if !ok {
		return "", fmt.Errorf("key must be keyvalue")
	}

	var keyValueInput keyValueTransientInput
	err = json.Unmarshal(keyValueAsBytes, &keyValueInput)
	if err != nil {
		return "", fmt.Errorf("Failed to decode JSON")
	}

	err = stub.PutState(keyValueInput.Key, []byte(keyValueInput.Value))
	if err != nil {
		return "", fmt.Errorf("Failed to set asset")
	}
	return keyValueInput.Value, nil
}

func setPrivateTransient(stub shim.ChaincodeStubInterface, args []string) (string, error) {

	type keyValueTransientInput struct {
		Key   string `json:"key"`
		Value string `json:"value"`
	}

	if len(args) != 0 {
		return "", fmt.Errorf("Incorrect arguments. Expecting no data when using transient")
	}

	transMap, err := stub.GetTransient()
	if err != nil {
		return "", fmt.Errorf("Failed to get transient")
	}

	// assuming only "name" is processed
	keyValueAsBytes, ok := transMap["keyvalue"]
	if !ok {
		return "", fmt.Errorf("key must be keyvalue")
	}

	var keyValueInput keyValueTransientInput
	err = json.Unmarshal(keyValueAsBytes, &keyValueInput)
	if err != nil {
		return "", fmt.Errorf("Failed to decode JSON")
	}

	err = stub.PutPrivateData("_implicit_org_Org1MSP", keyValueInput.Key, []byte(keyValueInput.Value))
	if err != nil {
		return "", fmt.Errorf("Failed to set asset")
	}
	return keyValueInput.Value, nil
}

// Get returns the value of the specified asset key
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
	if len(args) != 1 {
		return "", fmt.Errorf("Incorrect arguments. Expecting a key")
	}

	value, err := stub.GetState(args[0])
	if err != nil {
		return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
	}
	if value == nil {
		return "", fmt.Errorf("Asset not found: %s", args[0])
	}
	return string(value), nil
}

// Get returns the value of the specified asset key
func getPrivate(stub shim.ChaincodeStubInterface, args []string) (string, error) {
	if len(args) != 1 {
		return "", fmt.Errorf("Incorrect arguments. Expecting a key")
	}

	value, err := stub.GetPrivateData("_implicit_org_Org1MSP", args[0])
	if err != nil {
		return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
	}
	if value == nil {
		return "", fmt.Errorf("Asset not found: %s", args[0])
	}
	return string(value), nil
}

// main function starts up the chaincode in the container during instantiate
func main() {
	if err := shim.Start(new(SimpleAsset)); err != nil {
		fmt.Printf("Error starting SimpleAsset chaincode: %s", err)
	}
}
4、Fabric私有数据和暂态数据演示
首先启动First Network，不要部署默认链码，启用CouchDB选项：

cd fabric-samples/first-network
./byfn.sh up -n -s couchdb
当看到所有容器（5个排序节点，4个peer节点，4个couchdb，一个CLI）启动后：

fabric私有数据与暂态数据

创建一个新的链码目录：

cd fabric-samples/chaincode
cp -r sacc sacc_privatetransientdemo
cd sacc_privatetransientdemo
然后使用上面的链码替换sacc.go。

在第一次运行之前我们需要先加载依赖的模块：

GO111MODULE=on go mod vendor
最后我们使用lifecycle chaincode命令部署这个链码。

5、场景1演示：不使用Fabric私有数据和暂态数据输入
场景1时最常用的一种：使用普通的参数列表作为链码方法的输入，然后将其保存在公共状态中， 所有peer节点持有完全相同的数据。我们调用链码的set和get方法。

docker exec cli peer chaincode invoke -o orderer.example.com:7050 --tls \
  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
  --peerAddresses peer0.org1.example.com:7051 \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
  --peerAddresses peer0.org2.example.com:9051 \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
  -C mychannel -n mycc -c '{"Args":["set","name","alice"]}'

docker exec cli peer chaincode query -C mychannel -n mycc -c '{"Args":["get","name"]}'
结果如下：

fabric私有数据与暂态数据

我们首先查看世界状态。这个数据同时保存在peer0.org1.example.com 和peer0.org2.example.com的公共状态中（mychannel_mycc）。

fabric私有数据与暂态数据

当查看区块链中的交易记录时，我么看到WriteSet中的键/值对是：name/alice，采用base64编码。

我们也可以看到调用链码时的参数列表，3个base64编码的参数分别是：set、name和alice。

fabric私有数据与暂态数据

和预期一样，RWSet更新了公开状态，输入参数被记录在交易中。

6、场景2演示：使用私有数据，不适用暂态数据输入
在场景2中，链码调用还是采用普通的参数列表，数据则保存在Org1的私有数据集。 我们使用setPrivate和getPrivate访问链码

docker exec cli peer chaincode invoke -o orderer.example.com:7050 --tls \
  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
  --peerAddresses peer0.org1.example.com:7051 \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
  --peerAddresses peer0.org2.example.com:9051 \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
  -C mychannel -n mycc -c '{"Args":["setPrivate","name","bob"]}'
  
docker exec cli peer chaincode query -C mychannel -n mycc -c '{"Args":["getPrivate","name"]}'
fabric私有数据与暂态数据

我们首先查看世界状态。在peer0.org1.example.com中我们可以看到数据保存为私有数据， 创建了两个数据库：一个用于实际数据，一个用于数据哈希。在peer0.org2.example.com 上，我们看到只有哈希文件。

fabric私有数据与暂态数据

内容的哈希在两个机构的节点上都是一样的。此外，在peer0.org1.example.com中我们可以 看到调用链码时输入的数据。

fabric私有数据与暂态数据

当查看区块链中的交易记录时，我们看到没有RWSet。相反我们看到数据被应用于 Org1隐含的数据集，它指向已经保存在peer中的数据，通过hash得以保护这部分数据的隐私。

fabric私有数据与暂态数据

我们可以看到调用链码时的参数列表。3个base64编码的参数分别是：setPrivate、name、bob。

fabric私有数据与暂态数据

如果我们关系数据隐私，这可能存在问题。一方面数据保存在私有数据集中， 这样只有限定的机构节点可以保存。另一方面，这部分隐私数据的链码输入 却还是公开可见的，并且永久保存在所有peer节点的区块链中。如果这不是 你期望的，那么我们还需要将输入数据隐藏掉。这就是使用暂态数据输入的原因。

6、场景3演示：使用私有数据和暂态数据输入
如果你希望确保数据输入不会保存在链上，那么场景3是推荐的方式。在这种 场景下，采用暂态数据输入，并且数据保存在Org1的私有数据集。我们使用 setPrivateTransient和getPrivate方法访问链码：

在我们的链码中，我们实现函数时将暂态数据编码为特定的JSON格式 {“key”:”some key”, “value”: “some value”} (链码134–137行)。 我们也要求暂态数据包含一个keyvalue键(链码149行)。为了在命令行调用 中使用暂态数据，我们需要首先将其进行base64编码。

export KEYVALUE=$(echo -n "{\"key\":\"name\",\"value\":\"charlie\"}" | base64 | tr -d \\n)

docker exec cli peer chaincode invoke -o orderer.example.com:7050 --tls \
  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
  --peerAddresses peer0.org1.example.com:7051 \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
  --peerAddresses peer0.org2.example.com:9051 \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
  -C mychannel -n mycc -c '{"Args":["setPrivateTransient"]}' \
  --transient "{\"keyvalue\":\"$KEYVALUE\"}"
  
docker exec cli peer chaincode query -C mychannel -n mycc -c '{"Args":["getPrivate","name"]}'
结果如下：

fabric私有数据与暂态数据

同样，我们首先查看世界状态。这类似于我们在场景2中看到的内容。实际的数据 仅在peer0.org1.example.com保存，而哈希则在两个peer节点中都有保存。注意 修订版本的值目前是2，而在场景2中的第一次修订的值是1。是链码调用促成了对 数据的修订。

fabric私有数据与暂态数据

类似于场景2，在区块链上的交易记录中，我们可以看到没有Write Set。

fabric私有数据与暂态数据

我们也可以看到没有调用链码的参数列表。唯一的参数是链码函数名，setPrivateTransient。 具体的调用数据{“key”:”name”, “value”:”charlie”}没有出现在区块链上。

fabric私有数据与暂态数据

我们看到私有数据和隐私数据的组合提供了某种程度的数据隐私。

7、场景4演示：不适用私有数据，使用暂态数据输入
最后我们看一下想象的这个场景的演示。在场景3中，采用暂态数据作为数据， 然后保存在账本的公开状态。我们使用setTransient和get访问链码：

export KEYVALUE=$(echo -n "{\"key\":\"name\",\"value\":\"david\"}" | base64 | tr -d \\n)

docker exec cli peer chaincode invoke -o orderer.example.com:7050 --tls \
  --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
  --peerAddresses peer0.org1.example.com:7051 \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
  --peerAddresses peer0.org2.example.com:9051 \
  --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
  -C mychannel -n mycc -c '{"Args":["setTransient"]}' \
  --transient "{\"keyvalue\":\"$KEYVALUE\"}"
  
docker exec cli peer chaincode query -C mychannel -n mycc -c '{"Args":["get","name"]}'
结果如下：

fabric私有数据与暂态数据

可以看到公开状态被更新，两个节点目前有同样的数据。注意修订版本更新为2.

fabric私有数据与暂态数据

我们看到Write Set中的键/值对是name和david，base64编码。

fabric私有数据与暂态数据

我们没有看到参数中的输入数据，只看到调用的方法名setTransient。

fabric私有数据与暂态数据

原文链接：Private Data and Transient Data in Hyperledger Fabric