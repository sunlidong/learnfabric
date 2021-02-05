CORE_CLI_ADDRESS - 设置链码回调地址
设置客户端进程用来接受链码回调的地址。

默认值
CORE_CLI_ADDRESS的默认值为：0.0.0.0:30304

对应的配置文件
core.yaml

cli:
  cli_address: 0.0.0.0:30304


  #
CORE_REST_ENABLED - 启用/禁用REST服务
启用/禁用节点的REST服务。建议在生产环境中禁用验证节点 的REST服务，仅使用非验证节点提供REST服务。

默认值
CORE_REST_ENABLED的默认值为：true

对应的配置文件
core.yaml

rest:
  enabled: true


####
CORE_REST_ADDRESS - REST服务监听地址
节点若启用REST服务，则该环境变量指定REST服务的监听地址。

默认值
CORE_REST_ADDRESS的默认值为：0.0.0.0:5000。

对应的配置文件
core.yaml

rest:
  address: 0.0.0.0:5000