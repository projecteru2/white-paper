# 安装 Calico (optional)

每台机器上安装配置 Calico 需要多步配置：

### 安装 calicoctl 和启动 calico-node

需要注意的是，最新的 ```calicoctl``` 和 ```calico-node``` 只支持 ETCD v3 协议下的存储模型了，因此过去的 ```libnetwork-plugin``` 是不包含在 calico-node 的镜像中，也不会启动。

```
export ERU_ETCD=ETCD_IP_PORT
export CALICOCTL_VER=v3.4.0
ls /usr/bin | grep calicoctl &> /dev/null || curl -L https://github.com/projectcalico/calicoctl/releases/download/${CALICOCTL_VER}/calicoctl -o /usr/bin/calicoctl
chmod +x /usr/bin/calicoctl

mkdir -p /etc/calico
echo "apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "etcdv3"
  etcdEndpoints: "http://${ERU_ETCD}"
" > /etc/calico/calicoctl.cfg

calicoctl node run --node-image=calico/node
```

执行完毕后，```calico-node``` 就应该启动了。默认情况下 calico 是跑在 BGP Mesh 网络中的，节点中互相 mesh。对于超过100台机器的情况下这个网络模型是不太合适的，一来不适合管理二来性能受限。因此在超大集群模式下，推荐阅读 [Calico 网络模型](https://docs.projectcalico.org/v3.1/reference/private-cloud/l3-interconnect-fabric) 来选择适合自己的网络模式。

### 安装对 Docker 的支持

受限于最新的 [calico-libnetwork-plugin](https://github.com/projectcalico/libnetwork-plugin) 已经不再支持 v3 版本的 calico，Eru 做了一个 port 称之为 [eru-minions](https://github.com/projecteru2/minions)。部署方式和之前的的 libnetwork plugin 一样，但做了对最新的 Calico 支持。具体的修改的话可以参考对上游的 [PR](https://github.com/projectcalico/libnetwork-plugin/pull/183)。

因为 Docker 在 Crash 后重启时会先尝试恢复网络去调用 plugin，如果 plugin 同时也需要 docker 中容器的信息就会导致循环依赖从而 block all。因此插件的安装仅推荐使用 RPM 方式裸安装，通过 ```systemctl``` 运行。

在 Minions 项目中，可以通过 make-rpm 得到 RPM 包，然后执行 ```rpm -i``` 安装即可。安装完毕后只需要修改 ```/etc/eru/minions.conf``` 指定 ETCD 即可启动。启动后，在 docker 的 plugin 文件下应该就会出现 ```calico.sock``` 和 ```calico-ipam.sock``` 2个文件了。

### 系统配置

Calico 使用 3 layer 网络进行转发，因此每一台跑了 calico-node 的机器需要对 2 个系统参数修改：

```
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
echo 'net.netfilter.nf_conntrack_max=1000000' >> /etc/sysctl.conf
sysctl -p
```

### 配置 IP Pool

当我们配置子网的时候，随便找一台 calico 的 node 执行配置即可：

```
export NETPOOL=10.213.0.0/16
export NETNAME="etest"

cat << EOF | calicoctl create -f -
- apiVersion: projectcalico.org/v3
  kind: IPPool
  metadata:
    name: ${NAETNAME}
  spec:
    natOutgoing: true
    cidr: ${NETPOOL}
EOF
docker network create --driver calico --ipam-driver calico-ipam --subnet ${NETPOOL} ${NETNAME}
```

注意，不要采用与物理网络同样的子网。

### 测试

当我们完成这一切的时候，可以通过 ```docker -it --rm --network ${NETNAME} alpine ifconfig``` 看到是否添加了一块名叫 ```cali0``` 的网卡，这时候从任意 calico node 或者其他同一子网的其他容器 ping 通这个容器了。

具体 IP Pool 的配置可以参考[这篇](https://docs.projectcalico.org/v3.1/reference/calicoctl/resources/ippool)，余下的配置都可以参考 [Calico 官网的手册](https://docs.projectcalico.org/v3.1/reference/)。