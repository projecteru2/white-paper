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

在安装好 Core 之后，集群实际上还是不能启动的，需要先注册 Pod。在注册 Pod 之前我们需要选择使用哪种模型的 Pod，对于 Eru 而言分 CPU 优先（favor CPU）和 MEM 优先（favor MEM）。Pod 是虚拟的机器组概念，Eru 不支持实体富容器Pod，但通过这种形式可以把多种相关容器限定在一组机器中组成机器层面上的**富容器**。对于 CPU 优先的 Pod 中每个节点在部署容器的时候均会精确的分配容器所占用 CPU，内存控制交由容器内进程自行控制。对于自带内存控制又对 CPU 性能敏感的应用如 redis 而言，是一种比较好的模式。但默认情况下，一般都是 MEM 优先。内存优先的模型下，Eru 也会从全局的角度分配 CPU 运算量给容器，在机器空闲的时候这个 CPU 运算量是可以保证的，而机器繁忙的时候用户得注意 load 值然后及时的通过 API 来迁移容器。这个模式下内存是强限制，CPU 可以超售。

### 添加节点

创建好 Pod 之后就可以开始给 Pod 添加节点了，在被添加的已经配置好 https docker 的机器上，执行以下命令：

```
docker run -it --rm --privileged \
  --net host \
  -v /etc/docker/tls:/etc/docker/tls \
  projecteru2/cli \
  erucli node add ${POD_NAME}
```

这样节点就会加入到目标 Pod 中，然后就可以进行容器编排和部署了。就是在注册的时候时候建议冗余一些 CPU 和内存给 OS 本身。比如 4Core 的机器写 3Core，128G 内存注册为 120G 等。
