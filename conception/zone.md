# Zone

## 概念

因为 Eru Core 的无状态性，用户能很方便的在不同机房搭建不同的 Eru 集群。对于上层 Workflow 组件 [Citadel](https://github.com/projecteru2/citadel) 我们引入了 Zone 的概念。一个 Zone 是指一个 Eru 集群，类似于公有云的分区概念，用户可以选择在不同的 Zone 分别部署其应用。

## 属性

Zone 只存在于 Workflow 层面，对于 Core 而言他们是不会知道有另外一组 Core serve 其他集群的。
