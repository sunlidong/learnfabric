### ipfs refs - 显示对象链接清单
ipfs refs
使用ipfs refs <ipfs-path>...命令显示一个对象的链接清单。

命令行
ipfs refs [--format=<format>] [--edges | -e] [--unique | -u] [--recursive | -r] [--] <ipfs-path>...
<ipfs-path>... - 要显示其引用链接的ipfs对象路径

选项
--format            string - Emit edges with given format. Available tokens: <src> <dst> <linkname>. Default: <dst>.
-e,     --edges     bool   - Emit edge format: `<from> -> <to>`. Default: false.
-u,     --unique    bool   - Omit duplicate refs from output. Default: false.
-r,     --recursive bool   - Recursively list links of child nodes. Default: false.
说明
ipfs refs命令显示一个指定的IPFS或IPNS对象所包含的全部引用链接。格式如下：

<link base58 hash>
注意，可以使用-r选项递归列出所有的引用链接。

子命令
ipfs refs local - 列出所有的本地链接
使用ipfs refs <subcmd> --help显示子命令的详细帮助信息。

### ipfs refs local - 显示本地对象清单
ipfs refs local
使用ipfs refs local命令列出所有的本地引用。

命令行
ipfs refs local
说明
显示所有本地对象的哈希。
