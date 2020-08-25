# Health Check

### Old Health Check

Eru 是松耦合结构，Core 和 Agent 并非强制绑定的。因此在老的流程里面，Core 只负责创建容器，并且在 ETCD 中存储好元数据的 Key，至于 Value 则由各种 Agent 实现自行调用 Core 的 API 写入。从外部可以通过 ```DeployStatus``` 接口拿到数据变化和具体内容。

具体流程为：

1. Core 创建 ```/deploy/{APPNAME}/{ENTRYPOINT}/{NODE}/{CONTAINER_ID}``` 这样的 Key，值为空。
2. Agent 检测到容器启动
   1. 若没有 Health Check 设置，则直接写入一个 Json，包含 ```Health: true, Running: true```。
   2. 若有 Health Check，则先写入 ```Health: false, Running: false```，进行 health check 后再更改其值。
   3. 等 Agent 完成一轮 Health Check 后更新数据写入，并缓存下来
   4. 如果新的一 Health Check 得到的结果和缓存的不一致，调用 ```DeployContainer``` 接口进行更新

这个模式有几点好处：

1. 简单，单向反馈链路。只有 Key 存在的时候，Agent 才能写会成功，避免了数据加锁。
2. 数据反馈次数少，由 Agent 缓存住了。
3. 允许自定义 Agent 写入数据结构。

也有缺点：

1. Client 需要依赖 Core 的同时依赖各种 Agent 实现。
2. 假如 Node 断网，从外部 mark 容器为非健康非运行状态后，Agent 无法在 Node 恢复后自动更新运行和健康（因为被缓存了）。
3. 如果没有 Agent，Core 无法反馈任何容器状态。

因此对于这个 Health Check 流程做了一定的修改。

### New Health Check

对于 Core，增加了一个 ```Meta``` 数据结构，包含最小公约属性如 ```Networks``` 和 ```Running``` 等。各种 Agent 需要继承这个结构做自行扩展。这就解决了第一个缺点，使得 Client 即便在不知道各类 Agent 实现的前提下拿到部分运行时数据。

对于 Agent，通过 [go-cache](https://github.com/patrickmn/go-cache) 实现了 auto-clean 的状态缓存，这就解决了第二个缺点。如果外部 mark 容器健康状态后，一段时间后 Agent 也能正确的上报当前状态。从另外一个角度来说，提供了 maintenance 的能力。

另外重新设计了数据上报接口的逻辑，使得在满足有上报数据路径和上报状态变化 2 个条件的同时，Core 会修改容器的元数据，存储其 ```Running``` 和 ```Networks``` 2 个属性。这使得 Core 有能力在没有 Agent 或者 Agent 临时下线后依然能反馈一部分运行时装填，解决了第三个缺点。

### 资源生命周期

在实现新的 Health Check 后，资源的生命周期大体如下：

1. 创建容器，Core 会在元数据中存储创建后得到了 ```Running``` 和 ```Networks```。
2. Agent/Yavirtd 接手资源健康检查：
   1. 如果没有健康检查设置，Healthy 永远为 true。
   2. 如果有健康检查，Healthy 为 false，上报。
   3. 开始健康检查，上报检查结果（Healthy 属性）。
   4. 如此反复。
3. 如果 Core 接到的上报状态与之前的状态不一致则会更新元数据。

