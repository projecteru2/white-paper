# Features

### 基于配置文件定义应用

- 在现有的应用上只需要增加一个配置文件 app.yaml 即可定义应用在 Eru 集群里的编译和运行
- 对现有项目 0 侵入

### SDN 网络安全隔离

- 使用开源的 [calico](https://github.com/projectcalico/calico) 项目构建 [SDN 网络](https://zh.wikipedia.org/wiki/%E8%BB%9F%E9%AB%94%E5%AE%9A%E7%BE%A9%E7%B6%B2%E8%B7%AF)
- 高效率的应用内网络互通
- 多租户隔离(依托于 calico 自己的 ACL 由 Ops 层面决定)

### 基于容器技术支持多样化的技术栈

- 使用开源的 [docker](https://github.com/moby/moby) 项目构建容器云
- 自动生成 Dockerfile, 支持多步构建部分替代 CI 能力
- 应用镜像与 SCM 提交历史绑定，方便回滚操作
- 容器技术天然的支持隔离系统和应用的依赖
- 集群可配置全量缓存最近多个版本的镜像，加速部署回滚等行为
- 提供动态运行代码/命令入口 (类似于 [AWS lambda](https://aws.amazon.com/cn/lambda/))
- 提供无痛升级能力

### 应用在线扩容缩容

- 自行设计调度核心进行高效调度
- 支持用户从数量和资源两个维度进行扩容/缩容
- 支持应用在不下线的前提下实时重分配资源

### 节点在线扩容缩容

- 通过 [ansible](https://github.com/ansible/ansible) 开发集群管理运维工具包
- Pod/Node 操作均可以在线执行，不影响当前状态

### 统一认证

- Workflow 依托于自行开发统一认证组件 （sso）
- 支持 oauth2 的多种认证方式

### 服务健康检查

- 提供自定义服务健康检查，实时知道每一个应用容器的状态
- 绑定服务发现实现 0 中断服务保证

### 集群体系化的日志收集

- 支持多种日志 forward 行为，方便接入各类已有基础设施
- 默认收集应用的 stdout/stderr 日志收集
- 附带足够多元信息日志流方便规整和查询

### 集群自动服务发布

- 基于 [Openresty](https://openresty.org/en/) 实现了 7 层 Elastic load balancer
- 配合健康检查和 Eru 应用控制能力实现动态发布
- 在应用容器发生异常时通过 Eru 反馈机制实时进行节点摘除行为，避免服务崩溃

### 可选的集群体系化的备份和恢复

- 支持在 app.yaml 中显式声明 volume 备份需求和策略，以及设定备份策略
- 支持指定备份恢复
