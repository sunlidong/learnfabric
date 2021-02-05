### ipfs pin - 管理ipfs对象的固定
ipfs pin
使用ipfs pin命令在本地仓库中固定（或解除固定）ipfs对象。

命令行
ipfs pin
子命令
ipfs pin add <ipfs-path>...  - 将指定ipfs对象固定在本地存储中
ipfs pin ls [<ipfs-path>]... - 列表显示本地存储中被固定的ipfs对象
ipfs pin rm <ipfs-path>...   - 从本地存储中移除被固定的ipfs对象
使用ipfs pin <subcmd> --help查看子命令的详细帮助信息。

### ipfs pin add - 固定ipfs对象
ipfs pin add
使用ipfs pin add <ipfs-path>...命令将ipfs对象固定在本地存储中。

命令行
ipfs pin add [--recursive=false] [--progress] [--] <ipfs-path>...
<ipfs-path>... - 要固定的ipfs对象的路径

选项
-r,       --recursive bool - 是否递归固定指定的ipfs对象，默认值：true
--progress            bool - 是否显示命令执行进度
说明
ipfs pin命令将指定路径的IPFS对象固定在本地磁盘存储中。

### ipfs pin ls - 显示被固定对象列表
ipfs pin ls
使用ipfs pin ls [<ipfs-path>]...命令列表显示本地存储中已固定的ipfs对象。

命令行
ipfs pin ls [--type=<type> | -t] [--quiet | -q] [--] [<ipfs-path>...]
[<ipfs-path>]... - 要列举其被固定内容的ipfs对象路径

选项
-t, --type  string - 要显示的被固定密钥的类型，可以是"direct", "indirect", "recursive", 或者 "all"。默认值：all
-q, --quiet bool   - 是否仅显示被固定对象的哈希，默认值：false
说明
ipfs pin ls命令返回一组本地固定的ipfs对象。默认情况下，该命令将返回 所有被固定的对象，不过可以使用--type选项来限定要求仅返回特定类型的 被固定对象，或使用路径参数来限定仅返回指定ipfs对象所链接的被固定对象。

使用--type=<type>选项来指定要显示的密钥类型。可选值如下：

"direct": 选中指定对象
"recursive": 选中指定对象及其后代对象
"indirect": 被祖先对象间接选中，类似于refcount
"all"： 全部对象
当使用路径参数时，如果该路径所指向的对象不是被固定对象，ipfs pin ls 命令将失败。同时，如果使用--type=<type>选项指定一个与该路径对象不符 的类型，命令也会失败。

示例
$ echo "hello" | ipfs add -q
QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
$ ipfs pin ls
QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN recursive
# now remove the pin, and repin it directly
$ ipfs pin rm QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
unpinned QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
$ ipfs pin add -r=false QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
pinned QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN directly
$ ipfs pin ls --type=direct
QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN direct
$ ipfs pin ls QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN direct

### ipfs pin rm - 解除ipfs对象的固定
ipfs pin rm
使用ipfs pin rm <ipfs-path>...命令从本地存储中解除指定ipfs对象的固定。

命令行
ipfs pin rm [--recursive=false] [--] <ipfs-path>...
<ipfs-path>... - 要接触固定的对象路径

选项
-r, --recursive bool - 是否递归接触指定对象的固定，默认值：true
说明
ipfs pin rm命令用来接触指定对象的固定，以便其可以被垃圾回收机制 所处理。默认情况下，该命令将递归解除指定对象的及其后代的固定，使用 -r=false选项来解除直接固定。
