### ipfs diag - 系统诊断
ipfs diag
使用ipfs diag命令生成诊断报告。

命令行
ipfs diag
子命令
ipfs diag cmds - 列出当前IPFS节点运行的命令
ipfs diag net  - 生成网络诊断报告
ipfs diag sys  - 打印系统诊断信息
使用ipfs diag <subcmd> --help查看子命令的帮助详情。

### ipfs diag cmds - 显示运行的命令
ipfs diag cmds
使用ipfs diag cmds命令列表显示当前IPFS节点上运行的命令。

命令行
ipfs diag cmds [--verbose | -v]
选项
-v, --verbose bool - 是否显示详细信息，默认值：false
说明
ipfs diag cmds命令列出当前正在运行以及最近运行的命令。

子命令
ipfs diag cmds clear           - 从日志中清除交互请求
ipfs diag cmds set-time <time> - 设置非活动请求在日志中的保留时长
使用ipfs diag cmds <subcmd> --help查看子命令的帮助详情。

### ipfs diag cmds clear - 清理非活动请求
ipfs diag cmds clear
使用ipfs diag cmds clear命令清理日志中的非活动请求。

命令行
ipfs diag cmds clear

### ipfs diag cmds set-time - 设置非活动请求的保存时长
ipfs diag cmds set-time
使用ipfs diag cmds set-time <time>命令设置非活动请求在日志中的保存时长。

命令行
ipfs diag cmds set-time [--] <time>
<time> - 非活动请求在日志中的保存时长。

### ipfs diag sys - 显示系统信息
ipfs diag sys
使用ipfs diag sys命令显示系统诊断信息。 - Print system diagnostic information.

命令行
ipfs diag sys
说明
ipfs diag sys命令显示计算机的系统信息，以便调试。

