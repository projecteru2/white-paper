# Setup

在 Linux 拉起来一个最小的 eru 单点集群.

### 前置配置

节点需要安装 Docker 并且把 Docker 配置为监听 `0.0.0.0:2376`.

### 部署一个 etcd

单节点就行了, 用容器跑起来超简单.

```
docker run -d --net host --name etcd --restart always quay.io/coreos/etcd:v3.4.9 /usr/local/bin/etcd --enable-v2 --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://etcd:2379
```

### 准备一个配置文件

这是 eru-core 的配置文件, 详细的配置在[这里](https://github.com/projecteru2/core/blob/master/core.yaml.sample)

```
mkdir -p /etc/eru
cat <<! > /etc/eru/core.yaml
log_level: "DEBUG"
bind: ":5001"
service_address: 127.0.0.1:5001
statsd: "127.0.0.1:8125"
profile: ":12346"
global_timeout: 300s
lock_timeout: 30s
cert_path: "/tmp"
max_concurrency: 20

grpc:
    max_concurrent_streams: 100

etcd:
    machines:
        - "http://localhost:2379"
    prefix: "/eru"
    lock_prefix: "core/_lock"

docker:
    log:
      type: "json-file"
      config:
        "max-size": "10m"
    network_mode: "bridge"
    hub: "hub.docker.com"
    namespace: "projecteru2"
    build_pod: "local"
    local_dns: true

scheduler:
    maxshare: -1
    sharebase: 100
!
```

### 起服务

```
docker run -d --name eru-core --restart always -v /etc/eru:/etc/eru -v /root/.ssh:/root/.ssh --net host projecteru2/core eru-core
```

安装命令行 eru-cli

```
alias eru-cli='docker run -it -v $(pwd):/src -w /src --net host projecteru2/cli eru-cli'
echo alias eru-cli=\'docker run -it -v $(pwd):/src -w /src --net host projecteru2/cli eru-cli\' >> ~/.bashrc
```

运行我们的第一个 eru 命令:

```
eru-cli pod list
```

成功的话能看到输出:

```
┌──────┬─────────────┐
│ NAME │ DESCRIPTION │
├──────┼─────────────┤
└──────┴─────────────┘
```

因为还是一个空集群, 不过:

**Welcome to Eru world!**

### 加节点

就把自己所在的[节点](https://book.eru.sh/conception/node)加入 eru-core:

需要先把 dockerd 设置为绑定 2376 端口:

```
sed -i 's!ExecStart=.*!\0 -H tcp://0.0.0.0:2376!' /lib/systemd/system/docker.service
systemctl daemon-reload && systemctl restart docker
```

然后就可以在 eru-core 里注册[节点](https://book.eru.sh/conception/node)了. 不过你需要先添加一个 [pod](https://book.eru.sh/conception/pod)

```
eru-cli pod add testpod
eru-cli node add --nodename node1 --endpoint tcp://127.0.0.1:2376 testpod
```

顺利的话你能看到正确的输出:

```
root@localhost:~# eru-cli pod add testpod

┌─────────┬─────────────┐
│ NAME    │ DESCRIPTION │
├─────────┼─────────────┤
│ testpod │             │
└─────────┴─────────────┘
root@localhost:~# eru-cli node add --nodename node1 --endpoint tcp://127.0.0.1:2376 testpod
┌───────┬──────────────────────┬──────────┬──────────────────────┬─────────────┬─────────────┐
│ NAME  │ ENDPOINT             │ CPU      │ MEMORY               │ VOLUME      │ STORAGE     │
├───────┼──────────────────────┼──────────┼──────────────────────┼─────────────┼─────────────┤
│ node1 │ tcp://127.0.0.1:2376 │ 0.00 / 4 │ 0 / 6261624012 bytes │ 0 / 0 bytes │ 0 / 0 bytes │
└───────┴──────────────────────┴──────────┴──────────────────────┴─────────────┴─────────────┘
```

接下来我们可以开始创建应用了.
