### ipfs bootstrap - 管理启动引导节点
ipfs bootstrap - 管理启动引导节点
ipfs bootstrap命令用来显示或编辑引导节点列表。

命令行
ipfs bootstrap
子命令
ipfs bootstrap add [<peer>]... - 将一个或多个节点添加到引导列表
ipfs bootstrap list            - 显示引导列表中的节点
ipfs bootstrap rm [<peer>]...  - 从引导列表中删除一个或多个节点
说明
运行无参数的ipfs bootstrap等价于运行ipfs bootstrap list。

安全警告 SECURITY WARNING:
bootstrap命令操作的引导列表中包含引导节点旳地址，这些节点是网络中的 可信节点，通过它们来获取其他节点旳信息。在编辑引导列表之前，请务必 了解添加或删除节点旳风险。

使用ipfs bootstrap <subcmd> --help获取子命令的详细帮助信息。

### ipfs bootstrap add - 添加引导节点
ipfs bootstrap add
ipfs bootstrap add [<peer>]...命令将一个或多个节点添加到引导列表。

命令行
ipfs bootstrap add [--default] [--] [<peer>...]
[<peer>]... - 要添加的节点，格式为<multiaddr>/<peerID>

选项
--default bool - 是否添加默认的引导节点。该选项已废弃，请使用`default`子命令。
说明
该命令将输出新添加的节点列表，即之前不在引导列表中的新节点。

安全警告
bootstrap命令操作的引导列表中包含引导节点旳地址，这些节点是网络中的 可信节点，通过它们来获取其他节点旳信息。在编辑引导列表之前，请务必 了解添加或删除节点旳风险。

子命令
ipfs bootstrap add default - 将默认节点添加到引导列表
使用ipfs bootstrap add <subcmd> --help命令获取子命令的详细帮助信息。

### ipfs bootstrap list - 显示引导节点列表

ipfs bootstrap list - 显示引导节点列表
ipfs bootstrap list命令显示引导列表中的节点。

命令行
ipfs bootstrap list
说明
输出的节点格式为：<multiaddr>/<peerID>。

### ipfs bootstrap rm - 删除引导节点
ipfs bootstrap rm - 删除引导节点
ipfs bootstrap rm [<peer>]...命令从引导列表中移除一个或多个节点。

命令行
ipfs bootstrap rm [--all] [--] [<peer>...]
[<peer>]... - 要移除的节点，格式为：<multiaddr>/<peerID>

###　选项

--all bool - 删除所有的引导节点。该选项已废弃，请使用`all`子命令
说明
ipfs bootstrap rm命令输出被移除的节点列表。

安全警告
bootstrap命令操作的引导列表中包含引导节点旳地址，这些节点是网络中的 可信节点，通过它们来获取其他节点旳信息。在编辑引导列表之前，请务必 了解添加或删除节点旳风险。

子命令
ipfs bootstrap rm all - 删除引导列表中的所有节点
使用ipfs bootstrap rm <subcmd> --help获取子命令的详细帮助信息。
