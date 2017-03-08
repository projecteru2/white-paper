# Architecture

### 核心系统物理架构

>目标是做成可以横向扩展简单可依赖的调度核心

![](img/core.png)

物理层面来说，用户操作的 UI 和应用抽象都实现在 [citadel](https://github.com/projecteru2/citadel) 这个项目之中。它使用 gRPC 与 Core 交互，从而把业务的一个个 App 部署到 Eru 的集群之中。

往下则是 Eru 核心组件 [core](https://github.com/projecteru2/core)。core 是无状态的编排调度核心，可以依据集群大小进行横向扩展。其职责是将 citadel 的容器操作进行编排并且按照一定的资源维度调度到任意一个 Pod 中去。core 之间的共享数据存储在 etcd 之中。

这里的 Pod 是一个逻辑概念，用于描述一组机器（Nodes）。每一个 Pod 都是由一组网络互通的 Node 构成。如果在大二层的层面上可以做到多机房互通的话，Pod 是允许旗下的 Node 也是跨机房的，从而在逻辑层面抹平机房这个概念。

在每一个 Node 上，均包含了至少2个必要的组件:

- [Docker](https://www.docker.com/)
- [agent](https://github.com/projecteru2/agent)

其中 agent 是 eru 二元结构中负责在 Node 上监控容器，将日志流打上必要的元信息进行转发，以及 metrics 的收集。

在我们的实际用况中，我们使用了 [rsyslog](http://www.rsyslog.com/) 作为 Node 上日志收集器，并转发到远端。同时通过 [moosefs](https://moosefs.com/index.html) 来实现容器间的文件共享。

### 业务逻辑层面

### 各子系统

- SDN
