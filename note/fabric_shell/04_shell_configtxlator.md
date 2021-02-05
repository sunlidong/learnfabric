configtxlator
### configtxlator

configtxlator
configtxlator命令用来将fabric的数据结构在protobuf和JSON 之间进行转换，也可以用来创建配置更新。该命令可以启动一个REST 服务来通过HTTP暴露服务接口，也可以直接在命令行使用。

命令语法
configtxlator工具有5个子命令，列举如下：

start
proto_encode
proto_decode
compute_update
version

### configtxlator start
configtxlator start
启动configtxlator的REST服务。

使用方法
~$ configtxlator start [<flags>]
命令标志：

  --help                Show context-sensitive help (also try --help-long and
                        --help-man).
  --hostname="0.0.0.0"  The hostname or IP on which the REST server will listen
  --port=7059           The port on which the REST server will listen
示例代码
下面的示例在本地8080端口启动rest服务：

~$ configtxlator start --hostname="127.0.0.1" --port=8080

### configtxlator proto_encode

configtxlator proto_encode
将一个JSON文档转换为protobuf格式数据。

使用方法
~$ configtxlator proto_encode --type=TYPE [<flags>]
命令标志：

  --help                Show context-sensitive help (also try --help-long and
                        --help-man).
  --type=TYPE           The type of protobuf structure to encode to. For
                        example, 'common.Config'.
  --input=/dev/stdin    A file containing the JSON document.
  --output=/dev/stdout  A file to write the output to.
示例代码
下面的示例将控制台输入的json格式的policy，转换为protobuf格式并存入文件policy.pb：

~$ configtxlator proto_encode --type common.Policy --output policy.pb
在启动rest服务后，下面的示例使用curl命令通过rest api执行同样的操作：

~$ curl -X POST --data-binary /dev/stdin "${CONFIGTXLATOR_URL}/protolator/encode/common.Policy" > policy.pb

### configtxlator proto_decode

configtxlator proto_decode
将一个protobuf消息转换为JSON格式。

使用方法
~$ configtxlator proto_decode --type=TYPE [<flags>]
命令标志：

  --help                Show context-sensitive help (also try --help-long and
                        --help-man).
  --type=TYPE           The type of protobuf structure to decode from. For
                        example, 'common.Config'.
  --input=/dev/stdin    A file containing the proto message.
  --output=/dev/stdout  A file to write the JSON document to.
示例代码
下面的示例将名为fabric_block.pb的区块转换为json格式并在stdout输出：

~$ configtxlator proto_decode --input fabric_block.pb --type common.Block
在启动rest服务后，下面的示例使用curl命令通过rest api执行同样的操作：

~$ curl -X POST --data-binary @fabric_block.pb "${CONFIGTXLATOR_URL}/protolator/decode/common.Block"
### configtxlator compute_update

计算生成两个common.Config消息之间的配置更新。

使用方法
~$ configtxlator compute_update --channel_id=CHANNEL_ID [<flags>]
命令标志：

  --help                   Show context-sensitive help (also try --help-long and
                           --help-man).
  --original=ORIGINAL      The original config message.
  --updated=UPDATED        The updated config message.
  --channel_id=CHANNEL_ID  The name of the channel for this update.
  --output=/dev/stdout     A file to write the JSON document to.
示例代码
下面的示例计算从original_config.pb到modified_config.pb的配置更新，并在stdou输出JSON解码后的数据：

~$ configtxlator compute_update --channel_id testchan --original original_config.pb --updated modified_config.pb | configtxlator proto_decode --type common.ConfigUpdate
在启动rest服务后，下面的示例使用curl命令通过rest api执行同样的操作：

~$ curl -X POST -F channel=testchan -F "original=@original_config.pb" -F "updated=@modified_config.pb" "${CONFIGTXLATOR_URL}/configtxlator/compute/update-from-configs" | curl -X POST --data-binary /dev/stdin "${CONFIGTXLATOR_URL}/protolator/encode/common.ConfigUpdate"
### configtxlator version

configtxlator version
显示软件版本信息。

使用方法
~$ configtxlator version
命令标志：

  --help  Show context-sensitive help (also try --help-long and --help-man).