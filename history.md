## History

### Core

* 2.0

2016-2017 ENJOY 容器调度平台，使用 Golang 重写，放弃了复杂的 Websocket 反连结构。通过 gRPC 重新实现了 Core 的接口，架构上 Core 无状态化，和 Agent 平行。

* 1.0

2015-2016 芒果TV 容器调度平台，用 Python 实现，采用集中式架构。每个 Core 会去通过 Websocket 连上每台机器上的 Agent，计算出资源之后将资源邀约发到 Agent 上通过 Agent 来进行 Docker 操作。

* 0.5

2014-2015 芒果TV 容器化 PaaS，类似于 DAE 的 SDK-Runtime 结构，通过容器包装 Runtime 来支持不同的语言。

### Agent

* 2.0

和 Core 解耦，本质上是跑在 Node 上的一个监听代理，不负责具体的容器操作，负责日志和 metrics 等。

* 1.0

本质上是在 Node 上的一个 Websocket server，这样设计的好处是 Core 可以动态感知 Agent 的可用性，但带来了管理规模上的复杂度。这个版本里面尝试了各种日志输出手段，最后选择了 [logspout](https://github.com/gliderlabs/logspout) 作为参考对象，当然做了很大的[修改](https://github.com/gliderlabs/logspout/pull/15), 并做了一点微小的[工作](https://github.com/gliderlabs/logspout/pull/8)。

### [ELB](https://github.com/projecteru2/elb/blob/master/CHANGELOG.md)
