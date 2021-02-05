### ipfs config - 配置管理
ipfs config - 配置管理
ipfs config <key> [<value>]命令族用来读取或写入ipfs的配置信息。

命令行
ipfs config [--bool] [--json] [--] <key> [<value>]
<key> - 配置项的键，例如Addresses.API
[<value>] - 要写入的配置项的值
选项
--bool bool - 是否设置布尔值，默认值：false
--json bool - 是否解析 字符串化的JSON对象，默认值：false
说明
ipfs config命令用来操控配置变量。它非常类似于git config。配置值 保存在IPFS本地仓库中的配置文件。

示例
读取Datastore.Path配置项的值：

$ ipfs config Datastore.Path
设置Datastore.Path配置项的值：

$ ipfs config Datastore.Path ~/.ipfs/datastore
子命令
ipfs config edit           - 在环境变量`$EDITOR`指定的编辑器中编辑配置文件
ipfs config replace <file> - 使用`<file>`指定的文件替代当前配置文件
ipfs config show           - 显示当前配置文件的内容
使用ipfs config <subcmd> --help获取子命令的详细帮助信息。

### ipfs config edit - 编辑配置文件
ipfs config edit - 编辑配置文件
ipfs config edit命令在环境变量$EDITOR指定的编辑器中打开配置文件。

命令行
ipfs config edit
说明
使用ipfs config edit命令之前，需要先设置$EDITOR环境变量，使其指向 你喜欢的文本编辑器。

### ipfs config replace - 替换配置文件
ipfs config replace - 替换配置文件
ipfs config replace <file>命令使用指定的文件替换当前的配置文件。

命令行
ipfs config replace [--] <file>
<file> - 新的配置文件

说明
在执行ipfs config replace命令之前，请保存当前的配置文件，因为该 命令是不可恢复的。

### ipfs config show - 显示配置文件内容
ipfs config show - 显示配置文件内容
ipfs config show命令用来显示当前配置文件的内容。

命令行
ipfs config show
警告
你的私钥保存在配置文件中，因此在ipfs config show命令的输出内容 里也会包含私钥。

