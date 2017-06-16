# Component

Eru 的组成部分有很多，在上一部分我们介绍了核心组件的 features，而在这一部分我们会着重介绍这些组件的设计和实现。

在 Eru 整个体系中根据组件的不同分为2个层面的支持，最底层的自然是 [Core](https://github.com/projecteru2/core) - [Agent](https://github.com/projecteru2/agent) 二元结构。而在上面我们不但有 [ELB](https://github.com/projecteru2/elb) 还有 [Citadel](https://github.com/projecteru2/citadel) 作为 PaaS 层面的支持。

同时通过 [EruCtl](https://github.com/projecteru2/eructl) 工具做 Eru 本身的运维，达到自举的目的。