# Core Config

#### 介绍

Core Config 是用来决定 Core 行为的配置，一般会保存在 ```/etc/eru/core.yaml``` 也可以通过 ```core --config``` 来指定。一个标准的 core 配置如下：

```
log_level: "DEBUG"                                  决定 core 自身日志等级
bind: ":5001"                                       gRPC 服务绑定地址和端口
statsd: "127.0.0.1:8125"                            资源分配情况将会发送到这个 statsd 地址
image_cache: 2                                      每个 node 上缓存几个版本的镜像
profile: ":12346"                                   profile 端口
global_timeout: 300                                 全局操作超时，目前只用于容器删除的时候加锁防止重复删除
lock_timeout: 30                                    分布式锁超时时间

etcd:                                               ETCD 配置
    machines:
        - "http://127.0.0.1:2379"                   ETCD 地址，支持多机
    prefix: "/core"                                 数据 prefix
    lock_prefix: "core/_lock"                       全局锁的 prefix

git:                                                此处主要用于 Build
    public_key: "***REMOVED***"                     SSH pub key path
    private_key: "***REMOVED***"                    SSH priv key path
    token: "***REMOVED***"                          对 github/gitlab 而言需要在这里填 access token
    scm_type: "github"                              现在支持 github/gitlab

docker:                                             Docker 配置
    log_driver: "json-file"                         默认日志驱动
    network_mode: "bridge"                          默认网络模型
    cert_path: "/etc/eru/tls"                       Docker 的 tls 缓存地址
    hub: "hub.docker.com"                           Registry 地址
    namespace: "projecteru2"                        Registry 的 Namespace
    build_pod: "eru-test"                           采用哪个 Pod 用于 build
    local_dns: true                                 是否用本地 DNS，如果 create 容器的时候没下发 DNS 将会使用这个

scheduler:                                          CPU 优先调度器的配置
    maxshare: -1                                    最大有多少碎片核，-1 表示不限制
    sharebase: 10                                   碎片核最多能分多少份，10表示最小单位是10%，100表示为1%以此类推

syslog:                                             创建的容器如果是 DEBUG 模式则会是用这里的配置
    address: "udp://localhost:5111"                 地址
    facility: "daemon"                              facility 的 prefix
    format: "rfc5424"                               协议格式
```