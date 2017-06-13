# EruCtl

#### 设计思想

EruCtl 类似于 [CalicoCtl](https://github.com/projectcalico/calicoctl) 用于简化 Eru 的部署。

#### 主要功能

1. 在有 Docker 的机器上部署 Agent。和 CalicoCtl 一样，EruCtl 也会部署一个名为 eru-node 的容器，其中就是 agent 的进程。在部署过程中，EruCtl 会对这个容器进行以下几个操作。

    * 赋予 root 权限以 host 模型跑在宿主机上。
    * Mount /sys/fs/cgroup 到容器中，Agent 需要通过这个来获得容器 stats。
    * Mount /proc 到容器中，Agent 需要通过这个来获得容器网络的 stats。
    * Mount unix:///var/run/docker.sock 到容器中，Agent 需要通过这个来监听容器动态以及获取日志流。

2. Pod 管理。
3. Network 管理。
4. 在特定机器上部署或更新 Core。Core 的自举和 RPM 部署不同的在于，Core 会是一个 Raw type 的 Eru App。

#### 部署方式

CentOS 7 上我们提供了 RPM 进行部署。