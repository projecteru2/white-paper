# Scheduling

请先参考 [资源](todo)

eru-core 把 Scheduling 分为“容量计算”和“编排”两个步骤.

### 容量计算

调度的目的是按照资源请求计算每个节点能部署多少个实例, 以及每个部署方案的资源绑定情况.

比如:

1. 请求 memory 10M, 节点可用内存 100M, 那么该节点能部署(Capacity) 就是 10.
2. 请求 cpu 1 核, 绑定, 节点可用 cpu 状态是 `{0:0, 1:0, 2:1, 3:1}`(0/1 核已用光, 2/3 核还未被调度), 那么 Capacity 就是 2, 方案分别是 `{2:1, 3:1}`.
3. 请求 volume `AUTO:/data:rw:100`, 节点可用磁盘状态是 `{/sda0:1000, /sda1: 200}`, 那么 Capacity 是 12, 方案分别是 10 * `{/sda0:100}` 和 2 * `{/sda1:100}`.

实际的调度策略会比这更加复杂, 包括碎片核的调度, 独占式调度, 等.

### 编排

经过上一步容量计算后, 我们有了每个节点可部署的实例数目, 比如 `{node1:0, node2:5, node3:100}`, 接下来这一步会根据此 Capacity Map 和用户请求来选出在哪些节点上分别部署多少个实例.

eru-core 内置了 4 种编排策略, 分别是 Communism(Auto), Fill, Global, Average.

#### Communism(Auto)

Communism 策略的目的是保证在调度完成后该应用的所有实例(新+旧)在所有节点上数量尽量平均.

比如:

1. 已有实例状态是 `{node1:0, node2:0, node3:0}`, 请求 Count 3, 最终编排结果是三节点各部署一实例, 最终状态是(新+旧) `{node1:1, node2:1, node3:1}`
2. 已有实例状态是 `{node1:5, node2:4, node3:0}`, 请求 Count 3, 最终编排结果是全部部署到 `node3`, 最终状态是(新+旧) `{node1:5, node2:4, node3:3}`

`NodesLimit` 对 Auto 对语义是限制节点上实例总数(新+旧)的上限.

#### Fill

Fill 策略到目的是保证在调度完成后所有实例(新+旧)的总数量为 `Count` * `NodesLimit` 个; 若已有至少这么多实例报错.

比如:

1. 已有实例状态是 `{node1:0, node2:0, node3:0}`, 请求 Count 1, NodesLimit 3, 最终编排结果是三节点各部署一实例, 最终状态是(新+旧) `{node1:1, node2:1, node3:1}`
2. 已有实例状态是 `{node1:1, node2:0, node3:0}`, 请求 Count 1, NodesLimit 3, 最终编排结果是 `node2`/`node3` 各部署一实例, 最终状态是(新+旧) `{node1:1, node2:1, node3:1}`
3. 已有实例状态是 `{node1:1, node2:1, node3:1}`, 请求 Count 1, NodesLimit 3, 最终编排结果是报错, 因为已有状态已满足.

#### Average

Average 策略的目的是在 `NodesLimit` 个节点上部署 `Count` 个实例, 保证本次增量部署 `Count` * `NodesLimit` 个实例.

比如:

1. 已有实例状态是 `{node1:1, node2:0, node3:0}`, 请求 Count 1, NodesLimit 3, 最终编排结果是三节点各部署一实例, 最终状态是(新+旧) `{node1:2, node2:1, node3:1}`

#### Global

Global 策略的目的是保证节点的资源使用率尽量平均.

比如:

1. 当前节点的资源利用率是 `{node1:1%, node2:2%, node3:3%}`, 请求 Count 3, 一个实例在三个节点分别会占用的资源率是 `{node1:0.4%, node2:0.6%, node3:1%}`, 最终编排结果是在 `node1`/`node2` 分别部署 2/1 个, 最终资源利用率是 `{node1:1.8%, node2:2.6%, node3:3%}`

`NodesLimit` 对 Global 暂无语义.
