### ipfs object - 管理ipfs对象
ipfs object
使用ipfs object命令操作IPFS对象。

命令行
ipfs object
说明
ipfs object命令用来直接操作DAG对象。 directly.

子命令
ipfs object data <key>           - 输出ipfs对象的裸数据
ipfs object diff <obj_a> <obj_b> - 显示两个ipfs对象的差异
ipfs object get <key>            - 读取名称为`<key>`的DAG节点并进行序列化
ipfs object links <key>          - 输出指定对象指向的链接
ipfs object new [<template>]     - 使用给定的ipfs模版创建一个新的对象
ipfs object patch                - 基于现有对象创建一个新的DAG对象
ipfs object put <data>           - 将输入保存为DAG对象，并输出显示生成的密钥
ipfs object stat <key>           - 读取名称为`<key>`的DAG节点旳统计信息
使用ipfs object <subcmd> --help查看子命令的详细帮助信息。

### ipfs object data - 显示对象裸数据
ipfs object data
使用ipfs object data <key>命令输出指定ipfs对象的原始字节流。

命令行
ipfs object data [--] <key>
<key> - 要读取对象的键，base58编码，multihash格式

说明
ipfs object data命令用来读取DAG节点中保存的原始数据，并输出到 标准输出设备。参数<key>是base58编码的multihash哈希。

注意，--encoding选项不会影响输出，因为输出的是对象的原始数据。

### ipfs object diff - 显示对象差异
ipfs object diff
使用ipfs object diff <obj_a> <obj_b>命令显示两个ipfs对象的差异。

命令行
ipfs object diff [--verbose | -v] [--] <obj_a> <obj_b>
参数：

<obj_a> - 参与比较的对象a
<obj_b> - 参与比较的对象b
选项
-v, --verbose bool - 是否显示详细信息
说明
ipfs object diff命令可以显示两个ipfs对象的差异。

示例:

> ls foo
bar baz/ giraffe
> ipfs add -r foo
...
Added QmegHcnrPgMwC7tBiMxChD54fgQMBUecNw9nE9UUU4x1bz foo
> OBJ_A=QmegHcnrPgMwC7tBiMxChD54fgQMBUecNw9nE9UUU4x1bz
> echo "different content" > foo/bar
> ipfs add -r foo
...
Added QmcmRptkSPWhptCttgHg27QNDmnV33wAJyUkCnAvqD3eCD foo
> OBJ_B=QmcmRptkSPWhptCttgHg27QNDmnV33wAJyUkCnAvqD3eCD
> ipfs object diff -v $OBJ_A $OBJ_B
Changed "bar" from QmNgd5cz2jNftnAHBhcRUGdtiaMzb5Rhjqd4etondHHST8 to QmRfFVsjSXkhFxrfWnLpMae2M4GBVsry6VAuYYcji5MiZb.

### ipfs object get - 格式化显示对象数据
ipfs object get
使用ipfs object get <key>命令读取并序列化指定的DAG节点。

命令行
ipfs object get [--] <key>
<key> - 要读取对象的键，base58编码，multihash格式

说明
ipfs object get命令用来提取指定DAG节点旳内容。使用--encoding 选项声明序列化的格式。该命令在标准输出设备stdout上显示输出，参数 <key>是base58编码的multihash哈希。

ipfs object get支持以下序列化编码格式：

"protobuf"
"json"
"xml"
使用--encoding或--enc选项来声明上述编码格式。

### ipfs object links - 显示对象的链接
ipfs object links
使用ipfs object links <key>命令输出指定对象的链接对象。

命令行
ipfs object links [--headers | -v] [--] <key>
<key> - 对象的键，base58编码，multihash格式

选项
-v, --headers bool - 是否打印表头，例如哈希、大小、名称，默认值：false
说明
ipfs object links命令用来提取一个指定DAG节点旳链接对象，并将结果 在标准输出设备上列表显示，参数<key>为base58编码。

### ipfs object new - 创建新对象
ipfs object new
使用ipfs object new [<template>]命令可以基于模板创建一个新的ipfs对象。

命令行
ipfs object new [--] [<template>]
[<template>] - 要使用的模板，可选

说明
ipfs object new命令用来创建新的DAG节点。默认情况下它创建并返回一个空的节点， 但是可以传入一个可选的模板来创建有预定格式的节点。

目前可用的模板为：

unixfs-dir

### ipfs object patch - 派生新对象
ipfs object patch
使用ipfs object patch命令可以基于已有的对象创建一个新的DAG对象。

命令行
ipfs object patch
说明
ipfs object patch <root> <cmd> <args>命令用来创建定制DAG对象。该 命令修改已有的对象，然后得到一个新的对象，可以认为它是DAG版本的ipfs对象 修改方法。

子命令
ipfs object patch add-link <root> <name> <ref> - 为指定对象添加一个链接
ipfs object patch append-data <root> <data>    - 向DAG节点旳数据段追加数据
ipfs object patch rm-link <root> <link>        - 删除对象的指定链接
ipfs object patch set-data <root> <data>       - 设置IPFS对象的数据字段
使用ipfs object patch <subcmd> --help查看子命令的详细帮助信息。

### ipfs object patch add-link - 添加链接
ipfs object patch add-link
使用ipfs object patch add-link <root> <name> <ref>命令为指定对象添加一个新的链接。

命令行
ipfs object patch add-link [--create | -p] [--] <root> <name> <ref>
参数：

<root> - 要修改的节点哈希
<name> - 要创建的链接名称
<ref>  - 要链接的IPFS对象
选项
-p, --create bool - 是否创建中间节点，默认值：false
说明
ipfs object patch add-link命令为指定对象添加一个默克尔链接，并返回结果哈希。

示例
下面代码创建一个新的空目录，然后再其中添加一个名为foo的链接，该链接 指向一个内容为bar的文件，然后返回新对象的哈希：

$ EMPTY_DIR=$(ipfs object new unixfs-dir)
$ BAR=$(echo "bar" | ipfs add -q)
$ ipfs object patch $EMPTY_DIR add-link foo $BAR

### ipfs object patch append-data - 追加新数据
ipfs object patch append-data
使用ipfs object patch append-data <root> <data>命令向DAG节点旳数据段追加数据。

命令行
ipfs object patch append-data [--] <root> <data>
参数：

<root> - 要修改的节点哈希
<data> - 要追加的数据
说明
ipfs object patch append-data命令向指定对象的数据段追加新的数据。

示例
$ echo "hello" | ipfs object patch $HASH append-data
注意，上面的命令不会将数据写入文件 —— 它直接修改DAG对象的裸数据。一个对象 最多可保存1MB的数据，大于该尺寸的对象不会被网络接受。

### ipfs object patch rm-link - 删除链接
ipfs object patch rm-link
使用ipfs object patch rm-link <root> <link>方法移除一个对象的指定链接。

命令行
ipfs object patch rm-link [--] <root> <link>
参数：

<root> - 要修改的节点
<link> - 要移除的链接
说明
ipfs object patch rm-link命令从节点移除一个指定的链接。

### ipfs object patch set-data - 更新节点数据
ipfs object patch set-data
使用ipfs object patch set-data <root> <data>命令修改指定IPFS对象的数据字段。

命令行
ipfs object patch set-data [--] <root> <data>
参数：

<root> - 要修改的节点
<data> - 要设置的新数据
说明
ipfs object patch set-data命令读取标准输入stdin，然后用输入内容更新节点。

例如：

$ echo "my data" | ipfs object patch $MYHASH set-data

### ipfs object put - 将数据转化为ipfs对象
ipfs object put
使用ipfs object put <data>命令将输入内容保存为DAG对象，并输出该对象的键。

命令行
ipfs object put [--inputenc=<inputenc>] [--datafieldenc=<datafieldenc>] [--] <data>
<data> - 要保存的数据

选项
--inputenc     string - 输入数据的编码类型，有以下可选值：`protobuf`、 `json`，默认值： json
--datafieldenc string - 数据字段的编码类型，可以是`text`或`base64`，默认值：text
说明
ipfs object put命令利用输入创建一个新的对象，输出为base58编码的multihash值。

使用--inputenc选项设置输入数据的编码格式，可选以下值：

"protobuf"
"json" (default)
示例：

$ echo '{ "Data": "abc" }' | ipfs object put
如上命令使用数据abc创建一个新的不包含链接的节点。要创建一个包含链接的 对象，需要先创建一个文件，例如node.json，内容如下：

{
    "Data": "another",
    "Links": [ {
        "Name": "some link",
        "Hash": "QmXg9Pp2ytZ14xgmQjYEiHjVjMFXzCVVEcRTWJBmLgR39V",
        "Size": 8
    } ]
}
然后执行以下命令：

$ ipfs object put node.json

### ipfs object stat - 显示节点统计信息
ipfs object stat
使用ipfs object stat <key>命令读取指定DAG节点旳统计信息。

命令行
ipfs object stat [--] <key>
<key> - 要读取统计信息的节点的键，base58编码，multihas格式

说明
ipfs object stat命令可以输出显示DAG节点旳统计信息。 <key>参数为base58编码的哈希，格式为multihash。该命令输出到 标准输出设备stdout，内容如下：

NumLinks        int - 链接表中的链接数
BlockSize       int - 数据块大小
LinksSize       int - 链接段大小
DataSize        int - 数据段大小
CumulativeSize  int - 数据及链接的累计大小
