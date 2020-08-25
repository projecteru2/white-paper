# Overview

Eru 是类似于 [Kubernetes](https://kubernetes.io) 的分布式容器编排和部署系统。在整个架构中使用了若干种开源项目构建而成，包括不仅限于如 [etcd](https://github.com/coreos/etcd), [calico](https://www.projectcalico.org/) 等。

Eru 不算是一种 **[PaaS](https://zh.wikipedia.org/wiki/%E5%B9%B3%E5%8F%B0%E5%8D%B3%E6%9C%8D%E5%8A%A1)** 实现，更类似于 [Nomad](https://www.nomadproject.io/) 的 multiple type executor orchestration system。因此它并不会有诸如资源异常退出拉起或者 re-deploy 这样的能力，它只专注于编排和部署。Eru 不但提供了资源维度的调度，同时也负责内容编排，本质上来说是一种抢占式资源的全局调度器。

另外在 Eru 的实现中，我们通过高效的资源分配算法避免了传统上部署加锁的问题，使得 Eru 能高效透明的处理部署和编排行为。对于寻求高效运维方案的组织和 devops 人力缺乏的 startup 以及个人开发者而言更加友善。

同时通过上层支撑组件统一开发工作流，降低运维复杂度，提供了开发，集成，部署，运维的一揽子解决方案。
