# 安装 Agent

对于 Eru 而言，Agent 有 4 种方式：

1. 对于 CentOS 7 以上的机器 Agent 提供了 RPM 方式安装，可以通过源码中的 ```make-rpm``` 来生成 RPM 包来安装。
2. 对于已有 Eru 而言，也可以通过 Eru 来部署 Agent，具体可以参考 Agent 的 [README](https://github.com/projecteru2/agent#build-and-deploy-by-eru-itself)。
3. 使用 binary 裸运行 agent，binary 可以从[这里](https://github.com/projecteru2/agent/releases)下载
4. 当然也可以通过容器直接跑 Agent，执行以下命令即可：

```
docker run -d --privileged \
  --name eru_agent_$HOSTNAME \
  --net host \
  --restart always \
  -v /sys/fs/cgroup/:/sys/fs/cgroup/ \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /proc/:/hostProc/ \
  -v <HOST_CONFIG_DIR_PATH>:/etc/eru \
  projecteru2/agent \
  /usr/bin/eru-agent
```

对于生产环境而言，为了避免 Docker 自身奇怪的问题和保证 Eru 整个集群旁路控制，最好就是每台机器用 RPM 安装 Agent，只需要对 Agent 暴露 Core 的地址即可。
