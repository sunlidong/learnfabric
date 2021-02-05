### ipfs命令行简介
### ipfs add - 添加文件或目录
ipfs add将指定文件或目录添加到IPFS中。

命令行
ipfs add [--recursive | -r] [--quiet | -q] [--quieter | -Q] 
         [--silent] [--progress | -p] [--trickle | -t] [--only-hash | -n] 
         [--wrap-with-directory | -w] [--hidden | -H] [--chunker=<chunker> | -s] 
         [--pin=false] [--raw-leaves] [--nocopy] [--fscache] [--] <path>...
<path>... - 要添加到ipfs中的文件的路径

选项
-r,         --recursive           bool   - 递归添加目录内容，默认值：false
-q,         --quiet               bool   - 安静模式，执行过程中输出显示尽可能少的信息
-Q,         --quieter             bool   - 更安静模式，仅输出最终的结果哈希值
--silent                          bool   - 静默模式，不输出任何信息
-p,         --progress            bool   - 流式输出过程数据
-t,         --trickle             bool   - 使用trickle-dag格式进行有向图生成
-n,         --only-hash           bool   - Only chunk and hash - do not write to disk.
-w,         --wrap-with-directory bool   - 使用目录对象包装文件
-H,         --hidden              bool   - 包含隐藏文件，仅在进行递归添加时有效
-s,         --chunker             string - 使用的分块算法
--pin                             bool   - 添加时固定对象，默认值：true
--raw-leaves                      bool   - 叶节点使用裸块，实验特性
--nocopy                          bool   - 使用filestore添加文件，实验特性
--fscache                         bool   - 为已有块检查filestore，实验特性
说明
将<path>的内容添加到ipfs中。使用-r来添加目录。目录内容的添加 是递归进行的，以便生成ipfs的默克尔DAG图。

包装选项-w将文件包装到一个目录中，该目录仅包含已经添加的文件，这意味着 文件将保留其文件名。例如：

> ipfs add example.jpg
added QmbFMke1KXqnYyBBWxB74N4c5SBnJMVAiMNRcGu6x1AwQH example.jpg
> ipfs add example.jpg -w
added QmbFMke1KXqnYyBBWxB74N4c5SBnJMVAiMNRcGu6x1AwQH example.jpg
added QmaG4FuMqEBnQNn3C8XJ5bpW8kLs7zq2ZXgHptJHbKDDVx
现在可以通过网关访问添加的文件，类似于：

/ipfs/QmaG4FuMqEBnQNn3C8XJ5bpW8kLs7zq2ZXgHptJHbKDDVx/example.jpg

### ipfs cat - 显示对象内容
ipfs cat
ipfs cat <ipfs-path>...命令用来显示IPFS对象数据。

命令行
ipfs cat [--] <ipfs-path>...
<ipfs-path>... - 要显示的IPFS对象路径，可指定多个

说明
ipfs cat命令显示指定路径的一个或多个IPFS/IPNS对象的数据内容。
### ipfs commands - 显示可用命令
ipfs commands
ipfs commands命令用来列出所有可用的ipfs命令。

命令行
ipfs commands [--flags | -f]
选项
-f, --flags bool - 是否显示命令的标志，默认值：false
说明
ipfs commands命令将列表显示所有可用的命令以及子命令，然后退出。
### ipfs daemon - 节点服务进程
ipfs daemon - ipfs节点服务进程
ipfs daemon命令用来启动一个连接网络的IPFS节点。

命令行
ipfs daemon [--init] [--routing=<routing>] [--mount] [--writable] 
            [--mount-ipfs=<mount-ipfs>] [--mount-ipns=<mount-ipns>] 
            [--unrestricted-api] [--disable-transport-encryption] 
            [--enable-gc] [--manage-fdlimit=false] [--offline] [--migrate] 
            [--enable-pubsub-experiment] [--enable-mplex-experiment=false]
选项
--init                         bool   - 是否使用默认设置自动初始化ipfs，默认值：false
--routing                      string - 路由选项，默认值：dht
--mount                        bool   - 是否将IPFS挂载到文件系统，默认值：false
--writable                     bool   - 是否允许使用`POST/PUT/DELETE`修改对象，默认值： false.
--mount-ipfs                   string - 当使用--mount选项时IPFS的挂接点，默认值采用配置文件中的设置
--mount-ipns                   string - 当使用--mount选项时IPNS的挂接点，默认值采用配置文件中的设置
--unrestricted-api             bool   - 是否允许API访问未列出的哈希，默认值：false
--disable-transport-encryption bool   - 是否进制传输层加密，默认值：false。当调试协议时可开启该选项
--enable-gc                    bool   - 是否启用自动定时仓库垃圾回收，默认值：false
--manage-fdlimit               bool   - 是否按需自动提高文件描述符上限，默认值：false
--offline                      bool   - 是否离线运行，即不连接到网络，仅提供本地API，默认值：false
--migrate                      bool   - true对应于mirage提示时输入yes，false对应于输入no
--enable-pubsub-experiment     bool   - 是否启用发布订阅（pubsub）特性，该特性目前尚处于实验阶段
--enable-mplex-experiment      bool   - 是否启用`go-multiplex`流多路处理器，默认值：true
说明
服务进程将在指定的端口监听网络连接。使用ipfs config Addresses 命令修改默认端口。

例如，修改网关监听端口：

ipfs config Addresses.Gateway /ip4/127.0.0.1/tcp/8082
同样的方式修改API地址：

ipfs config Addresses.API /ip4/127.0.0.1/tcp/5002
在修改地址后，确保重新启动服务进程以便生效。

默认情况下，网络仅在本地可以访问，如果希望允许其他计算机访问，可以 使用地址0.0.0.0。例如：

ipfs config Addresses.Gateway /ip4/0.0.0.0/tcp/8080
当开放API访问时请千万小心，这存在一定的安全风险，因为任何人都可以 远程控制你的节点。如果你希望远程控制节点，请使用防火墙、授权代理 或其他服务来保护该API访问地址。

HTTP头
ipfs支持向API和网关传入任意HTTP头信息。你可以使用API.HTTPHeaders和 Gateway.HTTPHeaders配置项进行配置。例如：

ipfs config --json API.HTTPHeaders.X-Special-Header '["so special :)"]'
ipfs config --json Gateway.HTTPHeaders.X-Special-Header '["so special :)"]'
需要指出的是，值应当是字符串数组，因为HTTP头可以包含多个值，而且这样也 方面传给其他的库。

对于API而言，可以同样的方式为其设置CORS头：

ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin '["example.com"]'
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "GET", "POST"]'
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Credentials '["true"]'
停止服务
要停止服务进程，发送SIGINT信号即可，例如，使用Ctrl-C组合键。也可以发送 SIGTERM信号，例如，使用kill命令。服务进程需要稍等一下以便优雅退出，但是你 可以继续发送一次信息来强制服务进程立刻退出。

IPFS_PATH环境变量
ipfs使用本地文件系统建立本地仓库。默认情况下，本地仓库的目录是~/.ipfs， 可以设置IPFS_PATH环境变量来自定义本地仓库路径：

export IPFS_PATH=/path/to/ipfsrepo
路由
默认情况下，ipfs使用分布式哈希表（DHT）进行内容的路由。目前有一个尚处于 试验阶段的替代方案，使用纯客户端模式来操作分布式哈希表，可以在启动服务 进程时，使用如下命令启动这一替代路由方案：

ipfs daemon --routing=dhtclient
该选项在实验阶段结束后将转变为一个配置选项。

弃用通知
ipfs之前使用过环境变量API_ORIGIN：

export API_ORIGIN="http://localhost:8888/"
该环境变量已经被弃用。在当前版本中还可以使用，但在将来的版本中将会删除对 此环境变量的支持。请使用前述HTTP头信息设置方法来取代该环境变量。
### ipfs dns - 解析域名
ipfs dns
ipfs dns <domain-name>命令解析DNS链接。

命令行
ipfs dns [--recursive | -r] [--] <domain-name>
<domain-name> - 要解析的域名

选项
-r, --recursive bool - 是否递归解析直至结果不再是DNS链接，默认值：false
说明
ipfs使用的哈希（multihash）很难记忆，但是域名通常更容易记住。可以 使用DNS的TXT记录为ipfs哈希创建容易记忆的别名，TXT记录可以指向其他的DNS 链接、IPFS对象、IPNS键等等。

ipfs dns命令将指定的DNS链接解析为其目标对象。例如，根据如下DNS TXT记录：

> dig +short TXT _dnslink.ipfs.io
dnslink=/ipfs/QmRzTuh2Lpuz7Gr39stNr6mTFdqAghsZec1JoUnfySUzcy
解析结果如下：

> ipfs dns ipfs.io
/ipfs/QmRzTuh2Lpuz7Gr39stNr6mTFdqAghsZec1JoUnfySUzcy
解析器可以递归解析，例如：

> dig +short TXT recursive.ipfs.io
dnslink=/ipns/ipfs.io
> ipfs dns -r recursive.ipfs.io
/ipfs/QmRzTuh2Lpuz7Gr39stNr6mTFdqAghsZec1JoUnfySUzcy
### ipfs get - 读取ipfs对象
ipfs get
ipfs get <ipfs-path>命令用来下载指定路径的IPFS对象。

命令行
ipfs get [--output=<output> | -o] [--archive | -a] [--compress | -C] [--compression-level=<compression-level> | -l] [--] <ipfs-path>
<ipfs-path> - IPFS对象路径

选项
-o, --output            string - 对象内容输出路径
-a, --archive           bool   - 是否输出为TAR包，默认值：false
-C, --compress          bool   - 是否使用GZIP算法压缩输出文件，默认值：false
-l, --compression-level int    - 压缩等级，1-9，默认值：-1
说明
ipfs get命令将指定路径的IPFS/IPNS对象的数据下载到磁盘。

默认情况下，输出文件将保存在./<ipfs-path>，可以使用--output或-o 选项来指定一个其他路径。

使用--archive或-a选项输出tar包。

使用--compress或-C选项来压缩输出结果，格式为GZIP。可以同时使用-l=<1-9> 来指定压缩等级。
### ipfs id - 显示节点信息
ipfs id
ipfs id [<peerid>] - 显示ipfs节点的信息

命令行
ipfs id [--format=<format> | -f] [--] [<peerid>]
[<peerid>] - 要查看的节点id

选项
-f, --format string - 输出格式，可选
说明
显示输出指定对端节点旳信息。如果不指定节点，则显示输出本地 节点信息。

ipfs id命令的--format或-f选项支持一下输出格式键：

<id> : 节点id
<aver>: 代理版本
<pver>: 协议版本
<pubkey>: 公钥
<addrs>: 地址，采用换行符分隔多个地址
示例
显示id为Qmece2RkXhsKe5CRooNisBTh4SK119KrXXGmoK6V3kb8aH的节点旳地址：

ipfs id Qmece2RkXhsKe5CRooNisBTh4SK119KrXXGmoK6V3kb8aH -f="<addrs>\n"

### ipfs init - 初始化配置文件
ipfs init
ipfs init [<default-config>]命令初始化ipfs配置文件。

命令行
ipfs init [--bits=<bits> | -b] [--empty-repo | -e] [--] [<default-config>]
[<default-config>] - 使用该指定配置进行初始化

选项
-b, --bits       int  - 生成的RSA私钥位数，默认值：2048
-e, --empty-repo bool - 是否不在本地存储中添加、固定帮助文件。默认值：false
说明
ipfs init命令负责初始化ipfs配置文件并生成新的密钥对。

ipfs使用本地文件系统中的仓库。默认情况下，本地仓库的路径为~/.ipfs。可以使用 IPFS_PATH环境变量来自定义本地仓库路径。例如：

export IPFS_PATH=/path/to/ipfsrepo
### ipfs ls - 显示目录内容
ipfs ls
ipfs ls <ipfs-path>... 命令列表显示目录内容。

命令行
ipfs ls [--headers | -v] [--resolve-type=false] [--] <ipfs-path>...
<ipfs-path>... - 要列举其链接的IPFS对象路径

选项
-v,           --headers bool - 是否显示表头（哈希、大小、名称），默认值：false
--resolve-type          bool - 是否解析所链接对象的类型，默认值：true
说明
显示指定路径的IPFS或IPNS对象的内容，格式如下：

<link base58 hash> <link size in bytes> <link name>
JSON输出中包含有类型信息。

### ipfs mount - 挂载ipfs
ipfs mount
ipfs mount命令以只读方式将ipfs挂接到文件系统。

命令行
ipfs mount [--ipfs-path=<ipfs-path> | -f] [--ipns-path=<ipns-path> | -n]
选项
-f, --ipfs-path string - The path where IPFS should be mounted.
-n, --ipns-path string - The path where IPNS should be mounted.
说明
ipfs mount命令在操作系统中的只读挂接点挂载IPFS。

默认情况下使用配置文件中设定的/ipfs和/ipns挂接点，可以使用上述选项 自定义挂接点。

挂接后该目录中的所有IPFS对象都是可以访问的。需要指出的是，由于根目录 是虚拟的，因此不可列表查看其内容，只能直接访问指定的路径。

需要在使用ipfs mount命令之前创建/ipfs和/ipns目录：

> sudo mkdir /ipfs /ipns
> sudo chown `whoami` /ipfs /ipns
> ipfs daemon &
> ipfs mount
示例
下面的代码将创建目录foo和文件foo/bar，然后将目录foo添加到ipfs中：

# setup
> mkdir foo
> echo "baz" > foo/bar
> ipfs add -r foo
added QmWLdkp93sNxGRjnFHPaYg8tCQ35NBY3XPn6KiETd3Z4WR foo/bar
added QmSh5e7S6fdcu75LAbXNZAFY2nGyZUJXyLCJDvn2zRkWyC foo
> ipfs ls QmSh5e7S6fdcu75LAbXNZAFY2nGyZUJXyLCJDvn2zRkWyC
QmWLdkp93sNxGRjnFHPaYg8tCQ35NBY3XPn6KiETd3Z4WR 12 bar
> ipfs cat QmWLdkp93sNxGRjnFHPaYg8tCQ35NBY3XPn6KiETd3Z4WR
baz
下面的代码将foo对应的IPFS对象挂接到/ipfs目录，并访问其中的 bar文件：

# mount
> ipfs daemon &
> ipfs mount
IPFS mounted at: /ipfs
IPNS mounted at: /ipns
> cd /ipfs/QmSh5e7S6fdcu75LAbXNZAFY2nGyZUJXyLCJDvn2zRkWyC
> ls
bar
> cat bar
baz
> cat /ipfs/QmSh5e7S6fdcu75LAbXNZAFY2nGyZUJXyLCJDvn2zRkWyC/bar
baz
> cat /ipfs/QmWLdkp93sNxGRjnFHPaYg8tCQ35NBY3XPn6KiETd3Z4WR
baz

### ipfs ping - 测试节点连通性
ipfs ping
ipfs ping <peer ID>... - 向指定的ipfs主机发送回显请求包

命令行
ipfs ping [--count=<count> | -n] [--] <peer ID>...
<peer ID>... - 要ping的对端节点id

选项
-n, --count int - 要发送的ping包数量，默认值：10
说明
ipfs ping命令是一个测试向其他节点发送数据能力的工具，它利用路由系统 找到指定的节点，发送ping包，等待pong包，然后输出显示包往返的延迟信息。

### ipfs resolve - 名称解析

ipfs resolve
ipfs resolve <name>命令用来解析指定名称对应的目标值。

命令行
ipfs resolve [--recursive | -r] [--] <name>
<name> - 要解析的名称The name to resolve.

选项
-r, --recursive bool - 是否递归解析直至获得IPFS名称，默认值：false
说明
有许多可变名称协议彼此链接或链接到IPNS。例如IPFS引用（目前）可以指向一个IPFS对象， DNS链接可以指向其他DNS链接、IPNS实体或IPFS对象。

ipfs resolve命令支持将以上格式的标识符解析为其指向的目标。

示例
解析指定id对应的值：

$ ipfs resolve /ipns/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
/ipfs/Qmcqtw8FfrVSBaRmbWwHxt3AuySBhJLcvmFYi3Lbc4xnwj
解析另一个名称的值：

$ ipfs resolve /ipns/QmbCMUZw6JFeZ7Wp9jkzbye3Fzp2GGcPgC3nmeUjfVF87n
/ipns/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
递归协议另一个名称的值：

$ ipfs resolve -r /ipns/QmbCMUZw6JFeZ7Wp9jkzbye3Fzp2GGcPgC3nmeUjfVF87n
/ipfs/Qmcqtw8FfrVSBaRmbWwHxt3AuySBhJLcvmFYi3Lbc4xnwj
解析IPFS有向图路径的值：

$ ipfs resolve /ipfs/QmeZy1fGbwgVSrqbfh9fKQrAWgeyRnj7h8fsHS1oy3k99x/beep/boop
/ipfs/QmYRMjyvAiHKN9UTi8Bzt1HUspmSRD8T8DwxfSMzLgBon1

### ipfs update
### ipfs version - 显示版本信息