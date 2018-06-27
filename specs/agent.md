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
    - 127.0.0.1:8125                            可以配置多个 statsd 后端
api:                                            profile API 配置
  addr: 127.0.0.1:12345                         profile API 地址
log:                                            日志选项
  forwards:
    - tcp://127.0.0.1:5144                      日志远端
  stdout: False                                 是否把容器日志打到 agent 自身日志流里面
```