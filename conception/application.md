# Application

### 概念

Application 在 Eru 中用于描述一个可部署的项目，仅对于 Citadel 而言是有意义的。对于 Core 而言它并不关心跑的是哪一个 App，它只负责调度和编排，因此这是一个 Workflow 上的概念。对于 Eru 而言，上面的每一个容器都可以对应到一个具体的 Application 描述。当然由于一些历史原因，我们在定义 App 的时候允许一份代码多种用途，主要是为了方便项目在非拆分的情况下也能最小化迁移代价。当然对于原生 Docker 应用而言，我们还是建议一个容器一个进程一份目的和明确的 App 用途定义的。

### 属性

对于 App 而言，最重要的属性主要是：

1. Name, 每一个 Eru App 的 Name 都应该是唯一的。
2. Entrypoints，用于描述代码的入口。App 支持多种入口，每一种入口的日志都会自动的分流到对应远端。
3. Stages/Builds，用于描述如何讲项目打包成 Docker 镜像，也可以用于简单的 CI process。

当然还有其他一些属性，可以参考 [Citadel 的文档](https://github.com/projecteru2/citadel/blob/ce/docs/user-docs/specs.md)。

在 Eru 里面，只有 PaaS 层的 citadel 才有 app 的概念，在 core 中一切都是一个描述文件，具体可以参考 [agent 自举的描述文件](https://github.com/projecteru2/agent/blob/master/app.yaml)。