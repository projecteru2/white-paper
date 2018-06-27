# 管理 Eru 集群

Eru 完全是旁路控制，因此我们只需要添加好 Pod 和 Node 后就可以操作 Eru 集群了。

### 准备工作

Eru 提供了 [cli](https://github.com/projecteru2/cli) 来操作集群。cli 可以通过容器运行：

```
docker run -it --rm \
  --net host \
  --name eru-cli \
  projecteru2/cli \
  /usr/bin/erucli <PARAMS>
```

也可以直接把 binary 放在 ```$PATH``` 下作为命令使用，通过 ```make-rpm``` 能生成 rpm 包并安装在机器中。具体的子命令和参数可以使用 ```erucli -h``` 获得

### 创建 Pod

```
docker run -it --rm \
  --net host \
  projecteru2/cli \
  erucli pod add --favor MEM ${POD_NAME}
```

Pod 是虚拟的机器组概念，Eru 不支持实体富容器Pod，但通过这种形式可以把多种相关容器限定在一组机器中组成机器层面上的**富容器**。要注意的是，除了可以自行控制内存的容器（如 Redis），一般情况下 Pod 都应该是内存优先的。

### 添加节点

创建好 Pod 之后就可以开始给 Pod 添加节点了，在被添加的已经配置好 https docker 的机器上，执行以下命令：

```
docker run -it --rm --privileged \
  --net host \
  -v /etc/docker/tls:/etc/docker/tls \
  projecteru2/cli \
  erucli node add ${POD_NAME}
```

这样节点就会加入到目标 Pod 中，然后就可以进行容器编排和部署了。