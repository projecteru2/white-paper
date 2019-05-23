Prepare
========

一个 Eru 集群分2个部分，对于底层 Core 来说需要先准备以下基础设施：

1. [Docker](https://github.com/moby/moby) 18.07+
2. [etcd](https://github.com/coreos/etcd) 3.2.5+ 我们需要用到 3.2.5+ 提供的 mutex 行为
3. [calicoctl](https://github.com/projectcalico/calicoctl) 3.3+

#### 配置

我们在安装好最小依赖后还需要做以下设置：

* Docker 开启 IP 绑定，并设置好证书，需要注意的是 Docker 需要配置 Etcd 作为 discover。
* Etcd 服务默认启动，并能通过 IP 访问。
* Calicoctl 创建好子网。

对于以上服务而言，每一个有 Docker 的机器上都应该安装好 Calicoctl，并使用 plugin 的方式启动 calico-node。至于 Etcd 是否是独立服务并不强求。如果是单机测试整个集群的话，使用 Etcd 跑在同一台机器也是可以的。

建议启动本机 [etcd proxy](https://coreos.com/etcd/docs/latest/v2/proxy.html), 这样本机 Docker 均可以把 `cluster-store` 设置为本机，方便部署。

准备工作可以参考以下脚本:

* [docker.sh](https://github.com/projecteru2/quickstart/blob/master/docker.sh)
* [calico.sh](https://github.com/projecteru2/quickstart/blob/master/calico.sh)
* [etcd.sh](https://github.com/projecteru2/quickstart/blob/master/etcd.sh)

