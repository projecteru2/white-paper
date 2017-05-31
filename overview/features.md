# Features

### 基于配置文件定义应用

- 在现有的应用上只需要增加一个配置文件 app.yaml 即可定义应用在 Eru 集群里的编译和运行
- 对现有项目 0 侵入

### SDN 网络安全隔离

- 使用开源的 [calico](https://github.com/projectcalico/calico) 项目构建 [SDN 网络](https://zh.wikipedia.org/wiki/%E8%BB%9F%E9%AB%94%E5%AE%9A%E7%BE%A9%E7%B6%B2%E8%B7%AF)
- 高效率的应用内网络互通
- Pod 间网络默认隔离

### 基于容器技术支持多样化的技术栈

- 使用开源的 [docker](https://github.com/moby/moby) 项目构建容器云
- 自动生成 Dockerfile, 提供基于多种操作系统的 Base Image 继承树
- 应用镜像与 Git commit 绑定，方便回滚操作
- 容器技术天然的支持隔离系统和应用的依赖
- 集群全量缓存最近2个版本的镜像，加速部署行为
- 提供无痛升级能力

### 应用在线扩容缩容

- 自行设计调度核心进行高效调度
- 支持用户从数量和资源两个维度进行扩容/缩容

### 节点在线扩容缩容

- 通过 [ansible](https://github.com/ansible/ansible) 开发集群管理运维工具包
- Pod/Node 操作均可以在线执行，不影响当前状态

### 统一认证

- 集群自行开发统一认证组件 （sso）
- 支持 oauth2 的多种认证方式

### 服务健康检查

- 提供自定义服务健康检查，实时知道每一个应用容器的状态
- 绑定服务发现实现 0 中断服务保证

### 集群体系化的日志收集

- 使用 rsyslog 封装集群整体日志收集
- 默认收集应用的 stdout/stderr 日志收集
- 定制检测 web 服务 load balancer 的 nginx 日志收集和数据统计

### 集群自动服务发布

- 基于 [Openresty](https://openresty.org/en/) 实现了 7 层 Elastic load balancer
- 配合健康检查和 Eru 应用控制能力实现动态发布
- 在应用容器发生异常时通过 Eru 反馈机制实时进行节点摘除行为，避免服务崩溃

### 可选的集群体系化的备份和恢复 ( moosefs )

- 采用开源的 [moosefs](https://github.com/moosefs/moosefs) 作为分布式存储后端
- 支援在 app.yaml 中显式声明 volume 备份需求和策略，以及设定备份策略
- 支援指定备份恢复

### 可选的集群日志查询组件 （ kafka + elasticsearch + kibana ）

- 采用开源的 [kakfa](https://github.com/apache/kafka)  , [elasticsearch](https://github.com/elastic/elasticsearch) , [kibana](https://github.com/elastic/kibana) 搭建外部依赖的 kafka 集群和 elasticsearch 集群，封装集群可选组件 kibana
- 日志收集组件支援通过 rsyslog 发送所有日志到上述外部依赖 kafka
- 在 kibana 上支援对集群应用日志和入口 ELB 日志的条件组合查询
