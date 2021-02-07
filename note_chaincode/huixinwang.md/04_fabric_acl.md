FABRIC访问控制清单/ACL配置教程
标签： Hyperledger  Fabric

在这个教程中，我们将学习Hyperledger Fabric区块链的访问控制列表（ACL）的配置与动态更新方法。教程分为两个部分：1、理解并配置Hyperledger Fabric的访问控制列表2、动态更新通道配置中的访问控制列表。我们将介绍fabric中的默认ACL内容及格式，以通道管理员的角色进行通道ACL的配置管理。

相关教程：Fabric区块链Java开发详解 | Fabric区块链Node.JS开发详解

1、HYPERLEDGER FABRIC访问控制列表/ACL的基本概念
在Hyperledger Fabric中有两种类型的访问控制策略：

签名策略：Signature Policies
隐性元策略：Implicit Meta Policies
签名策略通过检查请求中的签名来识别特定的用户。例如：

Policies:
  MyPolicy:
    Type: Signature
    Rule: “Org1.Peer OR Org2.Peer”
签名策略支持的关键字包括：AND、OR和NOutOf，利用这几个关键字可以组合出强大的访问控制规则，例如：

A机构的管理员签名的请求可以放行
20个机构中超过半数的管理员签名的请求可以放行
隐性元策略则通过聚合后代签名策略来定义访问控制规则，它支持默认的访问规则例如“超过半数的机构管理员签名的请求可以放行”。隐性元策略的定义方法与签名策略类似但略有区别，其形式如下：

<ALL|ANY|MAJORITY> <sub_policy>
下面是一个隐性元策略的示例：

Policies:
  AnotherPolicy:
    Type: ImplicitMeta
    Rule: "MAJORITY Admins"
2、HYPERLEDGER FABRIC的默认访问控制清单
默认的访问控制规则定义在configtx.yaml中，用来供configtxgen生成通道配置。在官方提供的configtx.yaml示例中，第35行定义了签名策略，第194行定义了隐性元策略，而第131行则定义了访问控制清单/ACL。

3、自定义HYPERLEDGER FABRIC的访问控制清单
让我们编辑configtx.yaml中的Application: ACLs部分来修改以下内容：

peer/Propose: /Channel/Application/Writers
为：

peer/Propose: /Channel/Application/MyPolicy
其中MyPolicy这个策略定义如下：

Policies: 
    Readers:
        Type: ImplicitMeta
        Rule: "ANY Readers"
    Writers:
        Type: ImplicitMeta
        Rule: "ANY Writers"
    Admins:
        Type: ImplicitMeta
        Rule: "MAJORITY Admins"
    MyPolicy:
        Type: Signature
        Rule: "OR('Org1MSP.client')"
MyPolicy策略声明了只有Client角色可以执行相应的任务。

别忘了生成并更新CA和管理员证书。

现在让我们尝试从Org1Client来调用链码：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-m5xvsMUm-1577590128665)(hyperledger-fabric-acl-config/client-invoke.png)]

现在使用Org2Client来调用链码：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ty1esPuQ-1577590128666)(hyperledger-fabric-acl-config/client2-invoke.png)]

可以清楚的看到，peer/propose已经不接受ORG2的Client的调用了。

4、动态更新HYPERLEDGER FABRIC通道的ACL配置
有两种方法可以用来更新访问控制策略：

编辑configtx.yaml，仅适用于后续建立的新通道
直接更新特定通道中的ACL配置，适用于已有的通道
在下面我们将展示如何更新已有通道中的访问控制清单配置。

在执行以下操作之前，记得先启动你的Hyperledger Fabric网络。

4.1 访问命令行接口
Hyperledger Fabric有一个自动创建的cli容器，可以提供操作节点的命令行接口。执行如下命令进入cli界面：

docker exec -it cli bash
然后设置程序需要使用的环境变量：

export CHANNEL_NAME=mychannel
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
4.2 获取指定FABRIC通道的当前配置
执行下面命令获取通道的当前配置并写入文件config_block.pb：

peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
4.3 将通道配置转换为JSON格式
config_block.pb是二进制编码的区块配置数据，我们要将其先转换为
容易查看、修改的JSON格式：

configtxlator proto_decode --input config_block.pb --type common.Block | jq .data.data[0].payload.data.config > config.json
4.4 创建通道配置的JSON副本以便修改
后续的修改将在副本modified_config.json上进行：

cp config.json modified_config.json
4.5 修改JSON副本的通道配置
可以使用你喜欢的任何编辑器来修改JSON副本，比如用vim：

vim modified_config.json
我们将MyPolicy的描述从Org1MSP修改为Org2MSP：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-m43EBsyt-1577590128666)(hyperledger-fabric-acl-config/mypolicy.png)]

修改后记得保存。

4.6 将修改后的通道配置JSON副本转换为二进制格式
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
4.7 将CONFIG.JSON转换为区块二进制格式
configtxlator proto_encode --input config.json --type common.Config --output config.pb
4.8 生成修改前后通道配置的差异
configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_config.pb --output diff_config.pb
4.9 将配置的差异部分转换为JSON格式
configtxlator proto_decode --input diff_config.pb --type common.ConfigUpdate | jq . > diff_config.json
4.10 封装FABRIC配置更新消息
echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":'$(cat diff_config.json)'}}}' | jq . > diff_config_envelope.json
4.11 将配置更新消息转换为二进制格式
configtxlator proto_encode --input diff_config_envelope.json --type common.Envelope --output diff_config_envelope.pb
4.12 签名配置更新消息
首先以Org1的管理员签名，设置环境变量：

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
然后签名:

peer channel signconfigtx -f diff_config_envelope.pb
然后以Org2的管理员身份签名，设置环境变量：

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
签名：

peer channel signconfigtx -f diff_config_envelope.pb
4.13 提交通道配置更新
执行如下命令向排序节点提交通道更新交易：

peer channel update -f diff_config_envelope.pb -c $CHANNEL_NAME -o orderer.example.com:7050 --tls --cafile $ORDERER_CA
现在让我们检查下效果。首先用Org1的Client调用链码：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-eg3gS69A-1577590128667)(hyperledger-fabric-acl-config/client-invoke-2.png)]

果然失败了。接下来用Org2的Client调用链码：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-C3sZuvuM-1577590128668)(hyperledger-fabric-acl-config/client2-invoke-2.png)]

和预期也一样，成功了。