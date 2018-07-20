# 安装 Core

对于 Eru 而言，Core 有 3种方式：

1. 对于 CentOS 7 以上的机器 Core 提供了 RPM 方式安装，可以通过源码中的 ```make-rpm``` 来生成 RPM 包来安装。
2. 对于已有 Eru 而言，也可以通过 Eru 来部署 Core，具体可以参考 Core 的 [README](https://github.com/projecteru2/core#build-and-deploy-by-eru-itself)。
3. 当然也可以通过容器直接跑 Core，执行以下命令即可：

```
docker run -d \
  --name eru_core_$HOSTNAME \
  --net host \
  --restart always \
  -v <HOST_CONFIG_DIR_PATH>:/etc/eru \
  projecteru2/core \
  /usr/bin/eru-core
```

对于生产环境而言，为了避免 Docker 自身奇怪的问题和保证 Eru 整个集群旁路控制，同时 Core 是无状态的因此最好的情况是一组 VM 单独跑 Core。对于这些 VM 而言只需要暴露 ETCD 给 Core 访问就行了，如果有本地 ETCD Proxy 用户体验会更加简单。若干个 Core 其实就已经能负责上万台机器的调度和编排了。