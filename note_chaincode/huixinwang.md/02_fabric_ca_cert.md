安装FABIRC-CA-CLIENT
获取fabric-ca源码

go get github.com/hyperledger/fabric-ca
1
切换到v1.4.0分支

git checkout v1.4.0
1
编译安装client

cd cmd/fabric-ca-client
go install
1
2
注意：需要将GOPATH/bin 添加到环境变量

FABRIC-CA交互原理
启用TLS
这里说的tls是指fabric-ca-server和fabric-ca-client之间加密通信

先看下fabric-ca-server例子：

要启用tls，需要配置环境变量FABRIC_CA_SERVER_TLS_ENABLED=true

version: '2'

networks:
  fabric-ca:
    driver: bridge

services:
  rca-org1:
    container_name: rca-org1
    image: hyperledger/fabric-ca:1.4.0
    command: sh -c 'fabric-ca-server start -d -b rca-org1-admin:rca-org1-adminpw --port 7054'
    environment:
    - FABRIC_CA_SERVER_HOME=/tmp/hyperledger/fabric-ca/crypto #指定 文件生成目录
    - FABRIC_CA_SERVER_TLS_ENABLED=true # 为true 开启tls
    - FABRIC_CA_SERVER_CSR_CN=rca-org1
    - FABRIC_CA_SERVER_CSR_HOSTS=0.0.0.0
    - FABRIC_CA_SERVER_DEBUG=true
    volumes:
    - /tmp/hyperledger/org1/ca:/tmp/hyperledger/fabric-ca
    networks:
    - fabric-ca
    ports:
    - 7054:7054

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
当fabric-ca-server启动后会生成如下文件：

.
├── IssuerPublicKey
├── IssuerRevocationPublicKey
├── ca-cert.pem
├── fabric-ca-server-config.yaml
├── fabric-ca-server.db
├── msp
│   ├── cacerts
│   ├── keystore
│   │   ├── 152ffdda48e8cc8d94607b8643879d9be4491407ff7fb4c1276d34a58b1853f3_sk
│   │   ├── 5ec03bf46a18427f01bf1ad4dad0bb1ab2fe5a383a1af199f2f1ef479645b8bf_sk
│   │   ├── IssuerRevocationPrivateKey
│   │   └── IssuerSecretKey
│   ├── signcerts
│   └── user
└── tls-cert.pem
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
这里只需要关注ca-cert.pem文件即可，fabric-ca-client要使用该文件与fabric-ca-server安全通信；通过配置环境变量FABRIC_CA_CLIENT_TLS_CERTFILES 指明证书的位置即可以实现安全的通信。

如下例子：

# 证书环境变量
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
# 客户端 文件生成目录
export FABRIC_CA_CLIENT_HOME=./ca/admin
# 获取ca管理员的证书，后续需要ca的管理进行账号注册
fabric-ca-client enroll -d -u https://rca-org1-admin:rca-org1-adminpw@0.0.0.0:7054
1
2
3
4
5
6
证书生成
证书生成需要两步：

register 注册账号

fabric-ca-client register -d --id.name admin-org1 --id.secret org1AdminPW --id.type user -u https://0.0.0.0:7054
1
–id.name 是账号名（不可重复）

–id.secret 是密码（可不填，会自动生成）

–id.type 账户类型（有五种：admin、user、peer、orderer、client）

enroll 颁发证书

假设生成如下节点的证书：

peer1
peer2
admin （组织的admin）
先注册账号，使用ca的管理员（只有ca的管理员才有这个权限）注册账号

export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
# 事先已经enroll了 ca管理的证书，这里可以直接使用了
export FABRIC_CA_CLIENT_HOME=./ca/admin
# 注册该组织的admin
fabric-ca-client register -d --id.name admin-org1 --id.secret org1AdminPW --id.type client -u https://0.0.0.0:7054
fabric-ca-client register -d --id.name peer1-org1 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7054
fabric-ca-client register -d --id.name peer2-org1 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7054
1
2
3
4
5
6
7
颁发admin-org1（组织管理员）的证书

export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./admin
fabric-ca-client enroll -d -u https://admin-org1:org1AdminPW@0.0.0.0:7054
1
2
3
颁发peer1-org1的证书

export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./peer1
fabric-ca-client enroll -d -u https://peer1-org1:peer1PW@0.0.0.0:7054
1
2
3
颁发peer2-org1的证书

export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./peer2
fabric-ca-client enroll -d -u https://peer2-org1:peer2PW@0.0.0.0:7054
1
2
3
证书生成如图所示：

image-20200201154126497

使用FABIRC-CA生成各节点证书
基于官方案例搭建4Fabric-ca（orderer-ca、org1-ca、org2-ca、tls-ca）节点的2组织（2peer）1排序节点的solo版fabric网络，peer和orderer使用同一个tls-ca来颁发tls证书。

官方图片

先分析一下上图，黄色的线是向tls-ca申请tls证书用于fabric网络各组件之间的安全通信，蓝色的线是指peer与orerer进行通信，黑色的线这有两种类型，第一种用于peer-cli与peer进行通信（创建通道、安装链码、实例化链码、调用等），第二种是ca-cli向ca申请各个节点的msp证书。

启动TLS CA
编写tls-ca.yaml文件

version: '2'

networks:
  fabric-ca:
    driver: bridge

services:
  ca-tls:
   container_name: ca-tls
   image: hyperledger/fabric-ca:1.4.0
   command: sh -c 'fabric-ca-server start -d -b tls-ca-admin:tls-ca-adminpw --port 7052'
   environment:
      - FABRIC_CA_SERVER_HOME=/tmp/hyperledger/fabric-ca/crypto
      - FABRIC_CA_SERVER_CSR_CN=tls-ca
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_CSR_HOSTS=0.0.0.0
      - FABRIC_CA_SERVER_DEBUG=true
   volumes:
      - /tmp/hyperledger/tls-ca:/tmp/hyperledger/fabric-ca
   networks:
      - fabric-ca
   ports:
      - 7052:7052
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
启动 tls-ca

docker-compose -f tls-ca.yaml up
1
颁发TLS-CA管理员证书
mkdir tls && cd $_
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/tls-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./ca/admin
fabric-ca-client enroll -d -u https://tls-ca-admin:tls-ca-adminpw@0.0.0.0:7052
1
2
3
4
注册其他节点账号
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/tls-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./ca/admin
fabric-ca-client register -d --id.name orderer1-orderer --id.secret orderer1PW --id.type orderer -u https://0.0.0.0:7052
fabric-ca-client register -d --id.name peer1-org1 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7052
fabric-ca-client register -d --id.name peer2-org1 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7052
fabric-ca-client register -d --id.name peer1-org2 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7052
fabric-ca-client register -d --id.name peer2-org2 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7052
1
2
3
4
5
6
7
启动ORDERER-CA
编写orderer-ca.yaml文件

version: '2'

networks:
  fabric-ca:
    driver: bridge

services:
  rca-org0:
    container_name: orderer-ca
    image: hyperledger/fabric-ca:1.4.0
    command: sh -c 'fabric-ca-server start -d -b rca-org0-admin:rca-org0-adminpw --port 7053'
    environment:
    - FABRIC_CA_SERVER_HOME=/tmp/hyperledger/fabric-ca/crypto
    - FABRIC_CA_SERVER_TLS_ENABLED=true
    - FABRIC_CA_SERVER_CSR_CN=orderer-ca
    - FABRIC_CA_SERVER_CSR_HOSTS=0.0.0.0
    - FABRIC_CA_SERVER_DEBUG=true
    volumes:
    - /tmp/hyperledger/orderer-ca:/tmp/hyperledger/fabric-ca
    networks:
    - fabric-ca
    ports:
    - 7053:7053
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
启动orderer-ca

docker-compose -f orderer-ca.yaml up
1
颁发ORDERER-CA管理员证书
cd ../
mkdir orderer && cd $_
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/orderer-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./ca/admin
fabric-ca-client enroll -d -u https://rca-org0-admin:rca-org0-adminpw@0.0.0.0:7053
1
2
3
4
5
注册以下账号：

orderer1-orderer
admin-orderer
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/orderer-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./ca/admin
fabric-ca-client register -d --id.name orderer1-orderer --id.secret orderer1PW --id.type orderer -u https://0.0.0.0:7053
fabric-ca-client register -d --id.name admin-orderer --id.secret adminPW --id.type admin --id.attrs "hf.Registrar.Roles=client,hf.Registrar.Attributes=*,hf.Revoker=true,hf.GenCRL=true,admin=true:ecert,abac.init=true:ecert" -u https://0.0.0.0:7053
1
2
3
4
颁发ORDERER1-ORDERER的MSP证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/orderer-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./orderer1
fabric-ca-client enroll -d -u https://orderer1-orderer:orderer1PW@0.0.0.0:7053
1
2
3
颁发ORDERER1-ORDERER的TLS证书
# 这里指定的是tls-ca的ca-cert.pem
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/tls-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=tls-msp
# 这里需要指定额外的参数
# --enrollment.profile tls 证书类型tls
# --csr.hosts orderer1-orderer 访问orderer的域名
fabric-ca-client enroll -d -u https://orderer1-orderer:orderer1PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts orderer1-orderer
# 将orderer1/tls-msp/keystore/下的文件重命名为key.pem
mv orderer1/tls-msp/keystore/93b79eab8c8a62e5ba4eba3f361574ad7785ab9da6f0de13a5903eca6add2300_sk orderer1/tls-msp/keystore/key.pem
1
2
3
4
5
6
7
8
9
颁发ORDERER组织管理员MSP证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/orderer-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./admin
export FABRIC_CA_CLIENT_MSPDIR=msp
fabric-ca-client enroll -d -u https://admin-orderer:adminPW@0.0.0.0:7053
# 创建admincerts文件夹
mkdir orderer1/msp/admincerts
# 拷贝文件
cp admin/msp/signcerts/cert.pem  orderer1/msp/admincerts
1
2
3
4
5
6
7
8
颁发ORDERER组织的MSP证书
组织的msp证书来源于以下途径：

本组织CA的ca-cert.pem
TLS-CA的ca-cert.pem
组织管理员证书下的signcerts/cert.pem证书
mkdir msp
mkdir msp/admincerts
# 拷贝组织管理员证书下的signcerts/cert.pem证书
cp admin/msp/signcerts/cert.pem msp/admincerts/cert.pem
mkdir msp/cacerts
# 拷贝该组织ca的ca-cert.pem证书
cp /tmp/hyperledger/orderer-ca/crypto/ca-cert.pem msp/cacerts/cert.pem
mkdir msp/tlscacerts
# 拷贝tls-ca的ca-cert.pem证书
cp /tmp/hyperledger/tls-ca/crypto/ca-cert.pem msp/tlscacerts/cert.pem
1
2
3
4
5
6
7
8
9
10
完整的文件目录如下：

.
├── admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── IssuerPublicKey
│       ├── IssuerRevocationPublicKey
│       ├── cacerts
│       │   └── 0-0-0-0-7053.pem
│       ├── keystore
│       │   └── 8ecb3b8f0002e4bbcd180885954a64677e4e6382651db2cc5db675f44d5e32fc_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── ca
│   └── admin
│       ├── fabric-ca-client-config.yaml
│       └── msp
│           ├── IssuerPublicKey
│           ├── IssuerRevocationPublicKey
│           ├── cacerts
│           │   └── 0-0-0-0-7053.pem
│           ├── keystore
│           │   └── 665a09c0d2644f05f1526553aa42203ef5f4081852596b12c1f8118b7f373189_sk
│           ├── signcerts
│           │   └── cert.pem
│           └── user
├── msp
│   ├── admincerts
│   │   └── cert.pem
│   ├── cacerts
│   │   └── cert.pem
│   └── tlscacerts
│       └── cert.pem
└── orderer1
    ├── fabric-ca-client-config.yaml
    ├── msp
    │   ├── IssuerPublicKey
    │   ├── IssuerRevocationPublicKey
    │   ├── admincerts
    │   │   └── cert.pem
    │   ├── cacerts
    │   │   └── 0-0-0-0-7053.pem
    │   ├── keystore
    │   │   └── 8a13b4cbd603ecc7218d18c499eb58ca3b44297ec12ecd7d65074a750459ab63_sk
    │   ├── signcerts
    │   │   └── cert.pem
    │   └── user
    └── tls-msp
        ├── IssuerPublicKey
        ├── IssuerRevocationPublicKey
        ├── cacerts
        ├── keystore
        │   └── key.pem
        ├── signcerts
        │   └── cert.pem
        ├── tlscacerts
        │   └── tls-0-0-0-0-7052.pem
        └── user

30 directories, 27 files
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
启动ORG1-CA
编写rca-org1.yaml文件

version: '2'

networks:
  fabric-ca:
    driver: bridge

services:
  rca-org1:
    container_name: rca-org1
    image: hyperledger/fabric-ca:1.4.0
    command: sh -c 'fabric-ca-server start -d -b rca-org1-admin:rca-org1-adminpw --port 7054'
    environment:
    - FABRIC_CA_SERVER_HOME=/tmp/hyperledger/fabric-ca/crypto
    - FABRIC_CA_SERVER_TLS_ENABLED=true
    - FABRIC_CA_SERVER_CSR_CN=rca-org1
    - FABRIC_CA_SERVER_CSR_HOSTS=0.0.0.0
    - FABRIC_CA_SERVER_DEBUG=true
    volumes:
    - /tmp/hyperledger/org1/ca:/tmp/hyperledger/fabric-ca
    networks:
    - fabric-ca
    ports:
    - 7054:7054
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
启动rca-org1

docker-compose -f rca-org1.yaml up
1
颁发RCA-ORG1 CA的管理员证书
cd ..
mkdir org1 && cd $_
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp
export FABRIC_CA_CLIENT_HOME=./ca/admin
fabric-ca-client enroll -d -u https://rca-org1-admin:rca-org1-adminpw@0.0.0.0:7054
1
2
3
4
5
6
注册以下账号：

peer1-org1
peer2-org1
admin-org1
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./ca/admin
fabric-ca-client register -d --id.name admin-org1 --id.secret org1AdminPW --id.type client -u https://0.0.0.0:7054
fabric-ca-client register -d --id.name peer1-org1 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7054
fabric-ca-client register -d --id.name peer2-org1 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7054
1
2
3
4
5
颁发ADMIN-ORG1的MSP证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp
export FABRIC_CA_CLIENT_HOME=./admin
fabric-ca-client enroll -d -u https://admin-org1:org1AdminPW@0.0.0.0:7054
1
2
3
4
由于高版本的fabric开启了ou分类，admin的msp下需要添加一个ou分类文件config.yaml。

编写config.yaml文件

NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/0-0-0-0-7054.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/0-0-0-0-7054.pem
    OrganizationalUnitIdentifier: peer
  AdminOUIdentifier:
    Certificate: cacerts/0-0-0-0-7054.pem
    OrganizationalUnitIdentifier: admin
  OrdererOUIdentifier:
    Certificate: cacerts/0-0-0-0-7054.pem
    OrganizationalUnitIdentifier: orderer
1
2
3
4
5
6
7
8
9
10
11
12
13
14
颁发PEER1-ORG1的MSP证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp
export FABRIC_CA_CLIENT_HOME=./peer1
fabric-ca-client enroll -d -u https://peer1-org1:peer1PW@0.0.0.0:7054
#拷贝证书
mkdir peer1/msp/admincerts
cp admin/msp/signcerts/cert.pem peer1/msp/admincerts
1
2
3
4
5
6
7
peer1的msp下添加ou分类文件config.yaml:

NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/0-0-0-0-7054.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/0-0-0-0-7054.pem
    OrganizationalUnitIdentifier: peer
1
2
3
4
5
6
7
8
颁发PEER1-ORG1的TLS证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/tls-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_HOME=./peer1
fabric-ca-client enroll -d -u https://peer1-org1:peer1PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer1-org1
# 重命名
mv peer1/tls-msp/keystore/ca9008081c786e68842e150743017acbaa667d69bb6a03e9ba4b60c0e0e273ca_sk peer1/tls-msp/keystore/key.pem
1
2
3
4
5
6
颁发PEER2-ORG1的MSP证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp
export FABRIC_CA_CLIENT_HOME=./peer2
fabric-ca-client enroll -d -u https://peer2-org1:peer2PW@0.0.0.0:7054
#拷贝证书
mkdir peer2/msp/admincerts
cp admin/msp/signcerts/cert.pem peer2/msp/admincerts
1
2
3
4
5
6
7
peer2的msp下添加ou分类文件config.yaml:

NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/0-0-0-0-7054.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/0-0-0-0-7054.pem
    OrganizationalUnitIdentifier: peer
1
2
3
4
5
6
7
8
颁发PEER2-ORG1的TLS证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/tls-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_HOME=./peer2
fabric-ca-client enroll -d -u https://peer2-org1:peer2PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer2-org1
# 重命名
mv peer2/tls-msp/keystore/ca4026081c786e68842e150743017acbaa667d69bb6a03e9ba4b60c0e0e273ca_sk peer2/tls-msp/keystore/key.pem
1
2
3
4
5
6
颁发ORG1组织的MSP
mkdir msp
mkdir msp/admincerts
# 拷贝组织管理员证书下的signcerts/cert.pem证书
cp admin/msp/signcerts/cert.pem msp/admincerts/cert.pem
mkdir msp/cacerts
# 拷贝该组织ca的ca-cert.pem证书
cp /tmp/hyperledger/org1/ca/crypto/ca-cert.pem  msp/cacerts/cert.pem
mkdir msp/tlscacerts
# 拷贝tls-ca的ca-cert.pem证书
cp /tmp/hyperledger/tls-ca/crypto/ca-cert.pem msp/tlscacerts/cert.pem
1
2
3
4
5
6
7
8
9
10
org1组织msp同样需要ou文件，config.yaml:

NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/cert.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/cert.pem
    OrganizationalUnitIdentifier: peer
1
2
3
4
5
6
7
8
完整目录文件如下：

.
├── admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── IssuerPublicKey
│       ├── IssuerRevocationPublicKey
│       ├── cacerts
│       │   └── 0-0-0-0-7054.pem
│       ├── config.yaml
│       ├── keystore
│       │   └── f21a48f340f63306e52a0d7279f1170fa62bf0dbd8813b0434d5124d8cc3fa27_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── ca
│   └── admin
│       ├── fabric-ca-client-config.yaml
│       └── msp
│           ├── IssuerPublicKey
│           ├── IssuerRevocationPublicKey
│           ├── cacerts
│           │   └── 0-0-0-0-7054.pem
│           ├── keystore
│           │   └── d0b996c220c53215d205650d83ec63e7c68dd5cea3714da14681099bcbd333fd_sk
│           ├── signcerts
│           │   └── cert.pem
│           └── user
├── msp
│   ├── admincerts
│   │   └── cert.pem
│   ├── cacerts
│   │   └── cert.pem
│   ├── config.yaml
│   └── tlscacerts
│       └── cert.pem
├── peer1
│   ├── fabric-ca-client-config.yaml
│   ├── msp
│   │   ├── IssuerPublicKey
│   │   ├── IssuerRevocationPublicKey
│   │   ├── admincerts
│   │   │   └── cert.pem
│   │   ├── cacerts
│   │   │   └── 0-0-0-0-7054.pem
│   │   ├── config.yaml
│   │   ├── keystore
│   │   │   └── 20cb422f931ee2fbffec4dd16fb3d298bc561a9b2192fa7f1364859480548be1_sk
│   │   ├── signcerts
│   │   │   └── cert.pem
│   │   └── user
│   └── tls-msp
│       ├── IssuerPublicKey
│       ├── IssuerRevocationPublicKey
│       ├── cacerts
│       ├── keystore
│       │   └── key.pem
│       ├── signcerts
│       │   └── cert.pem
│       ├── tlscacerts
│       │   └── tls-0-0-0-0-7052.pem
│       └── user
└── peer2
    ├── fabric-ca-client-config.yaml
    ├── msp
    │   ├── IssuerPublicKey
    │   ├── IssuerRevocationPublicKey
    │   ├── admincerts
    │   │   └── cert.pem
    │   ├── cacerts
    │   │   └── 0-0-0-0-7054.pem
    │   ├── config.yaml
    │   ├── keystore
    │   │   └── 7f2713aa0b8a5b4cc34093a6399cf2603e93443656525d344cb3e0cdc5546f79_sk
    │   ├── signcerts
    │   │   └── cert.pem
    │   └── user
    └── tls-msp
        ├── IssuerPublicKey
        ├── IssuerRevocationPublicKey
        ├── cacerts
        ├── keystore
        │   └── key.pem
        ├── signcerts
        │   └── cert.pem
        ├── tlscacerts
        │   └── tls-0-0-0-0-7052.pem
        └── user

43 directories, 43 files
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
启动ORG2-CA
编写rca-org2.yaml文件

version: '2'

networks:
  fabric-ca:
    driver: bridge

services:
  rca-org2:
    container_name: rca-org2
    image: hyperledger/fabric-ca:1.4.0
    command: sh -c 'fabric-ca-server start -d -b rca-org2-admin:rca-org2-adminpw --port 7055'
    environment:
    - FABRIC_CA_SERVER_HOME=/tmp/hyperledger/fabric-ca/crypto
    - FABRIC_CA_SERVER_TLS_ENABLED=true
    - FABRIC_CA_SERVER_CSR_CN=rca-org2
    - FABRIC_CA_SERVER_CSR_HOSTS=0.0.0.0
    - FABRIC_CA_SERVER_DEBUG=true
    volumes:
    - /tmp/hyperledger/org2/ca:/tmp/hyperledger/fabric-ca
    networks:
    - fabric-ca
    ports:
    - 7055:7055
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
启动rca-org2

docker-compose -f rca-org1.yaml up
1
颁发RCA-ORG2 CA的管理员证书
cd ..
mkdir org2 && cd $_
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp
export FABRIC_CA_CLIENT_HOME=./ca/admin
fabric-ca-client enroll -d -u https://rca-org2-admin:rca-org2-adminpw@0.0.0.0:7055
1
2
3
4
5
6
注册以下账号：

peer1-org1
peer2-org1
admin-org1
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_HOME=./ca/admin
fabric-ca-client register -d --id.name admin-org2 --id.secret org2AdminPW --id.type client -u https://0.0.0.0:7055
fabric-ca-client register -d --id.name peer1-org2 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7055
fabric-ca-client register -d --id.name peer2-org2 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7055
1
2
3
4
5
颁发ADMIN-ORG1的MSP证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp
export FABRIC_CA_CLIENT_HOME=./admin
fabric-ca-client enroll -d -u https://admin-org2:org2AdminPW@0.0.0.0:7055
1
2
3
4
同样的，admin的msp下需要一个ou分类文件config.yaml：

NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/0-0-0-0-7055.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/0-0-0-0-7055.pem
    OrganizationalUnitIdentifier: peer
  AdminOUIdentifier:
    Certificate: cacerts/0-0-0-0-7055.pem
    OrganizationalUnitIdentifier: admin
  OrdererOUIdentifier:
    Certificate: cacerts/0-0-0-0-7055.pem
    OrganizationalUnitIdentifier: orderer
1
2
3
4
5
6
7
8
9
10
11
12
13
14
颁发PEER1-ORG2的MSP证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp
export FABRIC_CA_CLIENT_HOME=./peer1
fabric-ca-client enroll -d -u https://peer1-org2:peer1PW@0.0.0.0:7055
#拷贝证书
mkdir peer1/msp/admincerts
cp admin/msp/signcerts/cert.pem peer1/msp/admincerts
1
2
3
4
5
6
7
peer1的msp下添加ou分类文件config.yaml:

NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/0-0-0-0-7055.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/0-0-0-0-7055.pem
    OrganizationalUnitIdentifier: peer
1
2
3
4
5
6
7
8
颁发PEER1-ORG2的TLS证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/tls-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_HOME=./peer1
fabric-ca-client enroll -d -u https://peer1-org2:peer1PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer1-org2
# 重命名
mv peer1/tls-msp/keystore/adb7e0d72fd69df337f9d380e674e1c884f2f5ddb48d562fba52e7709c17adb2_sk peer1/tls-msp/keystore/key.pem
1
2
3
4
5
6
颁发PEER2-ORG2的MSP证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp
export FABRIC_CA_CLIENT_HOME=./peer2
fabric-ca-client enroll -d -u https://peer2-org2:peer2PW@0.0.0.0:7055
#拷贝证书
mkdir peer2/msp/admincerts
cp admin/msp/signcerts/cert.pem peer2/msp/admincerts
1
2
3
4
5
6
7
peer2的msp下添加ou分类文件config.yaml:

NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/0-0-0-0-7055.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/0-0-0-0-7055.pem
    OrganizationalUnitIdentifier: peer
1
2
3
4
5
6
7
8
颁发PEER2-ORG2的TLS证书
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/tls-ca/crypto/ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_HOME=./peer2
fabric-ca-client enroll -d -u https://peer2-org2:peer2PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer2-org2
# 重命名
mv peer2/tls-msp/keystore/ed9f1514f22deb03b15ffbb5c8ccc8997359373cbe29b82544d4cf3148e2c488_sk peer2/tls-msp/keystore/key.pem
1
2
3
4
5
6
颁发ORG2组织的MSP
mkdir msp
mkdir msp/admincerts
# 拷贝组织管理员证书下的signcerts/cert.pem证书
cp admin/msp/signcerts/cert.pem msp/admincerts/cert.pem
mkdir msp/cacerts
# 拷贝该组织ca的ca-cert.pem证书
cp /tmp/hyperledger/org2/ca/crypto/ca-cert.pem  msp/cacerts/cert.pem
mkdir msp/tlscacerts
# 拷贝tls-ca的ca-cert.pem证书
cp /tmp/hyperledger/tls-ca/crypto/ca-cert.pem msp/tlscacerts/cert.pem
1
2
3
4
5
6
7
8
9
10
org2组织msp同样需要ou文件，config.yaml:

NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/cert.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/cert.pem
    OrganizationalUnitIdentifier: peer
1
2
3
4
5
6
7
8
完整目录文件如下：

.
├── admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── IssuerPublicKey
│       ├── IssuerRevocationPublicKey
│       ├── cacerts
│       │   └── 0-0-0-0-7055.pem
│       ├── config.yaml
│       ├── keystore
│       │   └── 44f05d0f59f434fc8fcb7d50a1059b9235ba05d954ee7386974a89f9336ba3e4_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── ca
│   └── admin
│       ├── fabric-ca-client-config.yaml
│       └── msp
│           ├── IssuerPublicKey
│           ├── IssuerRevocationPublicKey
│           ├── cacerts
│           │   └── 0-0-0-0-7055.pem
│           ├── keystore
│           │   └── 88654d74808c929c79b8585985b32ea2df69630013ffd561b45a182a4c85cdee_sk
│           ├── signcerts
│           │   └── cert.pem
│           └── user
├── msp
│   ├── admincerts
│   │   └── cert.pem
│   ├── cacerts
│   │   └── cert.pem
│   ├── config.yaml
│   └── tlscacerts
│       └── cert.pem
├── peer1
│   ├── fabric-ca-client-config.yaml
│   ├── msp
│   │   ├── IssuerPublicKey
│   │   ├── IssuerRevocationPublicKey
│   │   ├── admincerts
│   │   │   └── cert.pem
│   │   ├── cacerts
│   │   │   └── 0-0-0-0-7055.pem
│   │   ├── config.yaml
│   │   ├── keystore
│   │   │   └── 5538a2a0e8b8bea523a90ca8ab2f49e6b630e7232907316c578b5c2d515dcd2a_sk
│   │   ├── signcerts
│   │   │   └── cert.pem
│   │   └── user
│   └── tls-msp
│       ├── IssuerPublicKey
│       ├── IssuerRevocationPublicKey
│       ├── cacerts
│       ├── keystore
│       │   └── key.pem
│       ├── signcerts
│       │   └── cert.pem
│       ├── tlscacerts
│       │   └── tls-0-0-0-0-7052.pem
│       └── user
└── peer2
    ├── fabric-ca-client-config.yaml
    ├── msp
    │   ├── IssuerPublicKey
    │   ├── IssuerRevocationPublicKey
    │   ├── admincerts
    │   │   └── cert.pem
    │   ├── cacerts
    │   │   └── 0-0-0-0-7055.pem
    │   ├── config.yaml
    │   ├── keystore
    │   │   └── 0272553b73e8280b17596be21f5ea9a09089b79428eb448f070f7f4e5d4aae03_sk
    │   ├── signcerts
    │   │   └── cert.pem
    │   └── user
    └── tls-msp
        ├── IssuerPublicKey
        ├── IssuerRevocationPublicKey
        ├── cacerts
        ├── keystore
        │   └── key.pem
        ├── signcerts
        │   └── cert.pem
        ├── tlscacerts
        │   └── tls-0-0-0-0-7052.pem
        └── user

43 directories, 43 files
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
启动FABRIC网络组件
生成创世区块及通道文件
configtx.yaml

# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

---

Organizations:
- &OrdererOrg
  Name: OrdererOrg
  ID: OrdererMSP
  MSPDir: ./orderer/msp
  Policies:
    Readers:
      Type: Signature
      Rule: "OR('OrdererMSP.member')"
    Writers:
      Type: Signature
      Rule: "OR('OrdererMSP.member')"
    Admins:
      Type: Signature
      Rule: "OR('OrdererMSP.admin')"

- &org1
  Name: org1MSP
  ID: org1MSP
  MSPDir: ./org1/msp
  Policies:
    Readers:
      Type: Signature
      Rule: "OR('org1MSP.admin', 'org1MSP.peer', 'org1MSP.client')"
    Writers:
      Type: Signature
      Rule: "OR('org1MSP.admin', 'org1MSP.client')"
    Admins:
      Type: Signature
      Rule: "OR('org1MSP.admin')"

  AnchorPeers:
  - Host: peer1-org1
    Port: 7051
- &org2
  Name: org2MSP
  ID: org2MSP
  MSPDir: ./org2/msp
  Policies:
    Readers:
      Type: Signature
      Rule: "OR('org2MSP.admin', 'org2MSP.peer', 'org2MSP.client')"
    Writers:
      Type: Signature
      Rule: "OR('org2MSP.admin', 'org2MSP.client')"
    Admins:
      Type: Signature
      Rule: "OR('org2MSP.admin')"

  AnchorPeers:
  - Host: peer1-org2
    Port: 7051
Capabilities:
    Global: &ChannelCapabilities
        V1_3: true
    Orderer: &OrdererCapabilities
        V1_1: true
    Application: &ApplicationCapabilities
        V1_3: true
        V1_2: false
        V1_1: false
Application: &ApplicationDefaults
    Organizations:
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

    Capabilities:
          <<: *ApplicationCapabilities
Orderer: &OrdererDefaults
    OrdererType: solo
    Addresses:
        - orderer1-orderer:7050
    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB

    Kafka:
        Brokers:
            - 1kafka0:8013
            - 1kafka1:8014
            - 1kafka2:8015
            - 1kafka3:8016
    Organizations:
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
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"
    Capabilities:
          <<: *OrdererCapabilities
Channel: &ChannelDefaults
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: "ImplicitMeta"
            Rule: "MAJORITY Admins"
    Capabilities:
          <<: *ChannelCapabilities
Profiles:
    OrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *org1
                    - *org2
    OrgsChannel:
        Consortium: SampleConsortium
        <<: *ChannelDefaults
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *org1
                - *org2
            Capabilities:
                <<: *ApplicationCapabilities

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
151
152
153
生成创世区块

configtxgen -outputBlock orderer/orderer1/genesis.block -profile OrgsOrdererGenesis --configPath=./
1
生成mychannel.tx

configtxgen -profile OrgsChannel --configPath=./ -outputCreateChannelTx mychannel.tx -channelID mychannel
1
复制mychannel.tx

cp mychannel.tx org1/peer1
cp mychannel.tx org2/peer1
1
2
启动网络组件
启动ORDERER
orderer1.yaml

version: '2'
networks:
  fabric-ca:
    driver: bridge

services:
  orderer1-org0:
    container_name: orderer1-orderer
    image: hyperledger/fabric-orderer
    environment:
    - ORDERER_HOME=/tmp/hyperledger/orderer:1.4.0
    - ORDERER_HOST=orderer1-orderer
    - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
    - ORDERER_GENERAL_GENESISMETHOD=file
    - ORDERER_GENERAL_GENESISFILE=/tmp/hyperledger/orderer/orderer/genesis.block
    - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
    - ORDERER_GENERAL_LOCALMSPDIR=/tmp/hyperledger/orderer/orderer/msp
    - ORDERER_GENERAL_TLS_ENABLED=true
    - ORDERER_GENERAL_TLS_CERTIFICATE=/tmp/hyperledger/orderer/orderer/tls-msp/signcerts/cert.pem
    - ORDERER_GENERAL_TLS_PRIVATEKEY=/tmp/hyperledger/orderer/orderer/tls-msp/keystore/key.pem
    - ORDERER_GENERAL_TLS_ROOTCAS=[/tmp/hyperledger/orderer/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem]
    - ORDERER_GENERAL_LOGLEVEL=debug
    - ORDERER_DEBUG_BROADCASTTRACEDIR=data/logs
    volumes:
    - /Users/finefine/fabric-ca-tls/orderer/orderer1:/tmp/hyperledger/orderer/orderer/
    networks:
    - fabric-ca

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
#启动orderer
docker-compose -f orderer1.yaml up -d
1
2
启动ORG1
peer1-org1

version: '2'

networks:
  fabric-ca:
    driver: bridge

services:
  peer1-org1:
    container_name: peer1-org1
    image: hyperledger/fabric-peer:1.4.0
    environment:
    - CORE_PEER_ID=peer1-org1
    - CORE_PEER_ADDRESS=peer1-org1:7051
    - CORE_PEER_LOCALMSPID=org1MSP
    - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/peer1/msp
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=fabric-ca-tls_fabric-ca
    - FABRIC_LOGGING_SPEC=debug
    - CORE_PEER_TLS_ENABLED=true
    - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org1/peer1/tls-msp/signcerts/cert.pem
    - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org1/peer1/tls-msp/keystore/key.pem
    - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org1/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
    - CORE_PEER_GOSSIP_USELEADERELECTION=true
    - CORE_PEER_GOSSIP_ORGLEADER=false
    - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1-org1:7051
    - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org1/peer1
    volumes:
    - /var/run:/host/var/run
    - /Users/finefine/fabric-ca-tls/org1/peer1:/tmp/hyperledger/org1/peer1
    networks:
    - fabric-ca
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
# 启动peer1-org1
docker-compose -f peer1-org1.yaml up -d
1
2
peer2-org1

version: '2'

networks:
  fabric-ca:
    driver: bridge

services:
  peer2-org1:
    container_name: peer2-org1
    image: hyperledger/fabric-peer
    environment:
    - CORE_PEER_ID=peer2-org1
    - CORE_PEER_ADDRESS=peer2-org1:7051
    - CORE_PEER_LOCALMSPID=org1MSP
    - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/peer2/msp
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=fabric-ca-tls_fabric-ca
    - FABRIC_LOGGING_SPEC=debug
    - CORE_PEER_TLS_ENABLED=true
    - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org1/peer2/tls-msp/signcerts/cert.pem
    - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org1/peer2/tls-msp/keystore/key.pem
    - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org1/peer2/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
    - CORE_PEER_GOSSIP_USELEADERELECTION=true
    - CORE_PEER_GOSSIP_ORGLEADER=false
    - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer2-org1:7051
    - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org1/peer2
    volumes:
    - /var/run:/host/var/run
    - /Users/finefine/fabric-ca-tls/org1/peer2:/tmp/hyperledger/org1/peer2
    networks:
    - fabric-ca
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
# 启动peer2-org1
docker-compose -f peer2-org1.yaml up -d
1
2
启动ORG2
peer1-org2

version: '2'

networks:
  fabric-ca:
    driver: bridge

services:
  peer1-org2:
    container_name: peer1-org2
    image: hyperledger/fabric-peer
    environment:
    - CORE_PEER_ID=peer1-org2
    - CORE_PEER_ADDRESS=peer1-org2:7051
    - CORE_PEER_LOCALMSPID=org2MSP
    - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org2/peer1/msp
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=fabric-ca-tls_fabric-ca
    - FABRIC_LOGGING_SPEC=debug
    - CORE_PEER_TLS_ENABLED=true
    - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org2/peer1/tls-msp/signcerts/cert.pem
    - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org2/peer1/tls-msp/keystore/key.pem
    - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
    - CORE_PEER_GOSSIP_USELEADERELECTION=true
    - CORE_PEER_GOSSIP_ORGLEADER=false
    - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1-org2:7051
    - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org2/peer1
    volumes:
    - /var/run:/host/var/run
    - /Users/finefine/fabric-ca-tls/org2/peer1:/tmp/hyperledger/org2/peer1
    networks:
    - fabric-ca

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
# 启动peer1-org2
docker-compose -f peer1-org2.yaml up -d
1
2
peer1-org2

version: '2'

networks:
  fabric-ca:
    driver: bridge

services:
  peer2-org2:
    container_name: peer2-org2
    image: hyperledger/fabric-peer
    environment:
    - CORE_PEER_ID=peer2-org2
    - CORE_PEER_ADDRESS=peer2-org2:7051
    - CORE_PEER_LOCALMSPID=org2MSP
    - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org2/peer2/msp
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=fabric-ca-tls_fabric-ca
    - FABRIC_LOGGING_SPEC=debug
    - CORE_PEER_TLS_ENABLED=true
    - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org2/peer2/tls-msp/signcerts/cert.pem
    - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org2/peer2/tls-msp/keystore/key.pem
    - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org2/peer2/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
    - CORE_PEER_GOSSIP_USELEADERELECTION=true
    - CORE_PEER_GOSSIP_ORGLEADER=false
    - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer2-org2:7051
    - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org2/peer2
    volumes:
    - /var/run:/host/var/run
    - /Users/finefine/fabric-ca-tls/org2/peer2:/tmp/hyperledger/org2/peer2
    networks:
    - fabric-ca
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
# 启动peer2-org2
docker-compose -f peer2-org2.yaml up -d
1
2
cli-org2

version: '2'
networks:
  fabric-ca:
    driver: bridge

services:
  cli-org1:
   container_name: cli-org2
   image: hyperledger/fabric-tools
   tty: true
   stdin_open: true
   environment:
   - GOPATH=/opt/gopath
   - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
   - FABRIC_LOGGING_SPEC=DEBUG
   - CORE_PEER_ID=cli-org2
   - CORE_PEER_ADDRESS=peer1-org2:7051
   - CORE_PEER_LOCALMSPID=org2MSP
   - CORE_PEER_TLS_ENABLED=true
   - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
   - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org2/admin/msp
   working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org2
   command: sh
   volumes:
   - /Users/finefine/fabric-ca-tls/org2/peer1:/tmp/hyperledger/org2/peer1
   - /Users/finefine/fabric-ca-tls/chaincode:/opt/gopath/src/github.com/hyperledger/fabric-samples/chaincode
   - /Users/finefine/fabric-ca-tls/org2/admin:/tmp/hyperledger/org2/admin
   networks:
     - fabric-ca
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
docker-compose -f cli-org2.yaml up -d
1
创建通道及加入通道
创建通道

# 进入cli-org1容器内
docker exec -it cli-org1 bash
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp
peer channel create -c mychannel -f /tmp/hyperledger/org1/peer1/mychannel.tx -o orderer1-orderer:7050 --outputBlock /tmp/hyperledger/org1/peer1/mychannel.block --tls --cafile /tmp/hyperledger/org1/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
exit
# 复制
cp org1/peer1/mychannel.block org2/peer1/mychannel.block
1
2
3
4
5
6
7
加入通道

# 进入 cli-org1 容器
docker exec -it cli-org1 bash

export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp

export CORE_PEER_ADDRESS=peer1-org1:7051
peer channel join -b /tmp/hyperledger/org1/peer1/mychannel.block

export CORE_PEER_ADDRESS=peer2-org1:7051
peer channel join -b /tmp/hyperledger/org1/peer1/mychannel.block
# 退出cli-org1容器
exit
# 进入cli-org2容器
docker exec -it cli-org2 bash

export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp

export CORE_PEER_ADDRESS=peer1-org2:7051
peer channel join -b /tmp/hyperledger/org2/peer1/mychannel.block

export CORE_PEER_ADDRESS=peer2-org2:7051
peer channel join -b /tmp/hyperledger/org2/peer1/mychannel.block
exit
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
安装CHAINCODE并初始化
安装
org1

docker exec -it cli-org1 bash

# peer1-org1
export CORE_PEER_ADDRESS=peer1-org1:7051
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric-samples/chaincode/go/chaincode_example02

# peer2-org1
export CORE_PEER_ADDRESS=peer2-org1:7051
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric-samples/chaincode/go/chaincode_example02
1
2
3
4
5
6
7
8
9
10
org2

docker exec -it cli-org2 bash

# peer1-org2
export CORE_PEER_ADDRESS=peer1-org2:7051
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric-samples/chaincode/go/chaincode_example02

# peer2-org2
export CORE_PEER_ADDRESS=peer2-org2:7051
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric-samples/chaincode/go/chaincode_example02
1
2
3
4
5
6
7
8
9
10
初始化
docker exec -it cli-org2 bash

peer chaincode instantiate -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -o orderer1-orderer:7050 --tls --cafile /tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem

1
2
3
4
查询和调用
cli-org1

docker exec -it cli-org1 bash

export CORE_PEER_ADDRESS=peer1-org1:7051
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp
# 查询结果应该为100
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
1
2
3
4
5
6
cli-org2

docker exec -it cli-org2 bash

export CORE_PEER_ADDRESS=peer1-org2:7051
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org2/admin/msp

# a转账给b 10
peer chaincode invoke -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}' --tls --cafile /tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
# a的查询结果应该为90
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'