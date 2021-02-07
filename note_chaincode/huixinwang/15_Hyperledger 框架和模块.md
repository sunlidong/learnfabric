Hyperledger和免许可区块链技术之间的差异。
Hyperledger的框架组件。
Hyperledger框架（Iroha，Sawtooth，Fabric，Indy和Burrow）。
Hyperledger模块（Cello, Explorer, and Composer）。
你可以将Hyperledger视为市场，数据共享网络，微货币和去中心化数字社区的操作系统。

 

Hyperledger和免许可区块链技术之间的差异：



说明：

Sawtooth可以配置为无权限。
关键的Hyperledger共识协议是Hyperledger Sawtooth中的Pooto，Hyperledger Indy中的RBFT，Hyperledger Burrow中的Tendermint，以及Hyperledger Iroha中的又一个共识（YAC）。该结构基于可插拔架构，开发人员可以使用最适合其需求的共识模块配置其部署。初始发布包将提供三种共识实现，供用户选择：
1) No-op（忽略共识）;
2) 经典PBFT;
3) SIEVE（经典PBFT的增强版）。
Hyperledger的框架组件
Hyperledger业务区块链框架用于为组织联盟构建企业区块链。它们与比特币区块链和以太坊等公共分类账不同。Hyperledger框架包括：

仅附加分布式分类帐。
用于同意分类帐更改的一致性算法。
通过许可访问进行交易的隐私。
处理交易请求的智能合约。
Hyperledger框架1：Iroha
Hyperledger Iroha是由Soramitsu，Hitachi，NTT Data和Colu提供的区块链框架。Hyperledger Iroha旨在简单易用地融入需要分布式分类帐技术的基础设施项目中。Hyperledger Iroha强调使用Android和iOS客户端库进行移动应用程序开发，使其与其他Hyperledger框架截然不同。受Hyperledger Fabric的启发，Hyperledger Iroha寻求补充Hyperledger Fabric和Hyperledger Sawtooth，同时为C++开发人员提供开发环境，以便为Hyperledger做出贡献。

总之，Hyperledger Iroha具有简单的构造，现代的，域驱动的C++设计，以及一致的算法YAC。

Hyperledger Framework 2：Sawtooth
由英特尔提供支持的Hyperledger Sawtooth是一个区块链框架，它利用模块化平台构建，部署和运行分布式账本。使用Hyperledger Sawtooth构建的分布式分类帐解决方案可以根据网络规模使用各种一致性算法。默认情况下，它使用消逝时间证明（PoET）一致性算法，该算法在没有高能耗的情况下提供比特币区块链的可扩展性。PoET允许高度可扩展的验证器节点网络。Hyperledger Sawtooth专为多功能性而设计，支持许可和无权限部署。

例如： Sawtooth lake，Sawtooth lake区块链技术可以提供各种商品（如鱼类）的出处和谱系的不可变记录。

Sawtooth的独特特征：

1.Sawtooth的设计使你可以扩大网络规模。
2.你实际上可以动态更改共识机制。
Hyperledger Framework 3：Fabric
Hyperledger Fabric是第一个代码库提案，它结合了Digital Asset Holdings，Blockstream的libconsensus和IBM的OpenBlockchain之前的工作。Hyperledger Fabric提供模块化架构，允许共识和会员服务等组件即插即用。Hyperledger Fabric在允许实体进行机密交易而不通过中央机构传递信息方面具有革命性。这是通过在网络内运行的不同频道以及表征网络中不同节点的分工来实现的。最后，重要的是要记住，与比特币（公共链）不同，Hyperledger Fabric支持许可部署。

“如果你有一个大的区块链网络，并且你想只与某些方共享数据，你可以创建一个只有那些参与者的私人频道。这是Fabric现在最独特的事情。“ ——Brian Behlendorf，Hyperledger执行董事，Linux基金会。

Hyperledger Framework 4：Indy
Hyperledger Indy是一种专为去中心化身份而设计的分布式分类账。Hyperledger Indy的目标是通过开发一组独立于任何特定分类账的去中心化身份规范和工件来实现这一目标，并实现跨任何支持它们的DLT的互操作性。

实际上，自2013年以来，已有超过90亿条数据记录丢失或被盗。令人惊讶的是，其中只有4％是加密的，因此在被盗后变得毫无安全性（也称为“安全漏洞”）。你可以在这里找到详细的统计。

Hyperledger Indy的一个关键原则是其“设计隐私”方法。鉴于DLT的不变性，更重要的是要极其谨慎地处理数字身份，保持人类价值观的前沿和中心。

“Hyperledger Indy允许用户根据他们愿意存储和共享的属性来验证身份。这可以减少业务中包含的责任金额，因为数据可以与用户保持在一起，并以你可以信任的方式再次呈现给你，并验证所说的内容是真实的，并且受到其他方的信任做生意。” —— Nathan George，Maintainer，Hyperledger Indy。

贡献者是：Sovrin基金会

Hyperledger Framework 5：Burrow
Hyperledger Burrow目前处于孵化阶段，是允许智能合约机，它为模块化区块链客户端提供了一个内置于以太坊虚拟机（EVM）规范的许可智能合约解释器。它是唯一可用的Apache许可的EVM实现。

以下是Burrow的主要组成部分：

Gateway为系统集成和用户界面提供接口。
智能合约应用程序引擎有助于集成复杂的业务逻辑。
Consensus Engine具有以下双重目的：
a. 维护节点之间的网络堆栈。
b. 订单交易。
应用程序区块链接口（ABCI）为共识引擎和智能合约应用程序引擎提供连接的接口规范。你可以在此处了解有关Hyperledger Burrow的更多信息。

Recap
Hyperledger Frameworks
1. Iroha- A simple and easy to incorporate into infrastructure projects requiring distributed ledger technology
2. Sawtooth- A modular blockchain framework that implements the PoET consensus algorithm
3. Fabric- Modular Blockchain with private channels
4. Indy — De-centralised Identity
5. Burrow — Smart Contract Machine
超级模块
Hyperledger模块是辅助软件，用于部署和维护区块链，检查分类账上的数据，以及设计，原型和扩展区块链网络的工具。

Hyperledger模块1：大提琴
对于想要部署Blockchain-as-a-Service的企业，Hyperledger Cello提供了满足这一需求的工具包。作为Hyperledger模块，“Cello旨在将按需服务部署模型引入区块链生态系统”，从而有助于进一步开发和部署Hyperledger的框架。Hyperledger Cello最初由IBM提供，赞助商来自Soramitsu，华为和英特尔。通过Cello，您可以构建Blockchain-as-a-Service（BaaS）平台。



Hyperledger模块2：Hyperledger Explorer
Hyperledger Explorer是一种可视化区块链操作的工具。它是有史以来第一个获得许可分类账的区块链资源管理器，允许任何人探索Hyperledger成员从内部创建的分布式分类帐项目，而不会影响他们的隐私。该项目由DTCC，英特尔和IBM提供。

Hyperledger Explorer旨在创建用户友好的Web应用程序，可以查看，调用，部署或查询：

块。
交易和相关数据。
网络信息（名称，状态，节点列表）。
智能合约（连锁代码和交易系列）。
存储在分类帐中的其他相关信息。
数据可视化的能力至关重要，以便从中提取业务价值。Hyperledger Explorer提供了这项急需的功能。关键组件包括Web服务器，Web UI，Web套接字，数据库，安全性存储库和区块链实现。

Hyperledger模块3：Hyperledger Composer
Hyperledger Composer提供了一套用于构建区块链业务网络的工具。这些工具允许你：

为你的业务区块链网络建模。
生成REST API以与区块链网络进行交互。
生成 skeleton Angular 应用程序。
Hyperledger Composer内置Javascript（yaey！），提供易于使用的一组组件，开发人员可以快速学习和实现这些组件。该项目由Oxchains和IBM提供。

Hyperledger Composer的好处是：

更快地创建区块链应用程序，消除了从头开始构建区块链应用程序所需的大量工作。
通过经过充分测试的高效设计降低风险，使业务和技术分析师之间的理解保持一致。
更高的灵活性，因为更高级别的抽象使迭代变得更加简单。
Recap
Hyperledger Modules:
* Cello - toolkit to build Blockchain as a service revenue stream for a consulting company
* Explorer - To create something similar to etherscan or blockchaininfo. A tool to visualise transactions, blocks etc
* Composer - suite of tools for building blockchain business networks
好的，那我们学到了什么？

Hyperledger和permmissionless区块链技术之间的差异。 Hyperroger框架概述，如Iroha，Sawtooth，Fabric，Indy和Burrow。 Hyperloger模块概述，如Cello，Explorer和Composer。

希望尽快学习Hyperroger最重要的框架fabric课程的请访问Fabric区块链开发详解，本课程面向初学者，内容即包含Hyperledger Fabric的身份证书与MSP服务、权限策略、通道配置与启动、链码通信接口等核心概念，也包含Fabric网络设计、nodejs链码与应用开发的操作实践，是Nodejs工程师学习Fabric区块链开发的最佳选择。