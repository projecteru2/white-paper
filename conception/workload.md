# Workload

### 概念

Workload 在 Eru 里代表一个最终部署实例抽象, 在不同的 [runtime](todu) 上可能代表容器/虚拟机/daemon 进程.

### 操作

Workload 拥有全局唯一到 ID, 通过 ID 可以完成一系列通用操作:

1. Create
2. Stop
3. Start
4. Remove
5. Execute: 在容器 namespace 里, 或者 VM 内部, 执行指定的命令
6. Realloc: 动态调整 workload 的资源配额. 详见 [资源](https://book.eru.sh/conception/resource)
7. Replace: 删除旧实例, 原地起新实例, 可继承网络配置
8. Copy: 从实例里拷贝指定文件到客户端
9. Send: 从客户端拷贝指定文件到实例内部

### 生命周期

Eru Workload 可以在创建时指定 `after_start` 和 `before_stop` hook, 可以用来完成一系列应用初始化和清理的工作. 详见 [spec](https://book.eru.sh/specs/app).

Eru Workload 还可以在创建时指定 `healthcheck` 来定义应用的健康状态, 可以定义 tcp ports 和 http url 作为检测方式. 详见 [spec](https://book.eru.sh/specs/app).

Workload 本身的状态由节点上的 eru-agent 监控和上报给 eru-core, 上报信息包括:

1. running: 实例是否存活
2. healthy: 在 `healthcheck` 定义下的实例是否健康
3. networks: 实例 IP
4. extension: 其他 tags

eru-core 提供了一个接口 `WorkloadStatusStream` 供第三方监听 Workload 的生命状态, 一旦有状态变化会推送消息给监听方, 从而完成 failover 等机制.
