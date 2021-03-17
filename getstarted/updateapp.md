# Update Application

应用是需要更新升级的, 这里总结出几大常用更新/升级场景.

### 更新资源

比如拿 cpu 来说, 从单核扩容到三核, 再缩减到两核, 再扩展到无限制, 可以用以下命令:

```
root@localhost:~# eru-cli workload realloc --cpu-request 2 --cpu-limit 2 fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736
INFO[2021-03-15 06:50:15] [Realloc] Success
root@localhost:~# eru-cli workload get fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736
┌──────────────────────────────────────────────────────────────────┬────────────────────────────────┬──────────────────────────┬──────────┐
│ NAME/ID/POD/NODE                                                 │ STATUS                         │ VOLUME                   │ NETWORKS │
├──────────────────────────────────────────────────────────────────┼────────────────────────────────┼──────────────────────────┼──────────┤
│ zc_zc_sCvwqB                                                     │ CPUQuotaRequest: 3.000000      │ VolumePlanRequest: map[] │          │
│ fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736 │ CPUQuotaLimit: 3.000000        │ VolumePlanLimit: map[]   │          │
│ testpod                                                          │ CPUMap: map[1:100 2:100 3:100] │                          │          │
│ node1                                                            │ MemoryRequest: 536870912       │                          │          │
│                                                                  │ MemoryLimit: 536870912         │                          │          │
│                                                                  │ StorageRequest: 0              │                          │          │
│                                                                  │ StorageLimit: 0                │                          │          │
│                                                                  │ Privileged: false              │                          │          │
└──────────────────────────────────────────────────────────────────┴────────────────────────────────┴──────────────────────────┴──────────┘
```

注意 `realloc` 接口接受的是增量, 因此 `--cpu-request 2` 语义是让容器新增 2 个 cpu 配额, 最终用个 3 cpu.

缩减的话就指定负数:

```
root@localhost:~# eru-cli workload realloc --cpu-request -1 --cpu-limit -1 fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736
```

无限制的话只要把当前的核数减到 0:

```
root@localhost:~# eru-cli workload realloc --cpu-request -2 --cpu-limit -3 fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736
```

也可以在绑核与不绑核之间转换状态:

```
eru-cli workload realloc --cpu-bind fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736
eru-cli workload realloc --cpu-unbind fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736
```

当然其他类型的资源可以如法炮制.

### 更新镜像

更新镜像可以采用 replace 的办法:

```
eru-cli workload replace --pod testpod --image python --entry zc ./spec.yaml
```

这句命令让容器从之前的 `bash` 镜像换到了 `python` 镜像上.

在实际应用上, 一般在发版后升级服务.

注意 replace 命令是不更新资源的, 即使你在请求里指定里新的资源也会被无视.

### 扩容

扩容也是很常见的需求, 我们可以通过 `--count` / `--nodes-limit` / `--deploy-strategy` 来指定.

一个简单的例子, 比如在之前已有一个实例的基础上要再加一个:

```
eru-cli workload deploy --pod testpod --image bash --entry zc --count 2 ./spec.yaml
```

就可以部署第二个容器, 不过依然在同一节点上.

在多节点的情况下 `deploy-strategy` 和 `nodes-limit` 有很重要的作用, 详见 [strategy](TODO).
