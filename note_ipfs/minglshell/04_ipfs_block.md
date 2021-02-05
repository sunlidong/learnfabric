### ipfs block - 裸块操作接口
ipfs block - IPFS裸块操作接口
ipfs block命令用来操作IPFS裸块。

命令行
ipfs block
说明
ipfs block命令用于操作IPFS裸块，它可以读取标准输入stdin或写入标准输出 stdout，<key>是base58编码的哈希（multihash）。

子命令
ipfs block get <key>    - 读取IPFS裸块
ipfs block put <data>   - 将输入数据存入IPFS块
ipfs block rm <hash>... - 移除指定的IPFS块
ipfs block stat <key>   - 打印指定IPFS裸块的信息
使用ipfs block <subcmd> --help获取子命令的详细信息。

### ipfs block get - 读取裸块
ipfs block get - 读取IPFS裸块
ipfs block get <key>命令读取指定的IPFS裸块。

命令行
ipfs block get [--] <key>
<key> - 要读取的块的base58哈希（multihash）

说明
ipfs block get用来提取指定IPFS裸块的信息并输出到标准输出设备stdout。 参数<key>是一个base58编码的哈希（multihash）。

### ipfs block put - 写入裸块
ipfs block put - 写入IPFS裸块
ipfs block put <data>命令将输入数据写入IPFS块。

命令行
ipfs block put [--format=<format> | -f] [--mhtype=<mhtype>] 
               [--mhlen=<mhlen>] [--] <data>
<data> - 要存入IPFS块的数据

选项
-f,     --format string - 要创建的块格式，默认值：v0
--mhtype         string - multihash哈希函数，默认值: sha2-256.
--mhlen          int    - multihash哈希长度，默认值： -1
说明
ipfs block put用于将数据存入IPFS裸块，它从标准输入设备stdin读取数据。

### ipfs block rm - 删除裸块
ipfs block rm
ipfs block rm <hash>...命令删除指定的IPFS块，可以删除多个。

命令行
ipfs block rm [--force | -f] [--quiet | -q] [--] <hash>...
<hash>... - 要删除的块的multihash哈希，Bash58编码

选项
-f, --force bool - 是否忽略不存在的块，默认值：false
-q, --quiet bool - 是否使用安静模式显示最少量输出信息，默认值：false
说明
ipfs block rm命令用来删除ipfs裸块。可以指定一组base58编码的待删除块的哈希值。

### ipfs block stat - 显示块信息
ipfs block stat - 显示块信息
ipfs block stat <key>命令用来显示一个ipfs裸块的信息。

命令行
ipfs block stat [--] <key>
<key> - 块的哈希值，base58编码

说明
ipfs block stat用来提取IPFS裸块的信息，它在标准输出设备stdout 上输出如下信息：

Key - base58编码的哈希（multihash）
Size - 块字节数