# Eru
[![MIT license](https://img.shields.io/github/license/mashape/apistatus.svg)](https://opensource.org/licenses/MIT)

# Introduction

Eru is a docker container orchestration platform.

作为 Eru 容器调度编排系统白皮书，会对 Eru 的架构/演进历史/安装部署/使用/参与开发等方方面面给出充分的说明。

目前最新版本为 Eru version 2.0.0 Aka Eru2。

## 设计目标

设计目标包括但不限于：

- 降低系统管理复杂度
- 简化服务的部署管理
- 优化基础服务的调配
- 提高资源的使用效率
- 统一开发测试生产三环境
- 持续交付工作流的良好支持
- 统一在线和离线资源调度

原则上来说 Eru 只是将 Docker 作为容器最小单元引擎，并不做过强的耦合和依赖。通过架构层面上的设计和优化，使得 Eru 可以支持上千甚至上万台物理机器集群，满足小型到大型公司平台层面的调度编排需求。

## License
Eru White Paper is released under the [MIT license](https://github.com/projecteru2/white-paper/blob/master/LICENSE).
