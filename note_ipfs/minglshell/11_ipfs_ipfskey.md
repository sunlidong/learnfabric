### ipfs key - 管理名称密钥
ipfs key
ipfs key命令用来管理IPNS名称密钥对。

命令行
ipfs key
说明
使用ipfs key gen命令创建一个新的密钥对，一边用于IPNS和ipfs name publish命令。

例如：

> ipfs key gen --type=rsa --size=2048 mykey
> ipfs name publish --key=mykey QmSomeHash
使用ipfs key list显示当前可用的密钥。例如：

> ipfs key list
self
mykey
子命令
ipfs key gen <name> - 创建新的密钥对
ipfs key list       - 列表显示本地保存的所有密钥对
使用ipfs key <subcmd> --help查看子命令的详细帮助信息。

### ipfs key gen - 生成名称密钥
ipfs key gen
ipfs key gen <name>命令用来创建新的密钥对。

命令行
ipfs key gen [--type=<type> | -t] [--size=<size> | -s] [--] <name>
<name> - 要创建的密钥对的名称

选项
-t, --type string - 密钥类型，支持：rsa和ed25519
-s, --size int    - 密钥的位数

### ipfs key list - 显示名称密钥列表
ipfs key list
ipfs key list命令列表显示本地所有密钥对。

命令行
ipfs key list [-l]
选项
-l bool - 是否显示密钥详情
