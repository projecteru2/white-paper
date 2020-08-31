## 安装的依赖和服务

运行完 [quickstart.sh](https://github.com/projecteru2/quickstart/blob/master/quickstart.sh) 之后，机器上自动安装并配置好的依赖和服务包括：

### ETCD

Quickstart 会默认安装 v3.3.4 版本的 ETCD，并启动；一些重要的默认配置包括：

```ini
name: etcd0
data-dir: /var/lib/etcd
listen-peer-urls: http://0.0.0.0:2380
listen-client-urls: http://0.0.0.0:2379
enable-v2: true
```

### Docker

Docker 会启动并监听在默认的 :2376 端口上，并使用上述配置的 ETCD 作为 cluster store

```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"],
  "cluster-store": "etcd://127.0.0.1:2379"
}
```

Docker 启动成功后会自动注册一个名为 "testpool" 的 Calico network 到该 Docker 集群。

### Calico node

Quickstart 会确保两项 Kernel 配置为期望值，包括激活 net.ipv4.ip_forward 和设置 net.netfilter.nf_conntract_max 值为 1000000；之后会下载并以 Container 形式启动 v3.4 版本的 Calico node；启动成功后会自动创建名为 "testpool" 的 IPPool，该 IPPool 的 CIDR 默认为 10.10.0.0/16

### ERU Core

ERU Core 将运行在默认的 :5001 端口，并将所有的元数据信息存储在 ETCD 的 /eru-core 目录下；启动成功后，Quickstart 会下载保存 eru-cli 命令行工具，并配置相应的 PATH 信息到 ~root 账号，以供未来使用；eru-cli 准备完毕后会自动注册一个名为 "eru" 的 POD 到 ERU Core，并在该 POD 下自动新建一个名为 "docker0" 的节点。

## How to Quickstart

### 运行时依赖

- Ubuntu >=16.04
- ansible >=2.9.10
- jmespath >=0.10.0

### Quickstart

准备好上述依赖后，即可在目标机器上执行

```bash
curl https://raw.githubusercontent.com/projecteru2/quickstart/master/quickstart.sh | bash
```

### 升级

重复执行 [quickstart.sh](https://github.com/projecteru2/quickstart/blob/master/quickstart.sh) 命令即可。