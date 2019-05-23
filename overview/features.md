# Features

### 基于配置文件定义应用

- 在现有的应用上只需要增加一个配置文件 spec.yaml 即可定义应用在 Eru 集群里的编译和运行
- 配置文件不与代码/可运行镜像绑定，可以放在任何位置
- 对现有项目 0 侵入

### 调度器中立

- Eru core 层面不限制 workflow 行为，只关心资源和编排调度
- 通过 [cli](https://github.com/projecteru2/cli) 可以使用本地或者远程配置直接通过 Eru 来部署已有的镜像
- 方便接入上层应用 PaaS 亦或是现有的其他基础设施

### SDN 网络安全隔离

- 默认使用开源的 [calico](https://github.com/projectcalico/calico) 项目构建 [SDN 网络](https://zh.wikipedia.org/wiki/%E8%BB%9F%E9%AB%94%E5%AE%9A%E7%BE%A9%E7%B6%B2%E8%B7%AF)
- 高效率的应用内网络互通
- 多租户隔离(依托于 calico 自己的 ACL 由 Ops 层面决定)
- 支持其他不同的 SDN Driver

### 基于容器技术支持多样化的技术栈

- 使用开源的 [docker](https://github.com/moby/moby) 项目构建容器云
- 自动生成 Dockerfile, 支持多步构建部分替代 CI 能力
- 提供动态运行代码/命令入口 (类似于 [AWS lambda](https://aws.amazon.com/cn/lambda/))
- 容器技术天然的支持隔离系统和应用的依赖
- 提供无痛升级能力

### 支持多种 executor

- Eru 可以混合编排容器和虚拟机
- Eru 允许自定义 Executor

### 应用在线扩容缩容

- 自行设计调度核心进行高效调度
- 支持用户从数量和资源两个维度进行扩容/缩容
- 支持应用在不下线的前提下实时重分配资源
- 支持 in-place 复用资源配额就地更新

### 服务健康检查

- 提供自定义服务健康检查，实时知道每一个应用状态 （Agent）
- 同时支持 tcp 和简单 http 健康检查行为

### 集群体系化的日志收集

- 支持多种日志 forward 行为，方便接入各类已有基础设施
- 默认收集应用的 stdout/stderr 日志收集
- 附带足够多元信息日志流方便规整和查询

### 集群自动服务发布

- 基于 [Openresty](https://openresty.org/en/) 实现了 7 层 Elastic load balancer
- 配合健康检查和 Eru 应用控制能力实现动态发布
- 在应用发生异常时通过 Eru 反馈机制实时进行节点摘除行为，避免服务崩溃

### 可选的存储配置和备份

- 支持在 spec.yaml 中显式声明 volume
- 支持备份

### 支持配置分离

- 允许创建时动态传入文件
- 允许自定义环境变量
- 允许运行时传入不同配置
