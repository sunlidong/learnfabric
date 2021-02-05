### ipfs name - 管理IPNS名称
ipfs name
使用ipfs name命令管理IPNS名称的发布和解析。

命令行
ipfs name
说明
IPNS是一个PKI名称空间， IPNS名称是公钥的哈希，私钥则用来发布签名的名称。 当发布或解析名称时，默认情况下总是使用发布者自身的节点ID，也就是节点 公钥的哈希。

可以使用ipfs key命令显示密钥列表、创建更多可用的名称及对应的密钥。

示例
使用节点默认名称发布一个<ipfs-path>：

> ipfs name publish /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
Published to QmbCMUZw6JFeZ7Wp9jkzbye3Fzp2GGcPgC3nmeUjfVF87n: /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
使用ipfs key命令创建一对新的密钥，然后用另一个名称发布一个<ipfs-path> ：

> ipfs key gen --type=rsa --size=2048 mykey
> ipfs name publish --key=mykey /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
Published to QmbCMUZw6JFeZ7Wp9jkzbye3Fzp2GGcPgC3nmeUjfVF87n: /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
解析默认名称：

> ipfs name resolve
/ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
解析指定的名称：

> ipfs name resolve QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
/ipfs/QmSiTko9JZyabH56y2fussEt1A5oDqsFXB3CkvAqraFryz
解析DNS域名：

> ipfs name resolve ipfs.io
/ipfs/QmaBvfZooxWkrv7D3r8LS9moNjzD2o525XMZze69hhoxf5
子命令
ipfs name publish <ipfs-path> - 发布IPNS名称
ipfs name resolve [<name>]    - 解析IPNS名称
使用ipfs name <subcmd> --help查看子命令的详细帮助信息。

### ipfs name publish - 发布IPNS名称
ipfs name publish
使用ipfs name publish <ipfs-path>命令发布IPNS名称。

命令行
ipfs name publish [--resolve=false] [--lifetime=<lifetime> | -t] [--ttl=<ttl>] [--key=<key> | -k] [--] <ipfs-path>
<ipfs-path> -要发布的ipfs对象的路径

选项
--resolve           bool   - 是否在发布前解析指定的路径，默认值：true
-t,      --lifetime string - 名称记录的有效时长，默认值：24h
    可以使用不同的计时单位，例如`300s`, `1.5h` 或 `2h45m`，有效的时间单位包括：
    "ns", "us" (or "µs"), "ms", "s", "m", "h".
--ttl               string - 名称记录允许缓存的时长。注意，这是一个实验阶段的特性
-k,      --key      string - 要使用的密钥对名称，可以使用`ipfs key list`查看可用的密钥对。默认值：self
说明
IPNS是一个PKI名称空间， IPNS名称是公钥的哈希，私钥则用来发布签名的名称。 当发布或解析名称时，默认情况下总是使用发布者自身的节点ID，也就是节点 公钥的哈希。

可以使用ipfs key命令显示密钥列表、创建更多可用的名称及对应的密钥。

示例
使用节点默认名称发布一个<ipfs-path>：

> ipfs name publish /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
Published to QmbCMUZw6JFeZ7Wp9jkzbye3Fzp2GGcPgC3nmeUjfVF87n: /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
使用ipfs key命令创建一对新的密钥，然后用另一个名称发布一个<ipfs-path> ：

> ipfs key gen --type=rsa --size=2048 mykey
> ipfs name publish --key=mykey /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
Published to QmbCMUZw6JFeZ7Wp9jkzbye3Fzp2GGcPgC3nmeUjfVF87n: /ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy


### ipfs name resolve - 解析IPNS名称
ipfs name resolve
使用ipfs name resolve [<name>]解析IPNS名称。

命令行
ipfs name resolve [--recursive | -r] [--nocache | -n] [--] [<name>]
[<name>] - 要解析的IPNS名称，默认值为节点ID

选项
-r, --recursive bool - 是否递归解析直至结果不再是IPNS名称。默认值： false
-n, --nocache   bool - 是否不使用缓存进行解析，默认值： false
说明
IPNS是一个PKI名称空间， IPNS名称是公钥的哈希，私钥则用来发布签名的名称。 当发布或解析名称时，默认情况下总是使用发布者自身的节点ID，也就是节点 公钥的哈希。

可以使用ipfs key命令显示密钥列表、创建更多可用的名称及对应的密钥。

示例
解析默认名称：

> ipfs name resolve
/ipfs/QmatmE9msSfkKxoffpHwNLNKgwZG8eT9Bud6YoPab52vpy
解析指定名称：

> ipfs name resolve QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
/ipfs/QmSiTko9JZyabH56y2fussEt1A5oDqsFXB3CkvAqraFryz
解析DNS名称：

> ipfs name resolve ipfs.io
/ipfs/QmaBvfZooxWkrv7D3r8LS9moNjzD2o525XMZze69hhoxf5
