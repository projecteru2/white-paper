# Node

### 概念

Node 是最小资源单元，也是最后执行部署的地方。通过 Core 的调度 App 容器/虚拟机会分布到某个 Pod 下的若干 Nodes 之中，并「消耗」掉其对应的资源。Node 指代的就是一台机器，多个 Node 组成了一个逻辑上的「业务组」也就是 Pod，多个 Pod 组成了一个 Eru 集群，而多个 Eru 集群通过不同的 Zone 来区分。

### 属性

Node 用于描述一个机器，最重要的属性主要有：

1. Memory，以 bytes 记录的内存值，无论是 mempod 还是 cpupod 均会使用到它。
2. CpuMap，把 CPU 抽象成 Index: Piece 的二元组，Index 表示是第几个核，Piece 表示最多能承载多少份算力。
3. Deploy Status, Node 会记录其上面某一类 App 的部署情况，通过这种信息我们可以在再次部署此类 App 的时候通过算法平衡同一 Pod 下各 Nodes 之间的容器/虚拟机数量，从而平均其容量，增强可用性。

还有一切其他的属性，可以参考其[定义](https://github.com/projecteru2/core/blob/master/types/node.go#L58)。