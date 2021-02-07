在这篇文章中，我们将学习链码读/写Hyperledger Fabric账本中世界状态的方法， 这是利用PutState和GetState这两个API来实现的。此外，GetState还有一些变体的 调用形式，使状态读取的方式更加灵活。

Hyperledger Fabric的账本由两部分组成：容纳交易记录的区块链和世界状态。 每个通道都有自己的账本，每个加入到通道中的对等节点都会保存一份账本副本。 链码（即超级账本中的智能合约）实现了所有参与方达成一致的商业逻辑， 与世界状态交互，是唯一的可信来源。

fabric state api

通过实例学习是一种好办法。这里我们选择fabric示例代码中的fabcar链码。出于 简化考虑我们使用基础网络来安装并实例化fabcar链码，它包含了用来保存 世界状态的CouchDB数据库。我们可以通过直接观察CouchDB的记录来了解Fabric的 状态是如何保存在世界状态数据库中的。

fabcar链码安装与实例化
要快速掌握超级账本链码及应用开发，推荐汇智网的在线课程： - Hyperledger Fabric Node.JS链码及应用开发详解 - Hyperledger Fabric Java链码及应用开发详解

我们使用一个fabric节点，其中包含了所有的Hyperledger Fabric的docker镜像并 安装了fabric-samples示例代码。

以下是安装步骤。

首先启动基础网络：

cd fabric-samples/basic-network
./start.sh
然后安装并实例化fabcar链码：

docker-compose up -d cli
docker exec cli peer chaincode install -n mycc -p github.com/fabcar/go -v 0

docker exec cli peer chaincode instantiate -o orderer.example.com:7050 -C \
       mychannel -n mycc github.com/fabcar/go -v 0 \
       -c '{"Args": []}' -P "OR('Org1MSP.member')"
最后调用initLedger()方法载入10个预定义的车辆记录：

docker exec cli peer chaincode invoke -C mychannel -n mycc -c '{"Args":["initLedger"]}'
Fabcar链码简介
Fabcar链码维护一个表型数据库，使用车辆ID（carid）作为索引（CARn），车辆记录 由型号、厂商、颜色和所有者这几个字段构成。Fabcar链码有5个链码函数可以调用或查询。

initLedger()： 加载10个预定义的车辆记录到账本中，只需在链码实例化之后调用一次
queryAllCars()： 返回所有车辆的记录
queryCar(carid)：返回指定ID的车辆记录
createCar(carid, make, model, colour, owner)：添加一个新的车辆记录
changeCarOwner(carid, newOwner)：修改车辆的所有者
在这些方法中我们可以看到使用了一些API来实现与账本的交互：PutState、GetState和GetStateByRange。 让我们在实例中学习这些API的工作原理。

PutState
PutState API很容易理解，作用就是设置指定key的状态。调用它需要传入一个key字符串和 一个value字节数组。函数原型如下：

func (stub *ChaincodeStub) PutState(key string, value []byte) error
让我们首先看一下initLedger()。initLedger()加载10个预定义的车辆记录到账本中。cars 是保存所有记录的数组。状态key是在迭代中生成的，而状态value则是利用数组内容生成的：

i := 0
for i < len(cars) {
  fmt.Println("i is ", i)
  carAsBytes, _ := json.Marshal(cars[i])
  APIstub.PutState("CAR"+strconv.Itoa(i), carAsBytes)
  fmt.Println("Added", cars[i])
  i = i + 1
}
可以看到上面使用了PutState：

状态键是CARi，其中i是从0开始每个循环递增1的数值，因此得到的键分别是CAR0, CAR1, CAR2, …
状态值是JSON序列化得到的字节数组
在调用initLedger()之后，我们将会看到这些记录保存在CouchDB中：

initledger result in couchdb

每个文档都是一个JSON对象，下面显示的是CAR0的内容：

couchdb car0 record

PutState没有变体形式的API。在createCar()和changeCarOwner()的 实现代码中也可以看到PutState，我们不再赘述。

GetState
GetState API返回指定键的字节数组类型的状态值，其函数原型如下：

func (stub *ChaincodeStub) GetState(key string) ([]byte, error)
我们首先看一下queryCar()函数，看看如何使用GetState方法：

carAsBytes, _ := APIstub.GetState(args[0])
return shim.Success(carAsBytes)
carAsBytes是指定键对应的字节数组值（args[0]对应cardid）。这也 容易理解。下面是queryCar()的使用示例：

querycar

在changeCarOwner()的实现中很好地展示了GetState和PutState的组合 使用方法。在这个函数中，我们首先使用GetState来获取指定carid的记录， 然后更新车辆所有者，最后使用PutState方法将结果写入账本。

carAsBytes, _ := APIstub.GetState(args[0])
car := Car{}

json.Unmarshal(carAsBytes, &car)
car.Owner = args[1]

carAsBytes, _ = json.Marshal(car)
APIstub.PutState(args[0], carAsBytes)
下面是其调用示例：

changecarowner

虽然GetState可以解决从账本读取状态的基本需求，但是它还不够灵活。 因此Fabric提供了GetState的一些变体形式，在fabcar示例中，我们可以看到 使用了GetStateByRange API。我们首先看一下GetStateByRange，然后 添加一个支持分页的新函数。

GetStateByRange
顾名思意，GetStateByRange API返回指定范围内的键对应的记录。只有在 状态键是以某种方式的范围排列时，这个方法才有意义。在fabcar示例中， 状态键的格式是CARn，因此可以使用这个API。

GetStateByRange的函数原型如下：

func (stub *ChaincodeStub) GetStateByRange(startKey, endKey string) (StateQueryIteratorInterface, error)
GetStateByRange要传入两个字符串参数：起始键startKey和结束键endKey。 返回一个迭代器，我们需要进一步处理以得到可以显示的结果。下面是queryAllCars() 中的代码：

startKey := "CAR0"
endKey := "CAR999"

resultsIterator, err := APIstub.GetStateByRange(startKey, endKey)
if err != nil {
  return shim.Error(err.Error())
}
defer resultsIterator.Close()
使用GetState我们可以直接得到一个指定键对应的值。GetStateByRange则 返回一个迭代器。下面的代码我们可以看到如何使用迭代器得到可以展示的结果：

// buffer is a JSON array containing QueryResults
var buffer bytes.Buffer
buffer.WriteString("[")

bArrayMemberAlreadyWritten := false
for resultsIterator.HasNext() {
  queryResponse, err := resultsIterator.Next()
  if err != nil {
    return shim.Error(err.Error())
  }
  // Add a comma before array members, suppress it for the first array member
  if bArrayMemberAlreadyWritten == true {
    buffer.WriteString(",")
  }
  buffer.WriteString("{\"Key\":")
  buffer.WriteString("\"")
  buffer.WriteString(queryResponse.Key)
  buffer.WriteString("\"")

  buffer.WriteString(", \"Record\":")
  // Record is a JSON object, so we write as-is
  buffer.WriteString(string(queryResponse.Value))
  buffer.WriteString("}")
  bArrayMemberAlreadyWritten = true
}
buffer.WriteString("]")

fmt.Printf("- queryAllCars:\n%s\n", buffer.String())

return shim.Success(buffer.Bytes())
下面是结果：

range result iterator

正如我们在示例中看到的，GetStateByRange返回指定范围内的键对应的所有 结果。假设我们只需要一小部分记录，就需要进行分页处理。

GetStateByRangeWithPagination
通过指定起始记录以及结果数量，分页提供了数据记录的滑动窗口。 这个方法基于GetStateByRange提供了额外的灵活性。 想象一下，假设你有数千条记录。你可以只选择部分感兴趣的记录返回。

GetStateByRangeWithPagination函数原型如下：

func (stub *ChaincodeStub) 
  GetStateByRangeWithPagination(startKey, endKey string, pageSize int32, bookmark string) 
  (StateQueryIteratorInterface, *pb.QueryResponseMetadata, error)
这个API的startKey和endKey参数与GetStateByRange一样，都用来指定要查询 的键的范围。除此之外，pageSize指定结果大小，bookmark则用来指定起始记录。

现在我们可以添加一个新的函数queryAllCarsWithPagination()，它需要两个参数： pageSize 和 bookmark。

我们首先在链码目录创建fabcar链码项目的一个拷贝，目录为chaincode/testrangepage/。

cd fabric-samples/chaincode
cp -r fabcar/go/ testrangepage/
cd testrangepage
mv fabcar.go testrangepage.go
让我们先处理链码文件testrangepage.go，有两部分需要修改。

首先，修改Invoke()，添加queryAllCarsWithPagination调用：

func (s *SmartContract) Invoke(APIstub shim.ChaincodeStubInterface) sc.Response {

	// Retrieve the requested Smart Contract function and arguments
	function, args := APIstub.GetFunctionAndParameters()
	// Route to the appropriate handler function to interact with the ledger appropriately
	if function == "queryCar" {
		return s.queryCar(APIstub, args)
	} else if function == "initLedger" {
		return s.initLedger(APIstub)
	} else if function == "createCar" {
		return s.createCar(APIstub, args)
	} else if function == "queryAllCars" {
		return s.queryAllCars(APIstub)
	} else if function == "changeCarOwner" {
		return s.changeCarOwner(APIstub, args)
	} else if function == "queryAllCarsWithPagination" {
		return s.queryAllCarsWithPagination(APIstub, args)
	}

	return shim.Error("Invalid Smart Contract function name.")
}
第二部分是新函数，它是对queryAllCars()的修改，部分代码基于另一个 链码示例marbles02：

func (s *SmartContract) queryAllCarsWithPagination(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {

	if len(args) < 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	startKey := "CAR0"
	endKey := "CAR999"

	pageSize, err := strconv.ParseInt(args[0], 10, 32)
	if err != nil {
		return shim.Error(err.Error())
	}
	bookmark := args[1]

	resultsIterator, _, err := APIstub.GetStateByRangeWithPagination(startKey, endKey, int32(pageSize), bookmark)
	if err != nil {
		return shim.Error(err.Error())
	}
	defer resultsIterator.Close()

	// buffer is a JSON array containing QueryResults
	var buffer bytes.Buffer
	buffer.WriteString("[")

	bArrayMemberAlreadyWritten := false
	for resultsIterator.HasNext() {
		queryResponse, err := resultsIterator.Next()
		if err != nil {
			return shim.Error(err.Error())
		}
		// Add a comma before array members, suppress it for the first array member
		if bArrayMemberAlreadyWritten == true {
			buffer.WriteString(",")
		}
		buffer.WriteString("{\"Key\":")
		buffer.WriteString("\"")
		buffer.WriteString(queryResponse.Key)
		buffer.WriteString("\"")

		buffer.WriteString(", \"Record\":")
		// Record is a JSON object, so we write as-is
		buffer.WriteString(string(queryResponse.Value))
		buffer.WriteString("}")
		bArrayMemberAlreadyWritten = true
	}
	buffer.WriteString("]")

	fmt.Printf("- queryAllCars:\n%s\n", buffer.String())

	return shim.Success(buffer.Bytes())
}
说明如下：

调用queryAllCarsWithPagination()时我们需要两个参数，页大小和书签（开始的carid）
startKey和endKey是GetStateByRangeWithPagination所需要的
结果是一个迭代器、响应元数据和错误。元数据中包含了记录数量。
结果迭代器的处理和queryAllCars()一样
最终的代码如下。为了清晰起见，我们删减了一些代码，使用新的链码启动一个 新的基础网络：

cd fabric-samples/basic-network
./teardown.sh
./start.sh

docker-compose up -d cli
docker exec cli peer chaincode install -n mycc -p github.com/testrangepage -v 0

docker exec cli peer chaincode instantiate -o orderer.example.com:7050 -C mychannel \
       -n mycc github.com/testrangepage -v 0 -c '{"Args": []}' -P "OR('Org1MSP.member')"

docker exec cli peer chaincode invoke -C mychannel -n mycc -c '{"Args":["initLedger"]}'
先看一下前面的queryAllCars()调用，可以看到已经有10条记录：

query

现在假设我们希望获取从CAR3开始的5条记录，可以使用queryAllCarsWithPagination()。 结果应当是CAR3 至 CAR7。

query

如何查询从CAR8开始的5条记录？我们只看到了CAR8和CAR9。

query

下面是分页的工作原理示意：

pagination