### ipfs files - 文件操作接口
ipfs files
ipfs files命令用来操作unixfs文件。

命令行
ipfs files [--f=false]
选项
-f, --flush bool - 是否在写文件后自动刷新目标及其父节点，默认值：true
说明
ipfs files命令族允许我们像操作unix文件系统一样操作IPFS对象。

注意，绝大多数的ipfs files子命令都支持--flush选项，其默认值为true。 如果将该选项设置为false，虽然可以提高大量文件操作时的性能，但代价是 放弃了一致性保证，如果服务进程在运行ipfs files flush命令之前被意外杀掉， 有可能丢失数据。因此请慎重考虑是否将该选项设置为false。

类似的，并发的ipfs repo gc命令如果使用--flush=false也存在潜在 的数据丢失问题。

子命令
ipfs files cp <source> <dest>  - 将文件拷贝到mfs中
ipfs files flush [<path>]      - 将指定路径的数据刷新到磁盘
ipfs files ls [<path>]         - 列表显示本地可变命名空间中的目录
ipfs files mkdir <path>        - 创建目录
ipfs files mv <source> <dest>  - 移动文件
ipfs files read <path>         - 读取指定mfs中的文件
ipfs files rm <path>...        - 删除指定文件
ipfs files stat <path>         - 显示文件状态
ipfs files write <path> <data> - 写入指定文件系统中的可变文件
可以使用ipfs files <subcmd> --help来获取子命令的详细帮助信息。

### ipfs files cp - 拷贝文件
ipfs files cp
ipfs files cp <source> <dest>命令将指定的文件拷贝到mfs中。

命令行
ipfs files cp [--] <source> <dest>
参数：

<source> - 要拷贝的源对象
<dest> - 拷贝目标

### ipfs files flush - 刷新磁盘文件
ipfs files flush
ipfs files flush [<path>]命令将指定路径的数据刷新到磁盘。

命令行
ipfs files flush [--] [<path>]
[<path>] - 要刷新的数据的路径，默认值：/。

说明
ipfs files flush命令负责将指定的路径刷新到磁盘。仅当其他ipfs files 命令以--flush=false选项执行时才需要使用该命令。

### ipfs files ls - 显示目录内容
ipfs files ls
ipfs files ls [<path>]命令列表显示本地可变命名空间中的目录内容。

命令行
ipfs files ls [-l] [--] [<path>]
[<path>] - 要显示其目录列表的路径，默认值：/

选项
-l bool - 是否使用加长显示格式。
说明
ipfs files ls命令用来显示本地可变命名空间中的目录内容。

例如，以下代码显示/welcome/docs目录的内容：

$ ipfs files ls /welcome/docs/
about
contact
help
quick-start
readme
security-notes
类似的，以下代码列表显示/myfiles/a/b/c/d目录的内容：

$ ipfs files ls /myfiles/a/b/c/d
foo
bar

### ipfs files mkdir - 创建目录
ipfs files mkdir
ipfs files mkdir <path>命令用来创建指定的目录。

命令行
ipfs files mkdir [--parents | -p] [--] <path>
<path> - 要创建的目录路径。

选项
-p, --parents bool - 是否根据需要自动创建父目录
说明
ipfs files mkdir命令可以创建指定路径的目录，如果该目录不存在的话。

注意，必须使用绝对路径。

例如：

$ ipfs mfs mkdir /test/newdir
$ ipfs mfs mkdir -p /test/does/not/exist/yet

### ipfs files mv - 移动/更名文件
ipfs files mv
ipfs files mv <source> <dest>命令用来移动指定文件到目标路径。

命令行
ipfs files mv [--] <source> <dest>
参数：

<source> - 要移动的源文件
<dest> - 目标路径
说明
ipfs files mv命令的作用是移动文件，类似于传统的unix命令mv。

例如，将文件/myfs/a/b/c更名为/myfs/foo/newc：

$ ipfs files mv /myfs/a/b/c /myfs/foo/newc

### ipfs files read - 读取文件
ipfs files read
ipfs files read <path>命令读取指定文件的内容。

命令行
ipfs files read [--offset=<offset> | -o] [--count=<count> | -n] [--] <path>
<path> - 要读取文件的路径

选项
-o, --offset int - 读取起始位置相对于文件头的字节偏移量
-n, --count  int - 要读取的字节数
说明
ipfs files read命令可以读取文件指定偏移位置开始、指定数量的数据。默认 情况下，该命令将读取整个文件的内容，类似于unix中的cat命令。

示例：

$ ipfs files read /test/hello
hello

### ipfs files rm - 删除文件
ipfs files rm
ipfs files rm <path>...命令删除指定的文件。

命令行
ipfs files rm [--recursive | -r] [--] <path>...
<path>... - 要删除的文件

选项
-r, --recursive bool - 是否递归删除目录
说明
ipfs files rm命令可以删除指定的文件或目录。

例如：

$ ipfs files rm /foo
$ ipfs files ls /bar
cat
dog
fish
$ ipfs files rm -r /bar

### ipfs files stat - 显示文件统计信息
ipfs files stat
ipfs files stat <path>命令显示指定文件的统计信息。

命令行
ipfs files stat [--format=<format>] [--hash] [--size] [--] <path>
<path> - 要显示其统计信息的文件路径

选项
--format string - 统计信息的显示格式，支持以下字段标志：`<hash> <size> <cumulsize> <type> <childs>`。默认值为：`<hash>`
--hash   bool   - 是否只显示哈希，该选项自动启用`--format=<hash>`，默认值：false
--size   bool   - 是否只显示哈希，该选项自动启用`--format=<cumulsize>`，默认值：false

### ipfs files write - 写入文件

ipfs files write
ipfs files write <path> <data>命令将数据写入指定的文件。

命令行
ipfs files write [--offset=<offset> | -o] [--create | -e] [--truncate | -t] [--count=<count> | -n] [--] <path> <data>
参数：

<path> - 要写入的文件路径
<data> - 要写入的数据内容
选项
-o, --offset   int  - 开始写入的文件内字节偏移量
-e, --create   bool - 如果目标文件不存在，是否自动创建
-t, --truncate bool - 是否在写入数据之前将文件清零
-n, --count    int  - 要写入的最大字节数
说明
ipfs files write命令将指定的数据写入指定的文件。该命令允许指定数据写入位置。

如果使用了--create选项，那么当目标文件不存在时将自动创建，但该命令无法自动创建 目标文件路径中不存在的中间路径部分。

如果在执行ipfs files write命令时设置--flush=false选项，那么对文件的修改将 不会传播到有向图的根节点，当执行大量写操作或写入深层目录时可以明显提升速度。

示例：

echo "hello world" | ipfs files write --create /myfs/a/b/file
echo "hello world" | ipfs files write --truncate /myfs/a/b/file
警告：

使用--flush=false选项时，在树刷新之前将无法保证数据一致性。可以在文件或其父 对象上执行ipfs files flush命令来执行刷新操作。