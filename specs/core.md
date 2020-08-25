# Core Config

#### 介绍

Core Config 是用来决定 Core 行为的配置，一般会保存在 ```/etc/eru/core.yaml``` 也可以通过 ```core --config``` 来指定。一个标准的 core 配置如下：

```
log_level: "DEBUG"                                  决定 core 自身日志等级
bind: ":5001"                                       gRPC 服务绑定地址和端口
lock_timeout: 30s                                   分布式锁超时时间
global_timeout: 300s                                全局操作超时，目前只用于容器删除的时候加锁防止重复删除
statsd: "127.0.0.1:8125"               optional     core 的状态将会发送到这个 statsd 地址
profile: ":12346"                      optional     profile 端口
cert_path: "/tmp"                      optional     临时存储 certs 文件地址

auth:                                  optional     gRPC basic auth
    username: "user"
    password: "password"

grpc:                                  optional     gRPC 相关配置
    max_concurrent_streams: 100
    max_recv_msg_size: 20971520
    service_discovery_interval: 15s
    service_heartbeat_interval: 15s

git:                                   optional     此处主要用于 Build
    scm_type: "github"                              现在支持 github/gitlab
    private_key: "***REMOVED***"                    SSH priv key path
    token: "***REMOVED***"                          对 github/gitlab 而言需要在这里填 access token
    clone_timeout: 300s                             clone 超时时间

etcd:                                               ETCD 配置
    machines:
        - "http://127.0.0.1:2379"                   ETCD 地址，支持多机
    prefix: "/core"                                 数据 prefix
    lock_prefix: "core/_lock"                       全局锁的 prefix
    ca: ""                             optional     ca
    key: ""                            optional     key
    cert: ""                           optional     cert
    auth:                              optional     etcd auth
        username: "user"
        password: "password"

docker:                                             Docker 配置
    version: "1.32"                                 Docker API Version
    network_mode: "bridge"                          默认网络模型
    hub: "hub.docker.com"              optional     Registry 地址
    namespace: "projecteru2"           optional     Registry 的 Namespace
    build_pod: "eru-test"              optional     采用哪个 Pod 用于 build
    local_dns: true                    optional     是否用本地 DNS，如果 create 容器的时候没下发 DNS 将会使用这个
    log:                               optional     默认日志驱动
        type: "journald"
        config:
            config1: "value"
    auths:                                          Push 和 Pull 验证，core 支持配置多个 registry
      hub.docker.com:
        username: "user1"
        password: "password1"
      hub2.docker.com:
        username: "user2"
        password: "password2"
      ...

scheduler:                                          调度器的配置
    maxshare: -1                                    最大有多少碎片核，-1 表示不限制
    sharebase: 10                                   碎片核最多能分多少份，10表示最小单位是10%，100表示为1%以此类推

virt:                                               yavirt api 版本
    version: "v1"

systemd:
    username: "root"                                systemd 引擎配置
```
