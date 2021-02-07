超级账本FARBIC运维可视化监控教程
标签： Hyperledger  Fabric

Hyperledger Fabric是强调运维的区块链，Fabric自1.4版本开始就包含了用于peer和orderer节点运维的特性。本教程将介绍如何配置Fabric网络节点的运维管理服务，以及如何使用Prometheus和statsD/Graphite来可视化监控Hyperledger Fabric网络中各节点的实时运行指标。

相关教程：
Fabric区块链Java开发详解 |
Fabric区块链Node.JS开发详解

1、配置HYPERLEDGER FABRIC节点的运维服务
Hyperledger Fabric 1.4提供了如下的特性用于peer和orderer节点的运维服务API：

日志等级管理：/logspec
节点健康检查：/healthz
运行监控指标：/metrics
配置Fabric区块链节点的运维服务虽然不是尖端的火箭科技，但是如果你漏掉了某些细节也会觉得不那么容易。

首先修改core.yaml来配置peer节点的运维服务，主要包括监听地址的配置和TLS的配置（我们先暂时禁用这部分）。

用编辑器打开core.yaml：

$ vi ~/fabric-samples/config/core.yaml
下图显示了peer节点的运维服务监听地址listenAddress的默认值：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-pI1yF16B-1577443767715)(hyperledger-fabric-monitoring/core-yaml.png)]

现在让我们启动BYFN网络，并试着访问日志等级api。

$ cd ~/fabric-samples/first-network
$ ./byfn.sh -m up
$ docker exec -it cli bash
 
# curl peer0.org1.example.com:9443/logspec
不幸的是，curl命令返回如下错误：

curl: (7) Failed to connect to peer0.org1.example.com port 9443: Connection refused
让我们看看到底是什么原因。

首先检查我们的运维服务配置，打开core.yaml文件：

# vi /etc/hyperledger/fabric/core.yaml
可能你还记得，我们使用127.0.0.1作为listenAddress的值，这意味着
从外部无法访问运维api。

让我们在peer容器内进行必要的操作以再次检查。这次我们将使用wget代替
curl，因为在容器内没有安装curl。

$ docker exec -it peer0.org1.example.com bash
 
# wget peer0.org1.example.com:9443/logspec
和预期的一样，错误信息再次出现：

Connecting to peer0.org1.example.com (peer0.org1.example.com)… failed: Connection refused
但是如果用127.0.0.1来连接就会成功：

# wget 127.0.0.1:9443/logspec
结果如下：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-op1HzmnG-1577443767716)(hyperledger-fabric-monitoring/wget-ok.png)]

这标明我们需要设置运维的监听地址。

为此，修改docker-compose文件，为每个peer指定CORE_OPERATIONS_LISTENADDRESS环境变量。

$ vi ~/fabric-samples/first-network/base/docker-compose-base.yaml
参考下图进行修改：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-LNk1koy3-1577443767717)(hyperledger-fabric-monitoring/peer-compose.png)]

现在让我们再次尝试访问/logspec这个API，别忘了重新启动BYFN网络。

$ docker exec -it cli bash
 
# curl peer0.org1.example.com:9443/logspec
输出结果如下：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-nsdDWAQf-1577443767718)(hyperledger-fabric-monitoring/curl-1.png)]

类似的，我们可以检查节点健康状况：

# curl peer1.org1.example.com:9443/healthz
输出结果如下：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-GbSrYUxT-1577443767718)(hyperledger-fabric-monitoring/curl-2.png)]

最后，让我们在docker-compose-base.yaml中为每个peer设置ORE_METRICS_PROVIDER=prometheus来启用prometheus指标，并修改core.yaml来声明指标提供器为prometheus：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-7H6Zgdzv-1577443767719)(hyperledger-fabric-monitoring/core-metrics-provider.png)]

如果配置正确，在cli容器内执行如下命令后将会输出一组监控指标的当前值：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-J6agbtqW-1577443767719)(hyperledger-fabric-monitoring/metrics-1.png)]

2、用PROMETHEUS监控HYPERLEDGER FABRIC网络的运行指标
首先下载最新版本的Prometheus并解压：

$ curl -LO https://github.com/prometheus/prometheus/releases/download/v2.7.1/prometheus-2.7.1.linux-amd64.tar.gz \
$ tar -xvzf prometheus-2.7.1.linux-amd64.tar.gz
在prometheus文件夹中可以找到prometheus.yml文件。我们需要修改这个文件以便抓取Hyperledger Fabric的运行指标数据：在scrape_configs部分添加job_name和targets。

现在我们可以用docker来运行prometheus：

$ sudo docker run -d --name prometheus-server -p 9090:9090 \
  --restart always \
  -v /home/mccdev/prometheus/prometheus/prometheus.yml:/prometheus.yml \
  prom/prometheus \
  --config.file=/prometheus.yml
注意由于docker总是在专用网络启动容器，我们需要将prometheus容器加入fabric网络。用如下命令查看fabric网络：

$ docker inspect peer1.org1.example.com
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-VJpFM0Ps-1577443767720)(hyperledger-fabric-monitoring/fabric-network.png)]

将prometheus容器接入fabric网络：

$ sudo docker network connect net_byfn 5b8cbf9d8fa6
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-LE8WpOSJ-1577443767720)(hyperledger-fabric-monitoring/prometheus-connect.png)]

可以在如下地址访问统计信息：http://localhost:9090/

要检查是否成功抓取了peer的运行指标，输入scrape_samples_scraped查看结果表，内容应该非空。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-lG02kv9X-1577443767721)(hyperledger-fabric-monitoring/prometheus-table.png)]

3、用STATSD/GRAPHITE监控HYPERLEDGER FABRIC网络的运行指标
现在我们换StatsD来可视化监控Fabric网络的运行指标，Prometheus是pull方式，而StatsD则采用push方式。

首先，我们需要先从这里获取Graphite和StatsD的docker镜像。

然后，启动graphite容器：

$ docker run -d\
 --name graphite\
 --restart=always\
 -p 80:80\
 -p 2003-2004:2003-2004\
 -p 2023-2024:2023-2024\
 -p 8125:8125/udp\
 -p 8126:8126\
 graphiteapp/graphite-statsd
现在该修改docker-copose-base.yaml了。每个peer都应该设置如下的环境变量：

CORE_OPERATIONS_LISTENADDRESS
CORE_METRICS_PROVIDER
CORE_METRICS_STATSD_ADDRESS
CORE_METRICS_STATSD_NETWORK
CORE_METRICS_STATSD_PREFIX
我们也应当确保指定端口8125。peer的配置示例如下：

peer0.org1.example.com:
  container_name: peer0.org1.example.com
  extends:
   file: peer-base.yaml
   service: peer-base
  environment:
   - CORE_PEER_ID=peer0.org1.example.com
   - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
   - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.example.com:7051
   - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
   - CORE_PEER_LOCALMSPID=Org1MSP
   # metrics
   - CORE_OPERATIONS_LISTENADDRESS=peer0.org1.example.com:8125
   - CORE_METRICS_PROVIDER=statsd
   - CORE_METRICS_STATSD_ADDRESS=graphite:8125
   - CORE_METRICS_STATSD_NETWORK=udp
   - CORE_METRICS_STATSD_PREFIX=PEER_01
  volumes:
     - /var/run/:/host/var/run/
     - ../crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/fabric/msp
     - ../crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls:/etc/hyperledger/fabric/tls
     - peer0.org1.example.com:/var/hyperledger/production
  ports:
   - 7051:7051
   - 7053:7053
   - 8125:8125
需要留意使用的端口：对于peer0，端口应当是8125:8125，peer1则应当使用9125:8125，依此类推。

Orderer应当按如下配置：

- ORDERER_OPERATIONS_LISTENADDRESS=orderer.example.com:8125
- ORDERER_METRICS_PROVIDER=statsd
- ORDERER_METRICS_STATSD_ADDRESS=graphite:8125
- ORDERER_METRICS_STATSD_NETWORK=udp
- ORDERER_METRICS_STATSD_PREFIX=ORDERER
 
ports:
   - 7125:8125
现在让我们启动BYFN网络。

记得将graphite容器接入BYFN网络，否则你会看到如下错误：

Error: error getting endorser client for channel: endorser client failed to 
connect to peer0.org1.example.com:7051: failed to create new connection: 
context deadline exceeded
peer容器的日志将显示在网络中没有找到graphite：

Error: failed to initialize operations subystems: dial udp: lookup graphite 
on 127.0.0.11:53: no such host
使用如下命令将graphite容器接入fabric网络：

$ sudo docker network connect net_byfn graphite
一切正常的话，你应该可以在如下地址看到运行指标：

http://localhost/metrics/index.json
结果如下：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Tv8sWCpY-1577443767721)(hyperledger-fabric-monitoring/statsd-metrics.png)]

访问如下地址来查看可视化的数据：

http://localhost/
结果页面如下：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ZpELypfM-1577443767722)(hyperledger-fabric-monitoring/graphite.png)]