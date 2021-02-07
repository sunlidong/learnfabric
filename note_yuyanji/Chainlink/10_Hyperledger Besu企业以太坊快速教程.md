Besu是Hyperledger中的企业以太坊产品，其最大优势在于兼容以太坊主网。 本教程介绍如何使用Hyperledger Besu快速启动一个企业以太坊网络并利用 JSON RPC进行数据查询和交易提交，以及如何使用Truffle开发企业以太坊 DApp并使用内置的工具进行数据调试和运维监控。

1、启动企业以太坊网络
以太坊教程推荐： Dapp入门 | 电商Dapp实战 | Token实战 | Php对接 | Java对接 | Python对接 | C#对接 | Dart对接

首先克隆Besu的quickstart仓库的源代码：

git clone https://github.com/PegaSysEng/besu-quickstart.git
然后进入besu-quickstart目录，执行如下命令构建besu的docker镜像：

./run.sh
上面的命令会构建docker镜像并启动4个容器来模拟一个 包含6个besu节点的企业以太坊网络。当脚本执行完成后，你可以看到如下 输出信息：

*************************************
Besu Quickstart <version>
*************************************
List endpoints and services
----------------------------------
              Name                            Command               State               Ports
---------------------------------------------------------------------------------------------------------
besu-quickstart_bootnode_1     /opt/besu/bootnode_sta ...   Up      30303/tcp, 8545/tcp, 8546/tcp
besu-quickstart_explorer_1     nginx -g daemon off;             Up      0.0.0.0:32768->80/tcp
besu-quickstart_grafana_1      /run.sh                          Up      3000/tcp
besu-quickstart_minernode_1    /opt/besu/node_start.s ...   Up      30303/tcp, 8545/tcp, 8546/tcp
besu-quickstart_node_1         /opt/besu/node_start.s ...   Up      30303/tcp, 8545/tcp, 8546/tcp
besu-quickstart_node_2         /opt/besu/node_start.s ...   Up      30303/tcp, 8545/tcp, 8546/tcp
besu-quickstart_node_3         /opt/besu/node_start.s ...   Up      30303/tcp, 8545/tcp, 8546/tcp
besu-quickstart_node_4         /opt/besu/node_start.s ...   Up      30303/tcp, 8545/tcp, 8546/tcp
besu-quickstart_prometheus_1   /bin/prometheus --config.f ...   Up      9090/tcp
besu-quickstart_rpcnode_1      /opt/besu/node_start.s ...   Up      30303/tcp, 8545/tcp, 8546/tcp
以及访问端结点列表：

****************************************************************
JSON-RPC HTTP service endpoint      : http://localhost:32768/jsonrpc
JSON-RPC WebSocket service endpoint : ws://localhost:32768/jsonws
GraphQL HTTP service endpoint       : http://localhost:32768/graphql
Web block explorer address          : http://localhost:32768
Prometheus address                  : http://localhost:32768/prometheus/graph
Grafana address                     : http://localhost:32768/grafana-dashboard                                                                        
****************************************************************
JSON-RPC HTTP服务用于DApp或Metamask钱包的访问
JSON-RPC WebSocket服务用于DApp通过websocket访问节点
GraphQL HTTP服务用于DApp或Metamask钱包的访问节点的GraphQL服务
Web区块浏览服务用于浏览区块，在你的浏览器中输入该地址即可
Prometheus服务用于为Prometheus仪表盘提供指标数据
Grafana服务用于为Grafana仪表盘提供数据
要再次显示访问端结点，可以使用如下命令：

./list.sh
2、使用企业以太坊区块浏览器
在本教程中我们使用Alethio轻量级以太坊浏览器，你也可以使用EthScan。

在你的浏览器中打开前面提到的web block explorer endpoint列出的地址，就可以 查看企业以太坊网络中的区块数据了。



可以在区块浏览器中看到有6个besu节点：4个普通节点、1个出块节点和一个引导节点。

点击 Best Block 右侧的区块号就可以显示该区块的详细数据：



点击左上角的放大镜，就可以搜索区块、交易哈希、或以太坊地址：



3、监视Besu节点的运行状况
可以使用Prometheus和Grafana这些运维监视工具来可视化节点的健康状态 和运行情况。参考前面给出的访问端结点，可以在你的浏览器中直接访问这些工具。 例如使用Grafana：



4、使用JSON-RPC访问Besu节点
Besu支持标准的以太坊JSON-RPC API接口。例如使用curl调用web3_clientVersion 命令来查看节点的版本：

curl -X POST --data '{
	"jsonrpc":"2.0",
	"method":"web3_clientVersion",
	"params":[],
	"id":1
}' <http-rpc-endpoint>
其中<http-rpc-endpoint>表示前面列出的访问端结点的地址，你需要 根据自己的实际情况替换，例如http://localhost:32768/jsonrpc。 上面命令的返回结果类似如下：

{
   "jsonrpc" : "2.0",
   "id" : 1,
   "result" : "besu/<version number>"
}
或者使用net_peerCount命令 查看节点已连接的Peer数量：

curl -X POST --data '{
	"jsonrpc":"2.0",
	"method":"net_peerCount",
	"params":[],
	"id":1
}' <http-rpc-endpoint>
结果如下：

{
  "jsonrpc" : "2.0",
  "id" : 1,
  "result" : "0x6"
}
或者使用eth_blockNumber查看 最新的区块号：

curl -X POST --data '{
	"jsonrpc":"2.0",
	"method":"eth_blockNumber",
	"params":[],
	"id":1
}' <http-rpc-endpoint>
结果如下：

{
  "jsonrpc" : "2.0",
  "id" : 1,
  "result" : "0x8b8"
}
5、使用MetaMask创建企业以太坊交易
在发送企业以太坊交易之前，我们需要先创建一个账号，或者使用这个私有网络的 创世配置中已经声明的几个账号：

账号1 ：同时也是币基地址

Address: 0xfe3b557e8fb62b89f4916b721be55ceb828dbd73
Private key : 0x8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63
Initial balance : 0xad78ebc5ac6200000 (200000000000000000000 in decimal)
账号2：

Address: 0x627306090abaB3A6e1400e9345bC60c78a8BEf57
Private key : 0xc87509a1c067bbde78beb793e6fa76530b6382a4c0241e5e4a9ec0a0f44dc0d3
Initial balance : 0x90000000000000000000000 (2785365088392105618523029504 in decimal)
账号3：

Address: 0xf17f52151EbEF6C7334FAD080c5704D77216b732
Private key : 0xae6ae8e5ccbfb04590405997ee2d52d2b330726137b875053c36d94e974d162f
Initial balance : 0x90000000000000000000000 (2785365088392105618523029504 in decimal
在登录进MetaMask之后，链接到我们建立的Besu网络的某个节点，然后你就可以 创建交易了。

6、基于Besu网络的Truffle宠物商店演示
要运行Truffle的Pet Shop演示，首先我们需要安装truffle及pet-shop模板， 然后还需要针对Besu的企业以太坊网络进行一些简单的调整。

首先安装truffle：

npm install -g truffle
然后创建pet-shop-tutorial目录并进入该目录：

mkdir pet-shop-tutorial
cd pet-shop-tutorial
然后解压Truffle的pet-shop box：

truffle unbox pet-shop
安装truffle-wallet：

npm install --save @truffle/hdwallet-provider
接下来修改pet-shop-tutorial目录中的truffle-config.js文件，以便 添加我们的钱包提供器。请参考以下内容进行修改：

const PrivateKeyProvider = require("truffle-hdwallet-provider");
const privateKey = "8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63";

module.exports = {
  // See <http://truffleframework.com/docs/advanced/configuration>
  // for more about customizing your Truffle configuration!
  networks: {
    development: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*" // Match any network id
    },
    quickstartWallet: {
      provider: () => new PrivateKeyProvider(privateKey, "<YOUR HTTP RPC NODE ENDPOINT>"),
      network_id: "*"
    },
  }
};
将 <YOUR HTTP RPC NODE ENDPOINT>替换为你的HTTP RPC访问端结点， 例如 http://localhost:32770/jsonrpc 。

将privateKey替换为前面的账户1，即币基地址，其中有以太币。

由于我们使用企业以太坊网络而不是Ganache仿真器，因此在执行合约 部署时，需要指定网络：

truffle migrate --network quickstartWallet
输出结果类似如下：

sing network 'quickstartWallet'.

Running migration: 1_initial_migration.js
  Deploying Migrations...
  ... 0xfc1dbc1eaa14fa283c2c4415364579da0d195b3f2f2fefd7e0edb600a6235bdb
  Migrations: 0x9a3dbca554e9f6b9257aaa24010da8377c57c17e
Saving successful migration to network...
  ... 0x77cc6e9966b886fb74268f118b3ff44cf973d32b616ed4f050b3eabf0a31a30e
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Deploying Adoption...
  ... 0x5035fe3ea7dab1d81482acc1259450b8bf8fefecfbe1749212aca86dc765660a
  Adoption: 0x2e1f232a9439c3d459fceca0beef13acc8259dd8
Saving successful migration to network...
  ... 0xa7b5a36e0ebc9c25445ce29ff1339a19082d0dda516e5b72c06ee6b99a901ec0
Saving artifacts...
你可以在区块浏览器中查看上述输出中的合约地址。

同样，在执行测试时也要指定使用我们的企业以太坊网络：

truffle test --network quickstartWallet
输出结果如下：

Using network 'quickstartWallet'.

Compiling ./contracts/Adoption.sol...
Compiling ./test/TestAdoption.sol...
Compiling truffle/Assert.sol...
Compiling truffle/DeployedAddresses.sol...


  TestAdoption
    ✓ testUserCanAdoptPet (2071ms)
    ✓ testGetAdopterAddressByPetId (6070ms)
    ✓ testGetAdopterAddressByPetIdInArray (6077ms)


  3 passing (37s)
7、企业以太坊网络的停止/重启/清理
使用如下脚本停止Besu构成的企业以太坊网络，但不删除dockers容器：

./stop.sh
使用如下命令重新启动企业以太坊网络：

./resume.sh
如果要停止网络并删除相应的docker容器，使用如下命令：

./remove.sh
