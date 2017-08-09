# Node

#### 概念

集群中的节点，称之为node，node是集群中运行和调度容器的基本单位。

#### 属性

1. node名称
2. node上docker engine的地址
3. node所属的资源池，pod
4. node的状态（是否可用）
5. node的CPU/内存状态
6. node是否对外公开


#### 用途

node上运行着eru-core调度而来的容器。

除了docker daemon这个基本服务，每个node节点上还运行着一个agent，负责监听docker事件，并更新etcd中关于node和node上运行的containers的信息，以及转发containers产生的日志。

另外node上还运行着calico网络服务，负责容器之间，容器与宿主机之间的通信。
