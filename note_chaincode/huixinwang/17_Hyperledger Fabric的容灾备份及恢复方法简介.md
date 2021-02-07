Hyperledger Fabric日益增强的潜力使得许多企业正在尝试使用fabric。当使用涉及更多peers和orderers的大型网络时，维护超级账本数据的备份非常重要。如果网络出现故障，这将有所帮助。它还有助于开发阶段，因为可以使用备份数据来执行将来的测试。

在本文中，我将介绍如何进行备份以及如何在Hyperledger Fabric中使用备份。

超级账本分类帐数据在容器中的位置
通常，peer中的分类帐数据存储在/var/hyperledger/production/location中。在orderer中，它位于/var/hyperledger/production/orderer中。我们需要备份这些文件夹。

采取备份的步骤
第1步：
要进行备份，必须创建空间来存储数据。当网络在Docker容器内运行时，我们将使用卷来实现这一点。我将使用一个基本网络与一个peer，一个orderer，一个ca和一个CouchDB。由于我们有一个peer和一个orderer，我们需要创建两个卷，比如backup_orderer和backup_peer。我们需要在docker-compose.yml文件中将这些指定为卷，如下所示。

networks:
  basic:

volumes:
  backup_peer:
  backup_orderer:
这将在启动网络时创建两个卷。

第2步：
下一步是将这些卷安装到容器中。对于peer，我们可以定义卷，如下所示。

volumes:
	- backup_peer:/var/hyperledger/production
对于orderer，我们可以定义如下所示的数量。

volumes:
	- backup_orderer:/var/hyperledger/production/orderer
现在我们准备通过执行docker-compose来启动网络。我们作为交易的一部分生成的所有数据现在将被复制到卷。可以使用云将数据存储在卷中。

Peer备份Underhood
在同行中，超级账本数据存储在/var/hyperledger/production中。production文件夹有三个子文件夹，即chaincodes，ledgersData，transientStore，在ledgersData中我们有六个文件夹，分别是bookkeeper，chains，configHistory，historyLeveldb，ledgerProvider，pvtdataStore。在chains中，我们有另外两个文件夹，即chains, index。chains文件夹包含所有通道数据，一个带有通道名称的文件夹和该通道的完整区块链（文件blockfile_000000）。

Orderer备份Underhood
在orderer中，超级账本数据存储在/var/hyperledger/production中。production有orderer文件夹。orderer有两个文件夹，即chain，index。chains具有名称为channel和testchainid的文件夹。所有文件夹都有blockfile_000000。

testchainid中的blockfile_000000包含通道的所有详细信息。其他文件夹中的其余blockfile_000000处理通道的超级账本数据。

使用备份数据的步骤
现在我们有了备份数据。你可以在机器中的/var/lib/docker/volumes中找到卷，如果存储在那里，则可以在云中找到卷。将卷的结束点挂载到docker-compose.yml中的peers和orderers的/var/hyperledger/production。

通过执行docker-compose文件启动网络。在装入卷时，超级账本数据将自动复制到新网络。无需创建通道并加入peers，因为所有必需的数据都被复制到容器中的相应文件夹中。通过执行查询来检查数据是否完美。

希望尽快学习的请访问Fabric区块链开发详解，本课程面向初学者，内容即包含Hyperledger Fabric的身份证书与MSP服务、权限策略、通道配置与启动、链码通信接口等核心概念，也包含Fabric网络设计、nodejs链码与应用开发的操作实践，是Nodejs工程师学习Fabric区块链开发的最佳选择。