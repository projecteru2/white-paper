# Agent Config

#### 介绍

Agent Config 是用来决定 Agent 行为的配置，一般会保存在 ```/etc/eru/agent.yaml``` 也可以通过 ```agent --config``` 来指定。一个标准的 agent 配置如下：

```
pid: /tmp/agent.pid                              pid path
store: grpc                             optional store 的类型，默认为 grpc，即 eru-core 的 grpc 服务
runtime: docker                         optional runtime 类型，默认为 docker
kv: etcd                                optional kv 类型，默认为 ETCD

core:                                            Core 的 API 地址
  - 127.0.0.1:5001                               可以配置多个地址

healthcheck:                                     Health Check 配置
  interval: 120                         optional Health Check 间隔
  timeout: 10                           optional Health Check 超时判定
  cache_ttl: 300                        optional 本地缓存的过期时间，仅当 enable_selfmon 为 true 时生效
  enable_selfmon: false                 optional 集群里是否启动了 selfmon

auth:                                   optional Core API Basic auth
  username: username
  password: password

docker:                                          Docker 配置
  endpoint: unix:///var/run/docker.sock          Docker url，支持 unix socket file 地址和 tcp 地址的形式

yavirt:                                          Yavirt 配置
  endpoint: grpc://127.0.0.1:9697                Yavirt url，支持 grpc 地址和 http 地址
  skip_guest_report_regexps:                     ID 符合任意正则表达式的 VM 将不会被检查
    - .+002

metrics:                                         Metrics 配置
  step: 30                                       发送间隔
  transfers:                            optional 默认支持 prometheus，也可以开启 statsd
    - 127.0.0.1:8125                             可以配置多个 statsd 后端

api:                                    optional profile API 配置
  addr: 127.0.0.1:12345                          profile API 地址

log:                                             日志选项
  forwards:
    - tcp://127.0.0.1:5144              optional 日志远端, 支持 tcp、udp、journal 和特殊的 __discard__
  stdout: False                                  是否把容器日志打到 agent 自身日志流里面

ha_keepalive_interval: 16s              optional selfmon 发送心跳的时间间隔

etcd:                                            ETCD 配置，仅当此 agent 以 selfmon 模式运行时生效
  machines:                                      ETCD 服务器地址
    - 127.0.0.1:2379
  prefix: /agent-selfmon                         Key的前缀
```

Agent 在监听到隶属 Eru 的容器启动后，会把 stdout/stderr 的日志 forward 到配置的远端，同时开始监听 metrics。

对于日志，格式如下：

```
{"id":"{CONTAINER_ID}","name":"{APP_NAME}","type":"{stderr || stdout}","entrypoint":"{APP_ENTRYPOINT}","ident":"{APP_IDENT}","data":"{LOG_MSG}","datetime":"2018-06-27 07:19:25.507018","extra":{CONTAINER_LABELS}}
```

在测试的时候甚至可以用 ```netcat``` 来模拟日志接收，使用

```
nc -l {PORT} -k
```

需要注意的是，log 参数支持 tcp 和 udp 2种形式，udp 对于单条日志大小限制为了1024K，因此对于线上系统而言，建议使用 TCP 模式。

对于 Metrics，Agent 会发一组数据到配置的远端 statsd 中：

```
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cpu_usage:0.06814310051085702|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cpu_user_usage:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cpu_sys_usage:40.00000000003638|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.mem_rss:6184960|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.mem_usage:2088960|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.mem_max_usage:2215936|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.mem_usage_percent:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.lo.drop.in:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.lo.drop.out:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.lo.err.in:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.lo.err.out:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.lo.bytes.recv:10445.6|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.lo.bytes.sent:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.lo.packets.recv:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.lo.packets.sent:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cali0.drop.in:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cali0.drop.out:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cali0.err.in:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cali0.err.out:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cali0.bytes.recv:5191.333333333333|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cali0.bytes.sent:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cali0.packets.recv:0|g
eru.{APPNAME}.{TAG}.{ENTRYPOINT}.{HOST}.{CONTAINER_ID}.cali0.packets.sent:0|g
```

通过这个数据我们就可以从容器外面监控容器行为。

如果没有开启 statsd，也可以通过 `API_HOST:PORT/metrics` 拿到 prometheus 规格的 metrics。