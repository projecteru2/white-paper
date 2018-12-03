# Agent Config

#### 介绍

Agemt Config 是用来决定 Agent 行为的配置，一般会保存在 ```/etc/eru/agent.yaml``` 也可以通过 ```agent --config``` 来指定。一个标准的 agent 配置如下：

```
health_check_interval: 5                        healthcheck 间隔
health_check_timeout: 10                        healthcheck 超时判定
core: 127.0.0.1:5001                            Core 的 API 地址

docker:                                         Docker 配置
  endpoint: unix:///var/run/docker.sock         Docker sockfile 地址
metrics:                                        Metrics 配置
  step: 30                                      发送间隔
  transfers:
    - 127.0.0.1:8125                            可以配置多个 statsd 后端，做负载均衡分片，目前仅支持`udp`协议的statsd，但如果挂掉其中一个地址，将会`持续`丢失这个分片监控
api:                                            profile API 配置
  addr: 127.0.0.1:12345                         profile API 地址
log:                                            日志选项
  forwards:
    - tcp://127.0.0.1:5144                      日志远端，做负载均衡分片，挂掉一个地址将会`持续`丢失这个分片的日志
  stdout: False                                 是否把容器日志打到 agent 自身日志流里面
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

