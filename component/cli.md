# Cli

#### 设计思想

我们考虑在没有 PaaS 的时候应该提供一个工具对 Eru 进行简单的操作，并且能通过这个工具完成 Eru 集群本身的自举行为，因此我们就开发了这个 Cli 工具。另外 Eru core 本身并不包括 Workflow 强制性的定义以及应用绑定，通过 Cli 工具我们可以很容易的完成对已有 image/source 0 侵入的部署，在没有任何其他开销的情况下体验 Eru 对资源操作和编排的能力。

另外我们考虑在非 Application 的情况下，如何使用 Eru 平台。比如本身是 [Serverless](https://en.wikipedia.org/wiki/Serverless_computing) 的架构，又或者是简单的一个脚本定时跑一下什么的，亦或是支持已有的上层设施。在这些情况下，传统的 Eru App 模式会变得相对而言比较重，因此我们提供了类似于 [AWS Lambda](https://aws.amazon.com/cn/lambda/) 的 Lambda 支持。但是要注意的是，在 Eru 的体系里面 Lambda 在系统层面表现出的是一个启动器的形式，它负责向 Core 申请资源并且运行某个镜像，至于这个镜像里面跑什么完全取决于启动器的传入参数。因此用户可以很方便的调用线上资源运行一些 Short time 脚本或者是 Serverless 的应用，并与上层如 [Dkron](http://dkron.io/) 结合起来实现大一统的平台支持。

在使用 Cli 的情况下，因此可以做到随调随用，保证了用户层面的灵活性，并且能很方便的用于对已有的基础设施进行集成，使得整个 Eru 成为最底层的大一统平台。同时通过 Eru 的强制超时机制保证了单一用户不能长时间占用资源配额，保证了平台层面的资源利用率。

#### 主要功能

1. 控制 Eru 集群，部署/删除/扩容/缩容等。
2. 运行一段脚本，通过配置可以方便的使用 ENV 传入所需信息。
3. 简单的 watcher 监控容器状态。
4. Lambda 有受限的 Stdin 支持。考虑到本身 Core 是基于 GRPC 实现的接口层，而 GRPC 是采用 Message 来进行传输的最小单元。因此在代码层面上 Lambda 所支持的 Stdin 是没法通过类似于 `os.Copy` 的行为直接 pipeline 到远端容器之中。所以我们限定了在此模式下 Lambda 会通过 `\n` 来处理每一条消息并包装发到远端的 Core 里，由 Core 来拆包传入 Container 中。
5. 简单的模拟 Runtime Shell。基于 2 提供的 Stdin，我们在 Lambda 启动器中提供了一个非 tty 的受限 Shell，最主要的目的是为了提供给线上服务一个 `maintenance` 环境，如 Django 应用的数据库操作等。

#### 使用方式

CentOS 7 上我们提供了 RPM 打包方式，可以通过 RPM 部署。
在其他系统上也可以通过 cli 的[镜像](https://hub.docker.com/r/projecteru2/cli/)来执行。另外多说一句，因为 Eru 是自举的，因此可以通过 cli 来调用 eru 使用受控 cli 完成 cli 操作，虽然很别扭，大概就是这个意思吧。
