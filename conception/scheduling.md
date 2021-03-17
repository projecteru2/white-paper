# Scheduling

请先参考 [资源](https://book.eru.sh/conception/resource)

eru-core 把 Scheduling 分为“容量计算”和“编排”两个步骤.

### 容量计算

调度的目的是按照资源请求计算每个节点能部署多少个实例, 以及每个部署方案的资源绑定情况.

比如:

1. 请求 memory 10M, 节点可用内存 100M, 那么该节点能部署(Capacity) 就是 10.
2. 请求 cpu 1 核, 绑定, 节点可用 cpu 状态是 `{0:0, 1:0, 2:100, 3:100}`(0/1 核已用光, 2/3 核还未被调度), 那么 Capacity 就是 2, 方案分别是 `{2:100}`和 `{3:100}`
3. 请求 volume `AUTO:/data:rw:100`, 节点可用磁盘状态是 `{/sda0:1000, /sda1: 200}`, 那么 Capacity 是 12, 方案分别是 10 * `{/sda0:100}` 和 2 * `{/sda1:100}`.

这一步计算会有两个产出:
- 第一个产出是容量, CapacityMap(`map[Node]int`), 比如 `{node1: 10, node2: 0, node3:1}`, 记录每个节点可部署的实例数目, 可能是无限大(我们允许资源请求为 0, 即无限制)
- 第二个产出是资源绑定方式, PlanMap(`map[Node][]map[string]int64`), 比如 `{node1: [{1:100}, {2:100}]}`, 记录每个节点上可用的资源绑定细节, 可能为空(比如内存资源并不需要绑定).

实际的调度策略会比这更加复杂, 包括碎片核的调度, 独占式调度, 等.

#### 容量计算算法

##### CPU 为主

假设所需 M bytes 内存和 X.Y 个 CPU，其中 X 表示整数位，Y 表示小数位。我们将单一节点上的 CPU 抽象成由 A 个整数核和 B 个碎片核组成。假如管理者设定的运算量为 P ，那么每个碎片核上的每一份能提供的就是 1/P 运算能力。因此对于单个 Node 而言最大可部署数量为 min( Memory/M，min( A/X，B*P/Y )) 个。

在 A+B = CPU 总数量的前提下，我们要通过不断的尝试组合 A, B 的值来确定多个 min( A/X, B*P/Y ) 结果中最多的那个，再与内存计算结果进行比较。由于 cpushare 是单个核全局一致的，因此没法拆分 Y 为更小的多个运算量之和，所以我们只允许一次绑定一个碎片核。实际实现中还需要考虑已有碎片核的处理和碎片核运算量不能跨核进行，因此会更复杂一些。

##### Memory 为主

简单的进行 Memory / M 即可，对于 CPU 需求而言采用 CPU period 和 quota 进行全局控制。

以上 2 种算法在得到资源组合之后，Eru 会对 Nodes 上可部署数量排序，由少到多。然后再通过计算判断 Nodes 上的可部署数量是否满足用户需求。这样虽然会带来单一 Node 资源利用率不够高的可能性，但在大规模部署的情况下会在 Nodes 层面更加平均，分布上会更好。但是当能承载最少数的那个 Node 也能满足用户需求的时候，多次部署会产生堆积到一台 Node 的情况，因此 Eru 引入了更高级的分布算法。

### 编排

经过上一步容量计算后, 我们有了 CapacityMap 和 PlanMap, 接下来这一步会根据此 CapacityMap 和用户请求来选出在哪些节点上分别部署多少个实例. 注意这一步不使用 PlanMap.

这一步计算会产出部署方式, DeployMap(`map[Node]int`), 比如 `{node1: 10, node3:4}`, 代表每个节点分别要部署多少实例. 这就是最终的要部署的实例数目了.

eru-core 内置了 4 种编排策略, 分别是 Communism(Auto), Fill, Global, Average.

#### Communism(Auto)

Communism 策略的目的是保证在调度完成后该应用的所有实例(新+旧)在所有节点上数量尽量平均.

`NodesLimit` 对 Auto 对语义是限制节点上实例总数(新+旧)的上限.

比如:

1. 已有实例状态是 `{node1:0, node2:0, node3:0}`, 请求 Count 3, 最终编排结果是三节点各部署一实例, 最终状态是(新+旧) `{node1:1, node2:1, node3:1}`
2. 已有实例状态是 `{node1:5, node2:4, node3:0}`, 请求 Count 3, 最终编排结果是全部部署到 `node3`, 最终状态是(新+旧) `{node1:5, node2:4, node3:3}`

#### Fill

Fill 策略到目的是保证在调度完成后所有实例(新+旧)的总数量为至少 `Count` * `NodesLimit` 个(至少 `NodeLimit` 个节点上有至少 `Count` 个实例); 若已满足此状态则报错.

`NodeLimit` 语义是“部署多少个节点”.

比如:

1. 已有实例状态是 `{node1:0, node2:0, node3:0}`, 请求 Count 1, NodesLimit 3, 最终编排结果是三节点各部署一实例, 最终状态是(新+旧) `{node1:1, node2:1, node3:1}`
2. 已有实例状态是 `{node1:1, node2:0, node3:0}`, 请求 Count 1, NodesLimit 3, 最终编排结果是 `node2`/`node3` 各部署一实例, 最终状态是(新+旧) `{node1:1, node2:1, node3:1}`
3. 已有实例状态是 `{node1:1, node2:1, node3:1}`, 请求 Count 1, NodesLimit 3, 最终编排结果是报错, 因为已有状态已满足.
4. 已有实例状态是 `{node1:2, node2:2, node3:0}`, 请求 Count 1, NodesLimit 3, 最终编排结果是 `node3` 部署1实例, 最终状态是 `{node1:2, node2:2, node3:1}`.
4. 已有实例状态是 `{node1:1, node2:1, node3:1}`, 请求 Count 2, NodesLimit 2, 最终编排结果是 `node1`/`node2` 各部署1实例, 最终状态是 `{node1:2, node2:2, node3:1}`.

#### Average

Average 策略的目的是在 `NodesLimit` 个节点上部署 `Count` 个实例, 保证本次增量部署 `Count` * `NodesLimit` 个实例.

`NodeLimit` 的语义和 Fill 一样, 是“部署多少个节点”的意思.

比如:

1. 已有实例状态是 `{node1:1, node2:0, node3:0}`, 请求 Count 1, NodesLimit 3, 最终编排结果是三节点各部署一实例, 最终状态是(新+旧) `{node1:2, node2:1, node3:1}`

#### Global

Global 策略的目的是保证节点的资源使用率尽量平均.

比如:

1. 当前节点的资源利用率是 `{node1:1%, node2:2%, node3:3%}`, 请求 Count 3, 一个实例在三个节点分别会占用的资源率是 `{node1:0.4%, node2:0.6%, node3:1%}`, 最终编排结果是在 `node1`/`node2` 分别部署 2/1 个, 最终资源利用率是 `{node1:1.8%, node2:2.6%, node3:3%}`

`NodesLimit` 对 Global 暂无语义.
