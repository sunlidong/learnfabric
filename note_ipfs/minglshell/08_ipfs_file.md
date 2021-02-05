### ipfs file
ipfs file
ipfs file命令用来与表征unix文件系统的IPFS对象进行交互。

命令行
ipfs file
说明
ipfs file命令为使用IPFS对象表示的文件系统提供了一个与传统 文件系统类似的操作接口，它隐藏了ipfs中的实现细节，例如扇出和切块等。

子命令
ipfs file ls <ipfs-path>... - 显示指定路径的ipfs对象的目录内容
可以使用ipfs file <subcmd> --help来显示子命令的详细帮助信息。

### ipfs file ls
ipfs file ls
ipfs file ls <ipfs-path>...命令显示unix文件系统对象的目录内容。

命令行
ipfs file ls [--] <ipfs-path>...
<ipfs-path>... - 要显示其目录内容的IPFS对象路径

说明
ipfs file ls命令用来显示指定路径的IPFS或IPNS对象的内容。

JSON输出中包括与大小相关的信息。对于文件，其子对象大小是文件内容的 总大小，对于目录，其子对象大小是IPFS对象链接数。

路径可以使用无前缀引用格式，此时默认使用/ipfs前缀而非/ipns前缀：

例如，下面两种使用方法是等效的：

> ipfs file ls QmW2WQi7j6c7UgJTarActp7tDNikE4B2qXtFCfLPdsgaTQ
cat.jpg
> ipfs file ls /ipfs/QmW2WQi7j6c7UgJTarActp7tDNikE4B2qXtFCfLPdsgaTQ
cat.jpg
注意，ipfs file ls命令已被废弃，在将来的版本中将被移除。可能的情况下 应该使用ipfs ls命令。

