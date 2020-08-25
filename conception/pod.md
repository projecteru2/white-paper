# Pod

### 概念

对于 Eru 而言，Pod 的概念和 [Kubernetes 的 Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) 是不太一样的。对于 K8s 来说 Pod 是最小部署单元，并且也是在 Pod 上实现了最小资源单元。在 Eru 中 Pod 是一个虚拟概念，用途就是很直观的告诉上层这一组机器（Node）是用来做什么的，然后依托于其他手段通过不同的网络参数来满足具体服务之间的隔离或者互通需求。App 能部署在某一个 Pod 或者某几个 Pod 上，Pod 下面的 Node 也不会限制在一个机房之内。

### 属性

Pod 只有一类属性，就是 Nodes，用于描述有多少个节点，对于 Eru 来说最小资源单位是 Node 上的资源总和。
