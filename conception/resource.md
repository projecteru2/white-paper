# Resource

Eru 目前实现了四种资源的调度, 每种资源又有 request 和 limit 的区分.

### 四种资源

首先, 四种资源在使用前都需要注册到 [node](https://book.eru.sh/conception/node):

```
eru-cli node add --nodename zc \
    --endpoint tcp://127.0.0.1:2376 \
    --cpu 8 \
    --memory 4G \
    --volumes /sda0:100G --volumes /sda1:200G \
    --storage 500G \
    zc_pod
```

#### CPU

CPU 分为绑核和不绑核两种请求, 在截止 v21.02.15 的 eru-core 里它们在容器和 Systemd Runtime 上的实现都是通过 Linux Cgroup V1, 具体如下:

1. cpu=1.2, cpu-bind=false: `cpu.cfs_quota_us=120000`, `cpu.cfs_period_us=100000`
2. cpu=1.2, cpu-bind=true: `cpu.cfs_quota_us=120000`, `cpu.cfs_period_us=100000`, `cpuset.cpus=1,2`

可以看到绑核请求最终会反映到 `cpuset.cpus` 上, 而 cpu quota 会反映到 `cpu.cfs_quota_us`.

绑核请求最终会绑定到那几个核上由 eru-core [scheduler](https://book.eru.sh/conception/scheduling) 计算得出.

CPU 调度行为可以参考[Overall/Resource](https://book.eru.sh/overview/resource).

#### Memory

内存最终也是反映到 Linux Cgroup V1:

1. memory=14M: `memory.`

#### Volume

Volume 分为普通调度挂载, 独占式调度挂载, 硬挂载.

1. 普通调度挂载请求: `AUTO:/data:rw:100`
a. `AUTO` 标识需要调度, 由 eru [scheduler](https://book.eru.sh/conception/scheduling) 调度后从节点上可用 volume 选出一个
b. `/data` 是 Workload 内挂载目标
c. `rw` 代表可读写, 另外还可指定 `ro` 只读
d. `100` 代表可写入字节限制 100 字节
2. 独占式调度挂载请求: `AUTO:/data:rwm:100`
a. `rwm` 里的 `m` 代表 monopoly, 是独占调度的标识. 有这个 flag 的请求会独占一个空的宿主机 volume, 并且视该 volume 被占满, 不会在未来的调度中再次被分配.
3. 硬挂载: `/var/log:/var/log`
a. 硬挂载其实就是传统的挂载, 不会被调度, 不占用宿主机 volume 资源

Volume 目前只有 Yavirt Runtime 实现了 quota, 而 Docker 和 Systemd 只实现了挂载. 换句话说 `AUTO:/data:rw:100` 里的最后一个参数 `100` 没有被 Docker 和 Systemd runtime 实现.

#### Storage

Storage 的语义是: workload 的根目录磁盘 quota.
Storage 目前主要是用在 Yavirt Runtime 上, 用来分配给虚拟机实例的根目录挂载大小.
本身也是可以用在 docker container 上, 但是只有特定的文件系统才支持.

### Request, Limit

所有四种类型的资源都区分 Request 和 Limit, 它们的区别是:

* Request 只用在 eru-core 计算资源和调度编排, 最终反映到 eru-core 管理的资源元数据扣减
* Limit 只用在 Runtime 限制实例资源 quota

通过设置 request 和 limit 不同值可以达到不同的效果, 比如:

1. cpu.request=1, cpu.limit=0, cpu-bind=false: 资源扣减 1 核, 但是实际对 workload 不限制 cpu (0=unlimit)
2. memory.request=1G, memory.limit=2G: 资源扣减 1G, 但是实际对 workload 限制 2G 内存使用
3. volumes.request=[AUTO:/data:rw:100M], volumes.limit=[AUTO:/data:rw:50M]: 资源扣减 100M, 实际限制 50M 磁盘使用

有一些特殊规则在里面, 比如:

1. cpu-bind=true 时, 强制设置 cpu.limit=cpu.request
2. volumes.request 和 volumes.limit 里除了 quota size 不一样, 其他必须一致

在实际使用的时候, 用户需要知道自己想要什么, 从而做出正确的调用.
