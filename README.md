# Eru
[![MIT license](https://img.shields.io/github/license/mashape/apistatus.svg)](https://opensource.org/licenses/MIT)

# Introduction

Eru is a short word for Eru iluvatar, the God in the lord of the ring's world. 

It is a hybrid orchestration system. Can orchestra multiple resource types like container, virtual machine, systemd etc.

作为 Eru 调度编排系统白皮书，会对 Eru 的架构/演进历史/安装部署/使用/参与开发等方方面面给出充分的说明。

## 设计目标

Eru 设计目标包括但不限于：

- 降低系统管理复杂度
- 简化服务的部署管理
- 优化基础服务的调配
- 提高资源的使用效率
- 统一开发/测试/生产等多种环境
- 持续交付工作流的良好支持
- 统一在线和离线资源调度

原则上来说 Eru 支持任意满足其接口定义的容器/虚拟化/系统引擎，并不做过强的耦合和依赖。通过架构层面上的设计和优化，使得 Eru 可以支持上千甚至上万台物理机器集群，满足小型到大型公司平台层面的调度编排需求。

## License
Eru is released under the [MIT license](https://github.com/projecteru2/white-paper/blob/master/LICENSE).
