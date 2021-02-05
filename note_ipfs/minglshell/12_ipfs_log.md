### ipfs log - 日志管理
ipfs log ls
使用ipfs log ls命令列出所有的日志子系统。

命令行
ipfs log ls
说明
ipfs log ls命令用来列出一个运行中的服务进程的所有日志 子系统。

### ipfs log level - 调整日志等级
ipfs log level
使用ipfs log level <subsystem> <level>命令修改日志等级。

命令行
ipfs log level [--] <subsystem> <level>
参数：

<subsystem> - 子系统日志标识符，all表示所有子系统
<level> - 日志等级，debug将显示最多的信息，critical将显示最少的信息。 可选以下值之一：debug, info, warning, error, critical。
说明
ipfs log level修改指定的一个或全部子系统的日志输出等级。该命令不影响事件日志。


### ipfs log tail - 跟踪显示事件日志
ipfs log tail
使用ipfs log tail命令跟踪读取并显示事件日志。

命令行
ipfs log tail
说明
ipfs log tail命令可以在事件生成时输出显示事件日志消息。

### ipfs log ls - 列举日志子系统
ipfs log ls
使用ipfs log ls命令列出所有的日志子系统。

命令行
ipfs log ls
说明
ipfs log ls命令用来列出一个运行中的服务进程的所有日志 子系统。
