# Watch Applications

和 K8s 不同, Eru 并不负责 PaaS 的职责, 比方说, K8s 里的 [Deployment] 和 [ReplicaSet] 等上层概念将不会由 Eru 提供.

那么如果我们想做类似的工作该怎么办呢?

### Eru Agent and all its friends

在之前的流程中, 我们添加节点只是在 eru-core 里注册了一个 endpoint, 然而这并不是完整的节点部署, 这样部署首当其中的问题是, 我们不知道容器的 IP:

```
root@localhost:~# eru-cli workload get-status cd7c894aa0736bdf1f884115b18416bbeeca4b23e812a986ae2d86b77a332c2c
┌────┬────────────────┬──────────┬────────────┐
│ ID │ STATUS         │ NETWORKS │ EXTENSIONS │
├────┼────────────────┼──────────┼────────────┤
│    │ Running: false │          │            │
│    │ Healthy: false │          │            │
└────┴────────────────┴──────────┴────────────┘
```

这是因为在节点上缺少一个 agent 服务用来上报给 eru-core. 来部署一个单个节点上的:

```
cat <<! > eru-agent.yaml
appname: "eru"
entrypoints:
  agent:
    cmd: "/usr/bin/eru-agent --hostname node1 --config /etc/eru/agent.yaml"
    restart: always
    publish:
      - "12345"
    healthcheck:
      tcp_ports:
        - "12345"
    privileged: true
volumes:
  - /sys:/sys:ro
  - /var/run/docker.sock:/var/run/docker.sock
  - /proc/:/hostProc/
  - /etc/eru:/etc/eru
!

cat <<! > /etc/eru/agent.yaml
pid: /tmp/eru-agent.sock
core: 127.0.0.1:5001

healthcheck:
  interval: 15
  timeout: 10
  status_ttl: 0
  cache_ttl: 300

docker:
  endpoint: unix:///var/run/docker.sock
metrics:
  step: 15
api:
  addr: 0.0.0.0:12345
log:
  forwards:
    - __discard__
  stdout: True

etcd:
  machines:
    - 127.0.0.1:2379
  prefix: /agent-selfmon
!
eru-cli workload deploy --pod testpod --image projecteru2/agent --entry agent eru-agent.yaml
```

然后再去看容器状态就有了:

```
root@localhost:~# eru-cli workload get-status cd7c894aa0736bdf1f884115b18416bbeeca4b23e812a986ae2d86b77a332c2c
┌──────────────────────────────────────────────────────────────────┬───────────────┬────────────────────┬───────────────────────────────────────────────┐
│ ID                                                               │ STATUS        │ NETWORKS           │ EXTENSIONS                                    │
├──────────────────────────────────────────────────────────────────┼───────────────┼────────────────────┼───────────────────────────────────────────────┤
│ cd7c894aa0736bdf1f884115b18416bbeeca4b23e812a986ae2d86b77a332c2c │ Running: true │ bridge: 172.17.0.7 │ ERU: 1                                        │
│                                                                  │ Healthy: true │                    │ ERU_META: {"Publish":null,"HealthCheck":null} │
└──────────────────────────────────────────────────────────────────┴───────────────┴────────────────────┴───────────────────────────────────────────────┘
```

eru-agent 是 eru 对 dockerd 容器 runtime 提供的节点 daemon, 对于不同的 runtime 有不同的服务, 比如 yavirtd 本身自己也包含了这部分健康监控和状态上报的机制.

实例的健康监控目前支持 http 和 tcp, 在 [spec 的文档里](https://book.eru.sh/specs/app) 可以看到详细的配置.

### Workload Status

workload status 里最重要的几条信息, 包括 IP, running status, healthy status, 而这个状态数据是可以通过 eru-core grpc api 去 watch 监控的.

```
rpc WorkloadStatusStream(WorkloadStatusStreamOptions) returns (stream WorkloadStatusStreamMessage) {};

message WorkloadStatusStreamOptions {
    string appname = 1;
    string entrypoint = 2;
    string nodename = 3;
    map<string, string> labels = 4;
}

message WorkloadStatusStreamMessage {
    string id = 1;
    Workload workload = 2;
    WorkloadStatus status = 3;
    string error = 4;
    bool delete = 5;
}
```

考虑实现类似 Kubernetes 的 ReplicaSet 的服务: 保证集群中的实例数目满足请求, 如果有实例下线就立刻新拉起.

ReplicaSet 的功能可以通过一个更上层的服务, 部署服务之后, 调用上述 `WorkloadStatusStream` 接口去 watch 状态, 如果发现实例挂掉就去补充上新实例; 如果旧的下线实例后来又恢复了则把多的实例清除掉.

通过这个机制可以实现高层的 PaaS 服务, 包括灾难恢复, 自动化迁移, 等上层业务系统.

可以参考 [Workload](https://book.eru.sh/conception/workload) 的文档.

健康检查的详细信息在[这里](https://book.eru.sh/conception/healthcheck)
