### ipfs bitswap
### ipfs bitswap - bitswap操作接口
ipfs bitswap - bitswap操作接口
ipfs bitswap用来操作bitswap代理。

命令行
ipfs bitswap
子命令
ipfs bitswap ledger <peer>   - 显示指定节点旳当前账本
ipfs bitswap stat            - 显示bitswap代理的诊断信息
ipfs bitswap unwant <key>... - 从期望列表（wantlist）中删除指定的块
ipfs bitswap wantlist        - 显示期望列表中的块
使用ipfs bitswap <subcmd> --help查看具体子命令的帮助信息。

### ipfs bitswap ledger - 显示指定节点的账本
ipfs bitswap ledger - 显示指定节点的账本
ipfs bitswap ledger <peer>用来显示指定节点旳当前账本。

命令行
ipfs bitswap ledger [--] <peer>
<peer> - 要查看账本的节点ID（B58）

说明
Bitswap决策引擎跟踪IPFS节点间交换的字节数量，并保存该信息作为账本集合。这个 命令可以打印指定节点旳账本。

### ipfs bitswap stat - 显示诊断信息

### ipfs bitswap unwant - 从需求列表移除块

### ipfs bitswap wanglist - 显示需求列
ipfs bitswap unwant - 从需求列表移除块
ipfs bitswap unwant <key>...命令用来从需求列表（wantlist）中移除指定的块。

命令行
ipfs bitswap unwant [--] <key>...
<key>... - 要从需求列表中移除的块，可以指定多个