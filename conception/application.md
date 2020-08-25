# Application

### 概念

Application 在 Eru 中用于描述一个可部署的项目。对于 Core 而言它并不关心跑的是哪一个 App，它只负责调度和编排，因此这是一个 Workflow 上的概念。对于 Eru 而言，上面的每一个容器/虚拟机都可以对应到一个具体的 Application 描述。当然由于一些历史原因，我们在定义 App 的时候允许一个 App 有多个入口（多种能力），主要是为了方便项目在非拆分的情况下也能最小化迁移代价。当然对于原生 Docker 应用而言，我们还是建议一个容器一个进程一份目的和明确的 App 用途定义的。

### 属性

对于 App 而言，最重要的属性主要是：

1. Name, 每一个 Eru App 的 Name 都应该是唯一的。
2. Entrypoint，用于描述代码的入口。App 支持多种入口，代表资源的不同行为，在容器引擎的实现下每一种入口的日志都会自动的分流到对应远端。
3. 可选 Stages/Builds，用于描述如何讲项目打包成镜像，也可以用于简单的 CI process。

在 Eru 里面一切都是一个描述文件，具体可以参考 [agent 自举的描述文件](https://github.com/projecteru2/agent/blob/master/app.yaml)。