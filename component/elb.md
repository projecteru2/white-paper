# Eru Load Balancer
> a.k.a ELB

#### 设计思想

在应用容器化之后，我们需要有一个统一的地方进行暴露和发布这些容器，但是在这里会有几个问题。

* 容器的 IP 不固定。容器在重启之后的 IP 是可变的，当然我们可以把它做成不可变，诸如外部的分配器或者是 VIP 绑定。但无论是 DNAT 还是 ovs 做外部控制面都会引入其他的问题，如 DNAT 依赖于 Node 上的操作，ovs 会有额外的 overhead 等。另外 IP 固定在做漂移的时候处理也会比较复杂，对于应用而言依然会有一定的离线时间。
* 不可能让客户端做复杂均衡逻辑。理想状态下容器是一组一组提供服务的，那么自然而然诸如 health check 上下线等都需要有一个角色去负责维护这个请求路由。抛开上层 load balancer 来看，让 client 去做是一件不太现实的事情。何况容器本身生命周期就低于传统运维部署的生命周期，如何同步容器的请求路由也是一件让人头疼的事情。

那么既然容器 IP 不固定，采用上层 proxy 发布的模式，又会带来什么呢？

* 容器透明化。客户端只需要一个 domain 到 proxy 即可，至于后面有多少个容器，都在哪里，没所谓。
* proxy 负责容器状态。无论是 health check 还是维护请求路由均由 proxy 负责。这样 client 可以最简化，方便开发行为。

当然，采用 proxy 也会有一定的劣势。

* 容器间调用会有额外的 overhead。以前知道 IP 就可以直连，如果双方容器均在一台机器上甚至可以做到本地连接的速率，而采用 proxy 之后请求流向会额外经过 proxy 导致有一定的损耗。
* 请求会被损耗。这个很容易理解，proxy 本身就是一层 overhead。

权衡之后，我们认为大部分的 7 层应用对于劣势的接受程度还是挺高的，况且对于 HTTP 来说增加的这点 overhead 实在不算啥。因此我们在 7 层服务的发布和均衡上就选择了 proxy 模式。

对于 4 层应用而言，proxy 模式一来没有特别好的 proxy 能支持频繁的 0 中断 IP 变化行为。另一方面既然用了 4 层自然而然希望 overhead 足够的小，因此 proxy 模式就不适合了。在这种情况下我们期望于应用方在框架 client 实现里面解决均衡逻辑和 health check，而发布方面放在第三方诸如 [zookeeper](https://zookeeper.apache.org/) 或者 [etcd](https://github.com/coreos/etcd) 中去。

#### 技术选型

我们采用基于 [nginx](https://nginx.org/en) 的 [openresty](https://openresty.org/en/) 作为 proxy。原因是有 lua 的支持后，可以自定义 proxy 的分流逻辑，可以根据应用不同维度进行针对性的分流，满足部分 Gateway 的需求。另外是可以自定义数据统计，做粗颗粒的请求跟踪和统计行为。

另外，我们也使用到了 [ngx_http_dyups_module](https://github.com/yzprofile/ngx_http_dyups_module) 来支持动态修改某一 backend 的 upstreams，实现真正意义上的 0 downtime。因为即便是 nginx 的 reload，在频繁调用的情况下依然会带来一些问题，比如 high load 等。

#### 主要功能

1. 动态的发布一组容器，并与一个或者一组绑定。
2. 支持请求统计，并发送到远端 [statsd](https://github.com/etsy/statsd)。
3. 多级分流模型，可以针对性的对一组容器中不同类别的分别导流，如 version，如 entrypoint 等。