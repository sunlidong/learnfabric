### ipfs repo - 管理ipfs仓库
ipfs repo
使用ipfs repo命令操作IPFS仓库。

命令行
ipfs repo
说明
ipfs repo命令用来操作ipfs仓库。

子命令
ipfs repo fsck    - 移除仓库的锁文件
ipfs repo gc      - 在仓库上执行垃圾回收处理
ipfs repo stat    - 读取当前使用的仓库的统计信息
ipfs repo verify  - 校验仓库中的块是否被破坏
ipfs repo version - 显示仓库版本
使用ipfs repo <subcmd> --help查看子命令的详细帮助信息。

### ipfs repo fsck - 删除仓库锁文件
ipfs repo fsck
使用ipfs repo fsck命令移除仓库的锁文件。

命令行
ipfs repo fsck
说明
ipfs repo fsck命令用来删除仓库和level db数据库的锁文件。只有ipfs 服务进程没有启动时，才可以执行该命令。

### ipfs repo gc - 回收磁盘空间
ipfs repo gc
使用ipfs repo gc命令对仓库执行垃圾回收扫描。

命令行
ipfs repo gc [--quiet | -q] [--stream-errors]
选项
-q,            --quiet bool - 是否仅输出少量信息，默认值：false
--stream-errors        bool - 是否使用错误流，默认值：false
说明
ipfs repo gc命令扫描仓库中的对象，并依照先后顺序移除没有固定的对象， 以便回收磁盘空间。

### ipfs repo stat - 显示仓库统计信息
ipfs repo stat
使用ipfs repo stat命令获取当前仓库的统计信息。

命令行
ipfs repo stat [--human]
选项
--human bool - 是否输出仓库大小，单位MB，默认值：false
说明
ipfs repo stat命令扫描仓库中保存的对象并打印统计信息，输出内容如下：

NumObjects      int  - 本地仓库中的对象数
RepoPath        string - 当前仓库的路径
RepoSize        int - 仓库占用的字节数
Version         string - 仓库版本

### ipfs repo verify - 校验仓库完好性
ipfs repo verify
使用ipfs repo verify命令对仓库的完好性进行校验。

命令行
ipfs repo verify

### ipfs repo version - 显示仓库版本信息
ipfs repo version
使用ipfs repo version命令显示仓库的版本信息。

命令行
ipfs repo version [--quiet | -q]
选项
-q, --quiet bool - 是否输出最少的信息
说明
ipfs repo version返回当前仓库的版本信息。
