# Component

### 1. Core

> 调度器核心

  - 无状态
  - 生成编排方案
  - 并发进行部署

### 2. Agent

> Node 上的控制器

  - 资源消耗低
  - 负责容器检查
  - 获取 Metrics 发送到远端
  - 转发日志

### 3. ELB (Eru load balancer)

> 7 层动态服务发布

  - 基于 [Openresty](https://openresty.org/en/)
  - 本身也是 Eru 应用之一
  - 通过指定的 Redis 进行发布工作
  - 应用上下线过程中保证流量平稳切换

### 5. Cli

> 命令行工具

  - 提供类似于 AWS Lambda 子命令
  - 通过 cli 操控集群本身
  - 通过 cli 可以在初始化集群之后进行集群自举

### 6. Minions

> A calico libnetwork plugin port

  - Calico libnetwork plugin 不支持 docker engine
  - 采用最新的 libcalico + etcdv3 实现
  - 行为和 calico-cni plugin 一致
  - 支持 bird 的最新版本和其特性
  - 支持原生 fixed IP 特性

### 7. Barrel

> A docker daemon wrapper

  - Docker wrapper for fixed IP feature
  - 原生区分了 stop/remove 行为 