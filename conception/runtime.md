# Runtime

### 概念

Runtime 是 eru-core 对接的 workload 实际引擎, 目前支持 Docker Container, Yavirt KVM, Systemd Daemon 三种引擎.

对用来来说, 不需要特别关注 Runtime 的实现细节, eru-core 会尽量抹平不同 Runtime 在接口上的区别, 让用户能通过指定不同的 [pod](https://book.eru.sh/conception/pod) 在不同的 Runtime 上部署 Workload.

### Docker Container

eru-core 与节点上的 Dockerd 通讯, 完成基于 dockerd-containerd-runc 的容器部署.

### Yavirt KVM

Yavirt 是 Shopee 自研的基于 QEMU-KVM 的虚拟机 Runtime, 并集成了 Calico SDN, 实现了 eru-core 对容器-虚拟机的混合编排系统.

Yavirt KVM Runtime 相比容器提供了更强的隔离性和安全性.

Yavirt 也实现了类似 Docker 的镜像打包和 hub 体系, 可以很方便地上传下载.

### Systemd Daemon

对于一些更基础的 DaemonSet 性质的服务, 或是容易与 dockerd 的启动呈相互依赖, 如 calico-node 和 CNM/CNI 插件, 使用 Systemd Daemon Runtime 更合理.

当前(2021-02) Systemd Daemon Runtime 尚在施工中, 尤其是二进制的部署方案尚未落地. 请参考[文档](https://github.com/projecteru2/core/issues/317).
