### ipfs stats - 显示ipfs节点统计信息
ipfs stats
使用ipfs stats命令查询ipfs的统计信息。

命令行
ipfs stats
说明
ipfs stats命令系列用来查看ipfs节点旳统计信息。

子命令
ipfs stats bitswap - 显示bitswap代理的诊断信息
ipfs stats bw      - 打印ipfs带宽利用信息
ipfs stats repo    - 获取当前仓库的统计信息
使用ipfs stats <subcmd> --help命令查看子命令的帮助详情。

### ipfs stats bitswap - 显示bitswap协议统计信息
ipfs stats bitswap
使用ipfs stats bitswap命令显示bitswap代理的诊断信息。

命令行
ipfs stats bitswap

### ipfs stats bw - 显示带宽利用信息
ipfs stats bw
使用ipfs stats bw命令打印ipfs的带宽利用信息。

命令行
ipfs stats bw [--peer=<peer> | -p] [--proto=<proto> | -t] [--poll] [--interval=<interval> | -i]
选项
-p,   --peer     string - 指定一个对端节点
-t,   --proto    string - 指定一个协议
--poll           bool   - 是否定时显示带宽利用信息，默认值：false
-i,   --interval string - 定时显示的间隔时间，仅当使用`--poll`选项时有效
                          可以使用不同的时间单位来指定定时间隔，例如"300s", "1.5h" 或"2h45m"。
                          有效的时间单位包括："ns", "us" (or "µs"), "ms", "s", "m", "h"。
                          默认值: 1s
说明
ipfs stats bw命令输出ipfs服务进程的带宽利用信息。显示内容包括：入流量、出流量、入流速和出流速。

默认情况下，显示的是所有协议的带宽利用总量。可以使用--peer选项限定显示与某一 对端节点之间的带宽利用情况。也可以使用--proto选项限定仅显示某一特定协议的带宽 利用情况。这两个选项不可以同时使用。

--proto选项的可选协议如下：

/ipfs/id/1.0.0
/ipfs/bitswap
/ipfs/dht
可以访问libp2p 进一步了解可选协议。

示例
> ipfs stats bw -t /ipfs/bitswap
Bandwidth
TotalIn: 5.0MB
TotalOut: 0B
RateIn: 343B/s
RateOut: 0B/s
> ipfs stats bw -p QmepgFW7BHEtU4pZJdxaNiv75mKLLRQnPi1KaaXmQN4V1a
Bandwidth
TotalIn: 4.9MB
TotalOut: 12MB
RateIn: 0B/s
RateOut: 0B/s

### ipfs stats repo - 显示仓库统计信息
ipfs stats repo
使用ipfs stats repo命令获取当前仓库的统计信息。

命令行
ipfs stats repo [--human]
选项
--human bool - 是否输出仓库大小，单位MiB，默认值：false
说明
ipfs repo stat命令扫描本地仓库中存储的对象并打印统计信息。输出内容 如下：

NumObjects      int - 本地仓库中的对象
RepoPath        string - 当前仓库的路径
RepoSize        int - 当前仓库占用空间的字节数
Version         string - 仓库版本
