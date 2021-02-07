configtx.yaml是Hyperledger Fabric区块链网络运维工具configtxgen用于 生成通道创世块或通道交易的配置文件，configtx.yaml的内容直接决定了所生成 的创世区块的内容。本文将给出configtx.yaml的详细中文说明。

如果需要快速掌握Fabric区块链的链码与应用开发，推荐访问汇智网的在线互动教程：

Fabric区块链Java开发详解
Fabric区块链NodeJS开发详解
Capabilities / 通道能力配置
Capabilities段用来定义fabric网络的能力。这是版本v1.0.0引入的一个新的配置段， 当与版本v1.0.x的对等节点与排序节点混合组网时不可使用。

Capabilities段定义了fabric程序要加入网络所必须支持的特性。例如，如果添加了一个新 的MSP类型，那么更新的程序可能会根据该类型识别并验证签名，但是老版本的程序就 没有办法验证这些交易。这可能导致不同版本的fabric程序中维护的世界状态不一致。

因此，通过定义通道的能力，就明确了不满足该能力要求的fabric程序，将无法处理 交易，除非升级到新的版本。对于v1.0.x的程序而言，如果在Capabilities段定义了 任何能力，即使声明不需要支持这些能力，都会导致其有意崩溃。

Capabilities:
    # Global配置同时应用于排序节点和对等节点，并且必须被两种节点同时支持。
    # 将该配置项设置为ture表明要求节点具备该能力
    Global: &ChannelCapabilities
        V1_3: true

    # Orderer配置仅应用于排序节点，不需考虑对等节点的升级。将该配置项
    # 设置为true表明要求排序节点具备该能力
    Orderer: &OrdererCapabilities
        V1_1: true

    # Application配置仅应用于对等网络，不需考虑排序节点的升级。将该配置项
    # 设置为true表明要求对等节点具备该能力
    Application: &ApplicationCapabilities
        V1_3: true
Organizations / 组织机构配置
Organizations配置段用来定义组织机构实体，以便在后续配置中引用。 例如，下面的配置文件中，定义了三个机构，可以分别使用ExampleCom、 Org1ExampleCom和Org2ExampleCom引用其配置：

Organizations:

    - &ExampleCom
        Name: ExampleCom
        ID: example.com
        AdminPrincipal: Role.ADMIN
        MSPDir: ./ordererOrganizations/example.com/msp
        Policies:
            Readers:
                Type: Signature
                Rule: OR('example.com.member')
            Writers:
                Type: Signature
                Rule: OR('example.com.member')
            Admins:
                Type: Signature
                Rule: OR('example.com.admin')
            Endorsement:
                Type: Signature
                Rule: OR('example.com.member')

    - &Org1ExampleCom
        Name: Org1ExampleCom
        ID: org1.example.com
        MSPDir: ./peerOrganizations/org1.example.com/msp
        AdminPrincipal: Role.ADMIN
        AnchorPeers:
            - Host: peer0.org1.example.com
              Port: 7051
        Policies:
            Readers:
                Type: Signature
                Rule: OR('org1.example.com.member')
            Writers:
                Type: Signature
                Rule: OR('org1.example.com.member')
            Admins:
                Type: Signature
                Rule: OR('org1.example.com.admin')
            Endorsement:
                Type: Signature
                Rule: OR('org1.example.com.member')

    - &Org2ExampleCom
        Name: Org2ExampleCom
        ID: org2.example.com
        MSPDir: ./peerOrganizations/org2.example.com/msp
        AdminPrincipal: Role.ADMIN
        AnchorPeers:
            - Host: peer0.org2.example.com
              Port: 7051
        Policies:
            Readers:
                Type: Signature
                Rule: OR('org2.example.com.member')
            Writers:
                Type: Signature
                Rule: OR('org2.example.com.member')
            Admins:
                Type: Signature
                Rule: OR('org2.example.com.admin')
            Endorsement:
                Type: Signature
                Rule: OR('org2.example.com.member')
Orderer / 排序节点配置
Orderer配置段用来定义要编码写入创世区块或通道交易的排序节点参数。

Orderer: &OrdererDefaults

    # 排序节点类型用来指定要启用的排序节点实现，不同的实现对应不同的共识算法。
    # 目前可用的类型为：solo和kafka
    OrdererType: solo
    Addresses:
        - orderer0.example.com:7050

    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 98 MB
        PreferredMaxBytes: 512 KB

    MaxChannels: 0
    Kafka:
        Brokers:
            - kafka0:9092
            - kafka1:9092
            - kafka2:9092
            - kafka3:9092

    Organizations:

    # 定义本层级的排序节点策略，其权威路径为 /Channel/Orderer/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: ANY Readers
        Writers:
            Type: ImplicitMeta
            Rule: ANY Writers
        Admins:
            Type: ImplicitMeta
            Rule: MAJORITY Admins
        # BlockValidation配置项指定了哪些签名必须包含在区块中，以便对等节点进行验证
        BlockValidation:
            Type: ImplicitMeta
            Rule: ANY Writers

    # Capabilities配置描述排序节点层级的能力需求，这里直接引用
    # 前面Capabilities配置段中的OrdererCapabilities配置项
    Capabilities:
        <<: *OrdererCapabilities
Channel / 通道配置
Channel配置段用来定义要写入创世区块或配置交易的通道参数。

Channel: &ChannelDefaults
    # 定义本层级的通道访问策略，其权威路径为 /Channel/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: ANY Readers
        # Writes策略定义了调用Broadcast API提交交易的许可规则
        Writers:
            Type: ImplicitMeta
            Rule: ANY Writers
        # Admin策略定义了修改本层级配置的许可规则
        Admins:
            Type: ImplicitMeta
            Rule: MAJORITY Admins

    # Capabilities配置描通道层级的能力需求，这里直接引用
    # 前面Capabilities配置段中的ChannelCapabilities配置项
    Capabilities:
        <<: *ChannelCapabilities
Application / 应用配置
Application配置段用来定义要写入创世区块或配置交易的应用参数。

Application: &ApplicationDefaults
    ACLs: &ACLsDefault
        # ACLs配置段为系统中各种资源提供默认的策略。
        # 这里所说的“资源”，可以是系统链码的函数，例如qscc系统链码的GetBlockByNumber方法
        # 也可以是其他资源，例如谁可以接收区块事件。
        # 这个配置段不是用来定义资源或API，而仅仅是定义资源的访问控制策略
        # 
        # 用户可以在通道定义中重写这些默认策略

        #---New Lifecycle System Chaincode (_lifecycle) function to policy mapping for access control--#

        # _lifecycle系统链码CommitChaincodeDefinition函数的ACL定义
        _lifecycle/CommitChaincodeDefinition: /Channel/Application/Writers

        # _lifecycle系统链码的QueryChaincodeDefinition函数的ACL定义
        _lifecycle/QueryChaincodeDefinition: /Channel/Application/Readers

        # _lifecycle系统链码的QueryNamespaceDefinitions函数的ACL定义
        _lifecycle/QueryNamespaceDefinitions: /Channel/Application/Readers

        #---Lifecycle System Chaincode (lscc) function to policy mapping for access control---#

        # lscc系统链码的getid函数的ACL定义
        lscc/ChaincodeExists: /Channel/Application/Readers

        # lscc系统链码的getdepspec函数的ACL定义
        lscc/GetDeploymentSpec: /Channel/Application/Readers

        # lscc系统链码的getccdata函数的ACL定义
        lscc/GetChaincodeData: /Channel/Application/Readers

        # lscc系统链码的getchaincodes函数的ACL定义
        lscc/GetInstantiatedChaincodes: /Channel/Application/Readers

        #---Query System Chaincode (qscc) function to policy mapping for access control---#

        # qscc系统链码的GetChainInfo函数的ACL定义
        qscc/GetChainInfo: /Channel/Application/Readers

        # qscc系统链码的GetBlockByNumber函数的ACL定义
        qscc/GetBlockByNumber: /Channel/Application/Readers

        # qscc系统 链码的GetBlockByHash函数的ACL定义
        qscc/GetBlockByHash: /Channel/Application/Readers

        # qscc系统链码的GetTransactionByID函数的ACL定义
        qscc/GetTransactionByID: /Channel/Application/Readers

        # qscc系统链码GetBlockByTxID函数的ACL定义
        qscc/GetBlockByTxID: /Channel/Application/Readers

        #---Configuration System Chaincode (cscc) function to policy mapping for access control---#

        # cscc系统链码的GetConfigBlock函数的ACl定义
        cscc/GetConfigBlock: /Channel/Application/Readers

        # cscc系统链码的GetConfigTree函数的ACL定义
        cscc/GetConfigTree: /Channel/Application/Readers

        # cscc系统链码的SimulateConfigTreeUpdate函数的ACL定义
        cscc/SimulateConfigTreeUpdate: /Channel/Application/Readers

        #---Miscellanesous peer function to policy mapping for access control---#

        # 访问对等节点上的链码的ACL策略定义
        peer/Propose: /Channel/Application/Writers

        # 从链码中访问其他链码的ACL策略定义
        peer/ChaincodeToChaincode: /Channel/Application/Readers

        #---Events resource to policy mapping for access control###---#

        # 发送区块事件的ACL策略定义
        event/Block: /Channel/Application/Readers

        # 发送过滤的区块事件的ACL策略定义
        event/FilteredBlock: /Channel/Application/Readers

    # Organizations配置列出参与到网络中的机构清单
    Organizations:

    # 定义本层级的应用控制策略，其权威路径为 /Channel/Application/<PolicyName>
    Policies: &ApplicationDefaultPolicies
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        LifecycleEndorsement:
            Type: ImplicitMeta
            Rule: "ANY Endorsement"
        Endorsement:
            Type: ImplicitMeta
            Rule: "ANY Endorsement"

    # Capabilities配置描述应用层级的能力需求，这里直接引用
    # 前面Capabilities配置段中的ApplicationCapabilities配置项
    Capabilities:
        <<: *ApplicationCapabilities
Profiles / 配置入口
Profiles配置段用来定义用于configtxgen工具的配置入口。包含委员会（consortium）的配置入口 可以用来生成排序节点的创世区块。如果在排序节点的创世区块中正确定义了consortium 的成员，那么可以仅使用机构成员名称和委员会的名称来生成通道创建请求。

Profiles:

    # SampleInsecureSolo定义了一个使用Solo排序节点的简单配置
    SampleInsecureSolo:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *ExampleCom
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *ExampleCom
            Capabilities:
                <<: *ApplicationCapabilities
            Policies:
                Readers:
                  Type: ImplicitMeta
                  Rule: ANY Readers
                Writers:
                  Type: ImplicitMeta
                  Rule: ANY Writers
                Admins:
                  Type: ImplicitMeta
                  Rule: MAJORITY Admins
                LifecycleEndorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
                Endorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1ExampleCom
                    - *Org2ExampleCom

    # SampleInsecureKafka定义了一个使用Kfaka排序节点的配置
    SampleInsecureKafka:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            OrdererType: kafka
            Addresses:
                - orderer0.example.com:7050
                - orderer1.example.com:7050
                - orderer2.example.com:7050
            Organizations:
                - *ExampleCom
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *ExampleCom
            Capabilities:
                <<: *ApplicationCapabilities
            Policies:
                Readers:
                  Type: ImplicitMeta
                  Rule: ANY Readers
                Writers:
                  Type: ImplicitMeta
                  Rule: ANY Writers
                Admins:
                  Type: ImplicitMeta
                  Rule: MAJORITY Admins
                LifecycleEndorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
                Endorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *ExampleCom
                    - *Org1ExampleCom
                    - *Org2ExampleCom

    # SampleSingleMSPSolo定义了一个使用Solo排序节点、包含单一MSP的配置
    SampleSingleMSPSolo:
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *ExampleCom
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *ExampleCom
            Capabilities:
                <<: *ApplicationCapabilities
            Policies:
                Readers:
                  Type: ImplicitMeta
                  Rule: ANY Readers
                Writers:
                  Type: ImplicitMeta
                  Rule: ANY Writers
                Admins:
                  Type: ImplicitMeta
                  Rule: MAJORITY Admins
                LifecycleEndorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
                Endorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *ExampleCom
                    - *Org1ExampleCom
                    - *Org2ExampleCom

    # SampleEmptyInsecureChannel定义了一个不包含成员与访问控制策略的通道
    SampleEmptyInsecureChannel:
        Capabilities:
            <<: *ChannelCapabilities
        Consortium: SampleConsortium
        Application:
            Organizations:
                - *ExampleCom
            Capabilities:
                <<: *ApplicationCapabilities
            Policies:
                Readers:
                  Type: ImplicitMeta
                  Rule: ANY Readers
                Writers:
                  Type: ImplicitMeta
                  Rule: ANY Writers
                Admins:
                  Type: ImplicitMeta
                  Rule: MAJORITY Admins
                LifecycleEndorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
                Endorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement

    # SysTestChannel定义了一个用于测试的通道
    SysTestChannel:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1ExampleCom
                - *Org2ExampleCom
            Capabilities:
                <<: *ApplicationCapabilities
            Policies:
                Readers:
                  Type: ImplicitMeta
                  Rule: ANY Readers
                Writers:
                  Type: ImplicitMeta
                  Rule: ANY Writers
                Admins:
                  Type: ImplicitMeta
                  Rule: MAJORITY Admins
                LifecycleEndorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
                Endorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement

    # SampleSingleMSPChannel定义了一个仅包含单一成员机构的通道。
    # 该配置通常与SampleSingleMSPSolo或SampleSingleMSPKafka同时使用
    SampleSingleMSPChannel:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1ExampleCom
                - *Org2ExampleCom
            Capabilities:
                <<: *ApplicationCapabilities
            Policies:
                Readers:
                  Type: ImplicitMeta
                  Rule: ANY Readers
                Writers:
                  Type: ImplicitMeta
                  Rule: ANY Writers
                Admins:
                  Type: ImplicitMeta
                  Rule: MAJORITY Admins
                LifecycleEndorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
                Endorsement:
                  Type: ImplicitMeta
                  Rule: ANY Endorsement
