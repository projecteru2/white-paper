# Application

#### 概念

一个应用即称之为一个application，简称app。app是运行container的一个类，app下又分了不同的entripoint，每一个entripoint对应一个或者多个容器；不同的entripoints就组成了一个app应用。

#### 属性

1. 应用名，appname
2. 应用入口，Entrypoints：一个应用可以有多个入口，入口也可以理解为应用的行为或者动作。比如一个应用中有api，monitor，test三个entrypoint，那么我们可以指定这个app的运行方式为三者之一，三个entripoint即三个不同的动作（服务方式）。
3. 数据卷，volumes：即app希望挂载本地的目录卷（mfs共享目录）

其中，entripoint又细分如下：

1. 启动命令：command
2. 运行启动命令之后的命令，运行启动命令之前的命令
3. 对外开放的端口
4. 网络模式：host模式，自定义的网络模式
5. 重启规则
6. 健康检查端口或url地址
7. 日志配置以及特权配置等

#### 用途

配置并定义容器运行的资源及规则。