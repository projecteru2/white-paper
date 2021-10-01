# Selfmon

### 简介

曾经每个 Eru-agent 都要不断地向 Core 上报 Workload 的状态，随后 Core 会把这些信息存入 ETCD，这样给 ETCD 造成了很大的写入压力。为了解决这个问题，我们设计了 Selfmon。

在引入 Selfmon 后，每次上报 Workload Status 之前，Agent 会和本地缓存中的 status 作对比，如果发生变化才会上报到 Core。并且上报时不再设置 TTL，即 ETCD 里对应的 Key 不会过期。

不再设置 TTL 产生的问题就是：如果 Agent 意外退出了，就无法上报 Workload Status，导致会有一部分 Workload 在 Eru 看来永远处于 Running & Healthy 状态。

所以 Selfmon 加入了监控 Node Status 的功能。每个 Agent 会定时上报自己所在 Node 的状态（带TTL），而 Selfmon 会持续监听这些 Node Status 的变化，并且做对应的处理。

如果某个 Agent 意外退出了，它最后一次上报的 Node Status 会在一段时间后过期。Selfmon 监听到了这个过期事件，就会调用 Core 的相关接口，把这个 Node 和上面所有的 Workload 标记为不可用。

Agent 启动时，Selfmon 则会调用 Core 的接口把这个 Node 设置为 Available，由 Agent 自己重新检查并上报 Workload Status。

Selfmon 支持 HA，多个 selfmon 同时只有一个会拿到分布式锁并且正常运行。可以在集群里部署多个 selfmon。

### 流程

下文中会提到 `Node Status` 和 `Node Info`，先解释一下它们的区别。

`Node Info` 存储了 Node 的 metadata，Eru 用来判断这个 Node 是否可用时只会参考 `Node Info`。

而 `Node Status` 是由 Agent 上报， 被 Selfmon 监听的。如果没有 Selfmon，`Node Status` 的变化不会造成任何影响。

Selfmon 会根据 Node Status 的变化来设置对应的 `Node Info`。例如某个 `Node Status` 因为过期被 ETCD 删除了，Selfmon 会把这个 Node 的 `Node Info` 设置为 `Available: false`。

Selfmon 主要的运行流程是：

1. 不断请求拿到分布式锁
2. 拿到锁之后进行初始化工作：调用 Core 的接口拿到所有 Node Status，然后设置对应的 Node Info。
3. 持续监听 `Node Status` 的变化，并做相应处理。