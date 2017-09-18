Install
========

对于核心 Eru 集群而言，安装起来很简单。

## Core

在 CentOS 系统上，Core 的安装只需要直接安装 RPM 包即可。对于其他系统或者是重 Docker 用户，可以直接使用我们已经打好的镜像 [projecteru2/core](https://hub.docker.com/r/projecteru2/core/) 来部署。

#### 配置

Core 的配置可以参考 [core.yaml.sample](https://github.com/projecteru2/core/blob/master/core.yaml.sample), 都比较直白，唯一对行为操作有影响的是 `schedule` 的配置。默认情况下 Core 会把一个 CPU 的一个核分成 10份算力去计算，然后进行编排。

## Agent

同样的，在 CentOS 系统上依然只需要直接安装 RPM 包，对于其他系统而言我们也提供了官方的镜像 [projecteru2/agent](https://hub.docker.com/r/projecteru2/agent/)。

#### 配置

Agent 的配置可以参考 [agent.yaml.sample](https://github.com/projecteru2/agent/blob/master/agent.yaml.sample), 配置的时候需要注意的是，log 参数支持 tcp 和 udp 2种形式，udp 对于单条日志大小限制为了1024K，因此对于线上系统而言，建议使用 TCP 模式。

## Pod

在安装好 Core 和 Agent 之后，集群实际上还是不能启动的，需要先注册 Pod。在注册 Pod 之前我们需要选择使用哪种模型的 Pod，对于 Eru 而言分 CPU 优先（favor CPU）和 MEM 优先（favor MEM）。对于 CPU 优先的 Pod 中每个节点在部署容器的时候均会精确的分配容器所占用 CPU，内存控制交由容器内进程自行控制。对于自带内存控制又对 CPU 性能敏感的应用如 redis 而言，是一种比较好的模式。

内存优先的模型下，Eru 也会从全局的角度分配 CPU 运算量给容器，但要注意的是在机器空闲的时候这个 CPU 运算量是可以保证的，而机器繁忙的时候用户得注意 load 值然后及时的通过 API 来迁移容器。这个模式下内存是强限制，CPU 可以超售。

## Node

注册 Node 并没什么太多可以说的，就是在写 Etcd 数据的时候建议冗余一些 CPU 和内存给 OS 本身。比如 4Core 的机器写 3Core，128G 内存注册为 120G 等。
