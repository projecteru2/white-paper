# Agent

#### 设计思想

Agent 的设计第一目标是尽可能的减少对 Node 的影响。因为每个 Agent 都跑在 Node 上面，我们假定前提是资源是线性的。如果 Agent 消耗过多资源，那么就会导致 Node 上容器消耗资源的不确定性，进而影响整个 Node 资源的估算。

#### 主要功能

1. 容器检查，基于 Docker 自身的 event listener API。在 Agent 激活的时候会根据 etcd 中记录的数据进行一次全量匹配，把 Offline 的标记下线。运行期中也会根据 API 返回数据更新 etcd 中容器状态属性。

2. 容器 Metrics 收集，基于 CGroups。在这个实现里面我们并未采用 Docker 自身的 API，原因有以下这些。

    * Docker API 曾经不靠谱。如 1.5 版本的 stats API 没有终止参数，也没有步长限制，一旦启动就按照一定的频率循环获取，对 Node 的负载而言是不可控的。而 1.9 版本的 stats API 一开始 CPU 值会丢失，诸如此类。最后在容器数量过多的 Node 上，频繁的调用 stats API 本身对 docker daemon 而言负担也是很重的。
    * 早期 Docker 并未支持第三方网络，而我们采用的 MacVLAN 需要通过其他方法才能获得其 stats ( /proc/pid/net/dev )。即便是 Plugin 版本上线后，基于之前的原因，我们继续了这种方法。

    因此在这个逻辑中，我们完全通过文件的操作来获得容器的 metrics，对于 Node 而言，也就是多了几个 fd 而已，相比于更重的 http requests 轻量得多了。

3. 日志转发，基于 attach API。我们认为传统 docker log API 先天会有几个限制。

    * 文件型 API 导致 Node 磁盘空间和 IO 的负载不可控，如 json-file 和 journal。
    * 远端型 API 信息不够丰富，比如 syslog 等，缺乏特定的元数据支持。在远端看到的就是 [containerID][data] 二元结构，然而通过 cid 反查到某个容器对应的 App 等是一件繁琐的事情。假如在远端进行聚合，如果采用这种方式需要将每一条 log 进行匹配，这样对远端的压力也大。

    因此，我们采用更底层的 attach API，截获容器的 stdout/stderr，再在输出到远端之前把 Eru 所需的元数据聚合到其中，生成一条复杂格式的 log stream 然后才发送到远端。这样远端就只需要聚合处理并不需要做任何复杂逻辑操作了。
    
#### 部署方式

1. CentOS 7 上我们提供了 RPM 打包方式，可以通过 RPM 部署。
2. 通过 [EruCtl](https://github.com/projecteru2/eructl) 进行容器化部署。