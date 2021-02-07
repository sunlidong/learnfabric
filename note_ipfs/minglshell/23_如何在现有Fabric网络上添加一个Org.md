如何在现有Fabric网络上添加一个Org？本指南基于IBM DeveloperWorks——使用简单的工具将组织添加到现有的Hyperledger Fabric区块链网络中。

感谢Bhargav Perepa和Jason Yellick的出色工作！而且我只是想以更礼貌的方式添加一些细节。:)

完成本实验后，你应该有一个额外的组织：

*.org3.example.com
假设
在byfn.sh所在的同一路径上启动此实验。
Fabric网络已经创建并运行（BYFN样本）。
条件
使用最新的Fabric build>= 1.1.0-preview
安装jq工具
这个实验需要jq二进制文件。从jq存储库下载它们。

OS X
wget https://github.com/stedolan/jq/releases/download/jq-1.5/jq-osx-amd64
chmod +x jq-osx-amd64
sudo mv jq-osx-amd64 /usr/local/bin/jq
linux
wget https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64
chmod +x jq-linux64
sudo mv jq-linux64 /usr/local/bin/jq
为Org3创建证书
创建配置文件：

cat > crypto-config-add.yaml <<EOF
PeerOrgs:
- Name: Org3
  Domain: org3.example.com
  Template:
    Count: 1
  Users:
    Count: 1
EOF
从配置文件生成证书：

../bin/cryptogen generate --config=./crypto-config-add.yaml
您在./crypto-config/peerOrganizations/org3.example.com/上获得了Org3的证书。

运行configtxlator
configtxlator工具旨在支持重新配置Fabric网络。

../bin/configtxlator start &
查询当前配置
查询cli容器的当前配置：

docker exec -it cli peer channel fetch config config_block.pb -o orderer.example.com:7050 -c mychannel --tls --cafile ./crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

docker cp cli:/opt/gopath/src/github.com/hyperledger/fabric/peer/config_block.pb .
解码配置
使用configtxlator解码获取的配置文件：

curl -X POST --data-binary @config_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > config_block.json
扩展配置部分
从解码的配置文件config中提取扩展配置部分：

jq .data.data[0].payload.data.config config_block.json > config.json
从配置部分提取Org1MSP部分：

jq .channel_group.groups.Application.groups.Org1MSP config.json > Org1MSP.json
创建新配置
基于Org1MSP.json创建Org3MSP.json文件：

ADMIN_CERT=$(cat ./crypto-config/peerOrganizations/org3.example.com/users/Admin\@org3.example.com/msp/signcerts/Admin\@org3.example.com-cert.pem |base64 |tr -d '\n')
ROOT_CERT=$(cat ./crypto-config/peerOrganizations/org3.example.com/ca/ca.org3.example.com-cert.pem  |base64 |tr -d '\n')
TLS_ROOT_CERT=$(cat ./crypto-config/peerOrganizations/org3.example.com/tlsca/tlsca.org3.example.com-cert.pem |base64 |tr -d '\n')
jq --arg admin ${ADMIN_CERT} --arg root ${ROOT_CERT} --arg tls ${TLS_ROOT_CERT} '.values.MSP.value.config.admins[0] = $admin | .values.MSP.value.config.root_certs[0] = $root | .values.MSP.value.config.tls_root_certs[0] = $tls' Org1MSP.json > Org3MSP.json
sed -i 's/Org1MSP/Org3MSP/g' Org3MSP.json
将Org2MSP.json的内容放在config.json文件中的Org2MSP之后：

ORG3=$(cat Org3MSP.json)
jq --argjson org3 "$ORG3" '.channel_group.groups.Application.groups.Org3MSP = $org3' config.json > updated_config.json
对原始配置和修改配置进行编码
通过configtxlatr，编码配置文件，config.json和updated_config.json：

curl -X POST --data-binary @config.json http://127.0.0.1:7059/protolator/encode/common.Config > config.pb
curl -X POST --data-binary @updated_config.json http://127.0.0.1:7059/protolator/encode/common.Config > updated_config.pb
将它们发送到configtxlator以计算配置更新增量
计算配置更新增量：

curl -X POST -F original=@config.pb -F updated=@updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs -F channel=mychannel > config_update.pb
解码配置更新并将其包装到配置更新信封中
将配置更新文件解码为JSON：

curl -X POST --data-binary @config_update.pb http://127.0.0.1:7059/protolator/decode/common.ConfigUpdate > config_update.json
为配置更新消息创建一个信封：
echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' > config_update_as_envelope.json
创建新的配置交易
将封装的消息编码为protobuf格式：

curl -X POST --data-binary @ config_update_as_envelope.json http://127.0.0.1:7059/protolator/encode/common.Envelope> config_update_as_envelope.pb
在cli容器上复制新交易：

docker cp config_update_as_envelope.pb cli:/opt/gopath/src/github.com/hyperledger/fabric/peer/
通过提交新签名的配置交易来更新频道
在所有MSP上签署配置更新交易：

将Org1MSP的签名添加到新交易中

docker exec -it \
-e CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
-e CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key \
-e CORE_PEER_LOCALMSPID=Org1MSP \
-e CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt \
-e CORE_PEER_TLS_ENABLED=true \
-e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp \
cli \
peer channel signconfigtx -f config_update_as_envelope.pb \
-o orderer.example.com:7050 --tls --cafile ./crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
将Org2MSP的签名添加到新交易中

docker exec -it \
-e CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
-e CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key \
-e CORE_PEER_LOCALMSPID=Org2MSP \
-e CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt \
-e CORE_PEER_TLS_ENABLED=true \
-e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp \
cli \
peer channel signconfigtx -f config_update_as_envelope.pb \
-o orderer0.example.com:7050 --tls --cafile ./crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
提交更新的交易：

docker exec -it \
-e CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
-e CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key \
-e CORE_PEER_LOCALMSPID=Org1MSP \
-e CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt \
-e CORE_PEER_TLS_ENABLED=true \
-e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp \
cli \
peer channel update -f config_update_as_envelope.pb \
-o orderer.example.com:7050 -c mychannel --tls --cafile ./crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
现在添加Org3就完成了！

运行peer0.org3节点
为peer0.org3创建一个compose文件：

cat > docker-compose-peer0-org3.yaml <<EOF
version: '2'

networks:
  byfn:

services:
  peer0.org3.example.com:
    container_name: peer0.org3.example.com
    extends:
      file: base/peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer0.org3.example.com
      - CORE_PEER_ADDRESS=peer0.org3.example.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org3.example.com:7051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org3.example.com:7051
      - CORE_PEER_LOCALMSPID=Or3MSP
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/msp:/etc/hyperledger/fabric/msp
        - ./crypto-config/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls:/etc/hyperledger/fabric/tls
    ports:
      - 11051:7051
      - 11053:7053
    networks:
      - byfn
EOF
启动 peer0.org3:

docker-compose -f docker-compose-peer0-org3.yaml up -d
在peer2.org2上执行链码：
执行cli容器的shell：

docker exec -it cli bash
在shell上，运行以下命令：

CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/server.key
CORE_PEER_LOCALMSPID=Org3MSP
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/server.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/users/Admin\@org3.example.com/msp
CORE_PEER_ADDRESS=peer0.org3.example.com:7051

CHANNEL_NAME=mychannel
peer channel join -b ${CHANNEL_NAME}.block
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'