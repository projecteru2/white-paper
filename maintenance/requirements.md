# Requirements

Eru 集群只需要以下几个东西 

* [ETCD](https://github.com/coreos/etcd) 大于 3.1 的版本
* [Docker](https://www.docker.com/) 大于 17.05 的版本

以下是可选依赖

* [statsdaemon](https://github.com/bitly/statsdaemon) 用于 metrics 收集。目前 Eru 只支持 statsd 格式的数据发送，任意支持 statsd 格式的接受者都可以
* 任意日志收集器，如本地 [syslog](http://man7.org/linux/man-pages/man3/syslog.3.html) 等。

#### 安装依赖

1. ETCD 建议参考官方安装[简介](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/clustering.md)在生产环境中组成 3 节点以上的静态集群或者动态集群。不过在这里建议每一台 Node 上通过 [etcd proxy](https://coreos.com/etcd/docs/latest/v2/proxy.html) 来提升使用体验。ETCD 集群只需要一个即可，余下的每台机器上都可以批量部署 Proxy。

2. Docker 初始化可以参考以下[脚本](https://github.com/projecteru2/quickstart/blob/master/docker.sh)。以 CentOS 7 为例，每一台 Docker 机初始化的时候我们要注意这些地方：

    * IP 一定是要 Core 能访问到的 IP，多网卡的情况下可以手动指定，单网卡可以参考以下脚本：

    ```
    export IP=$(ip addr | grep 'state UP' -A2 | tail -n1 | awk '{print $2}' | cut -f1  -d'/')
    echo 'Current ip:' ${IP}
    ```

    * 配置 Docker 的时候直接把配置存在 ```/etc/docker/daemon.json``` 即可，如果是在中国大陆可以加入以下2个选项来提升体验：
    ```
    "dns": ["114.114.114.114"]
    "registry-mirrors": ["https://registry.docker-cn.com"]
    ```

    * 创建 tls 的时候一定要和选定的访问 IP 一致

    因此最后我们得到脚本如下：

    ```
    yum install -y docker-ce
    mkdir -p /etc/docker/tls
    echo "{
        \"hosts\": [\"unix:///var/run/docker.sock\", \"tcp://${IP}:2376\"],
        \"tlsverify\": true,
        \"tlscacert\": \"/etc/docker/tls/ca.crt\",
        \"tlscert\": \"/etc/docker/tls/server.crt\",
        \"tlskey\": \"/etc/docker/tls/server.key\",
        \"cluster-store\": \"etcd://${ERU_ETCD}\"
    }" > /etc/docker/daemon.json
    openssl req -x509 -newkey rsa:2048 -nodes -keyout ca.key -out ca.crt -days 3650 -subj /C=CN
    openssl req -newkey rsa:2048 -nodes -keyout server.key -out server.csr -subj /CN=${IP}
    openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt -days 3650
    openssl req -newkey rsa:2048 -nodes -keyout client.key -out client.csr -subj /CN=client
    openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in client.csr -out client.crt -days 3650
    chmod 600 ca.key client.key server.key
    rm -rf server.csr client.csr
    mv ca.* client.* server.* /etc/docker/tls
    ```

完成机器的配置后，就可以来部署安装 Eru 了。
