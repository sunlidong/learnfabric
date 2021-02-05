ipfs - 命令行简介
ipfs是基于默克尔有向无环图（merkle dag）的全球性p2p文件系统。
命令行
ipfs [--config=<config> | -c] [--debug=<debug> | -D] 
     [--help=<help>] [-h=<h>] [--local=<local> | -L] 
     [--api=<api>] <command> ...
命令行选项
-c, --config string - 配置文件路径
-D, --debug  bool   - 开启调试模式，默认值：false
--help       bool   - 是否显示完整的命令帮助文档，默认值：false
-h           bool   - 显示简明版的命令帮助文档，默认值：false
-L, --local  bool   - 仅在本地执行命令，不使用后台进程。默认值：false
--api        string - 使用指定的API实例，默认值：`/ip4/127.0.0.1/tcp/5001`
基本子命令
init          初始化ipfs本地配置
add <path>    将指定文件添加到IPFS
cat <ref>     显示指定的IPFS对象数据
get <ref>     下载指定的IPFS对象
ls <ref>      列表显示指定对象的链接
refs <ref>    列表显示指定对象的链接哈希
数据结构子命令
block         操作数据仓中的裸块
object        操作有向图中的裸节点
files         以unix文件系统方式操作IPFS对象
dag           操作IPLD文档，目前处于实验阶段
高级子命令
daemon        启动后台服务进程
mount         挂接只读IPFS
resolve       名称解析
name          发布、解析IPNS名称
key           创建、列表IPNS名称键值对
dns           解析DNS链接
pin           在本地存储中固定IPFS对象
repo          操作IPFS仓库
stats         各种运营统计
filestore     管理文件仓，目前处于实验阶段
网络子命令
id            显示IPFS节点信息
bootstrap     添加、删除启动节点
swarm         管理p2p网络的连接
dht           查询分布哈希表中的值或节点信息
ping          检测连接延时
diag          打印诊断信息
工具子命令
config        管理配置信息
version       显示ipfs版本信息
update        下载并应用go-ipfs更新
commands      列表显示全部可用命令
使用ipfs <command> --help来了解特定命令的详细帮助信息。

本地仓库路径
ipfs使用本地文件系统中的仓库存储内容。默认情况下，本地仓库位于 ~/.ipfs。你可以设置IPFS_PATH环境变量来定义本地仓库的位置：

export IPFS_PATH=/path/to/ipfsrepo
命令行退出状态
命令行的退出码如下：

0：执行成功
1：执行失败