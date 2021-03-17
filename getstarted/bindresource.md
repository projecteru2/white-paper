# Bind Resources

### 资源

eru 目前管理了四大资源: CPU, Memory, Volume, Storage, 详细的文档在[这里](TODO).

不过我们可以先用几个典型使用场景来感受一下.

1. 限制容器的 cpu 和 memory

```
root@localhost:~# eru-cli workload deploy --pod testpod --image bash --entry zc --cpu-request 1 --cpu-limit 2 --memory-request 15M --memory-limit 15M ./spec.yaml
INFO[2021-03-15 02:26:46] [Deploy] Success ae96772e9cc60b4bc36d663cf66d0b04a849d144cad2905513b2d45ac169b8d3 zc_zc_EwUJow node1 1 2 map[] 15728640 15728640 map[] map[]
```

命令行里的 `--cpu-request 1 --cpu-limit 2` 和 `--memory-request 15M --memory-limit 15M` 就是用来指定资源配额的.

request 和 limit 的区别在[这里](TODO), 不过如果你搞不清楚的话直接让两个都配置一样的值.

查看容器的时候会显示资源:


```
root@localhost:~# eru-cli workload get ae96772e9cc60b4bc36d663cf66d0b04a849d144cad2905513b2d45ac169b8d3
┌──────────────────────────────────────────────────────────────────┬───────────────────────────┬──────────────────────────┬──────────┐
│ NAME/ID/POD/NODE                                                 │ STATUS                    │ VOLUME                   │ NETWORKS │
├──────────────────────────────────────────────────────────────────┼───────────────────────────┼──────────────────────────┼──────────┤
│ zc_zc_EwUJow                                                     │ CPUQuotaRequest: 1.000000 │ VolumePlanRequest: map[] │          │
│ ae96772e9cc60b4bc36d663cf66d0b04a849d144cad2905513b2d45ac169b8d3 │ CPUQuotaLimit: 2.000000   │ VolumePlanLimit: map[]   │          │
│ testpod                                                          │ CPUMap: map[]             │                          │          │
│ node1                                                            │ MemoryRequest: 15728640   │                          │          │
│                                                                  │ MemoryLimit: 15728640     │                          │          │
│                                                                  │ StorageRequest: 0         │                          │          │
│                                                                  │ StorageLimit: 0           │                          │          │
│                                                                  │ Privileged: false         │                          │          │
└──────────────────────────────────────────────────────────────────┴───────────────────────────┴──────────────────────────┴──────────┘
```

2. cpu 的绑定

cpu 绑定的行为稍微复杂一点, 可以看[这里](TODO).

不过可以简单理解为进程会绑定在指定的 cpu core 上.

使用上的话只要多指定一个 `--cpu-bind` 就可以了.

```
root@localhost:~# eru-cli workload deploy --pod testpod --image bash --entry zc --cpu-request 1 --cpu-limit 1 --cpu-bind ./spec.yaml
INFO[2021-03-15 03:09:06] [Deploy] Success fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736 zc_zc_sCvwqB node1 1 1 map[1:100] 536870912 536870912 map[] map[]
root@localhost:~# eru-cli workload get fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736
┌──────────────────────────────────────────────────────────────────┬───────────────────────────┬──────────────────────────┬──────────┐
│ NAME/ID/POD/NODE                                                 │ STATUS                    │ VOLUME                   │ NETWORKS │
├──────────────────────────────────────────────────────────────────┼───────────────────────────┼──────────────────────────┼──────────┤
│ zc_zc_sCvwqB                                                     │ CPUQuotaRequest: 1.000000 │ VolumePlanRequest: map[] │          │
│ fb689b8227fc274538cb7fe9d4ad81562ae3882567acef71f02da768d3c18736 │ CPUQuotaLimit: 1.000000   │ VolumePlanLimit: map[]   │          │
│ testpod                                                          │ CPUMap: map[1:100]        │                          │          │
│ node1                                                            │ MemoryRequest: 536870912  │                          │          │
│                                                                  │ MemoryLimit: 536870912    │                          │          │
│                                                                  │ StorageRequest: 0         │                          │          │
│                                                                  │ StorageLimit: 0           │                          │          │
│                                                                  │ Privileged: false         │                          │          │
└──────────────────────────────────────────────────────────────────┴───────────────────────────┴──────────────────────────┴──────────┘
```

这时候可以看到 `CPUMap` 里记录了容器绑定的 cpu.

3. 无限制的容器

在 eru 资源请求里指定 `0` 即代表无限制, 如

```
root@localhost:~# eru-cli workload deploy --pod testpod --image bash --entry zc --cpu-request 0 --cpu-limit 0 --memory-request 0 --memory-limit 0 ./spec.yaml
INFO[2021-03-15 03:13:07] [Deploy] Success efe21af8d01a526787e73d5d870ad37cd9722647b4cceed9c56ce291d099c91f zc_zc_BweUgb node1 0 0 map[] 0 0 map[] map[]
root@localhost:~# eru-cli workload get efe21af8d01a526787e73d5d870ad37cd9722647b4cceed9c56ce291d099c91f
┌──────────────────────────────────────────────────────────────────┬───────────────────────────┬──────────────────────────┬──────────┐
│ NAME/ID/POD/NODE                                                 │ STATUS                    │ VOLUME                   │ NETWORKS │
├──────────────────────────────────────────────────────────────────┼───────────────────────────┼──────────────────────────┼──────────┤
│ zc_zc_BweUgb                                                     │ CPUQuotaRequest: 0.000000 │ VolumePlanRequest: map[] │          │
│ efe21af8d01a526787e73d5d870ad37cd9722647b4cceed9c56ce291d099c91f │ CPUQuotaLimit: 0.000000   │ VolumePlanLimit: map[]   │          │
│ testpod                                                          │ CPUMap: map[]             │                          │          │
│ node1                                                            │ MemoryRequest: 0          │                          │          │
│                                                                  │ MemoryLimit: 0            │                          │          │
│                                                                  │ StorageRequest: 0         │                          │          │
│                                                                  │ StorageLimit: 0           │                          │          │
│                                                                  │ Privileged: false         │                          │          │
└──────────────────────────────────────────────────────────────────┴───────────────────────────┴──────────────────────────┴──────────┘
```

注意到 request 和 limit 是分离的语义, 所以可以指定 request=0 但是 limit>0, 代表着“不消耗 eru 资源池, 但是在操作系统层面依然做限制”; 或者 request>0 但是 limit=0, 代表“消耗 eru 资源池, 但是不做实际的限制”.

4. 挂载 volume 资源

使用 volume 之前要先注册节点上的 volume 资源.

其实 cpu 和 memory 也需要指定注册, 但是由于是系统指标可以自动采集, 所以不指定的时候默认注册节点上的全部 cpu 和 memory.

```
root@localhost:~# eru-cli pod nodes testpod
┌───────┬──────────────────────┬──────────┬───────────────────────────────┬─────────────┬─────────────┐
│ NAME  │ ENDPOINT             │ CPU      │ MEMORY                        │ VOLUME      │ STORAGE     │
├───────┼──────────────────────┼──────────┼───────────────────────────────┼─────────────┼─────────────┤
│ node1 │ tcp://127.0.0.1:2376 │ 2.00 / 4 │ 1089470464 / 6261624012 bytes │ 0 / 0 bytes │ 0 / 0 bytes │
└───────┴──────────────────────┴──────────┴───────────────────────────────┴─────────────┴─────────────┘
```

这是一开始的节点资源.

```
root@localhost:~# eru-cli node set --delta-volume /data:2G node1
INFO[2021-03-15 03:24:26] [SetNode] set node node1 success
root@localhost:~# eru-cli node get node1
┌───────┬──────────────────────┬──────────┬───────────────────────────────┬──────────────────────┬──────────────────────┐
│ NAME  │ ENDPOINT             │ CPU      │ MEMORY                        │ VOLUME               │ STORAGE              │
├───────┼──────────────────────┼──────────┼───────────────────────────────┼──────────────────────┼──────────────────────┤
│ node1 │ tcp://127.0.0.1:2376 │ 2.00 / 4 │ 1089470464 / 6261624012 bytes │ 0 / 2147483648 bytes │ 0 / 2147483648 bytes │
└───────┴──────────────────────┴──────────┴───────────────────────────────┴──────────────────────┴──────────────────────┘
```

可以看到新加了 2G 的盘 `/data`, `VOLUME` 和 `STORAGE` 都有相应的增加.


然后就可以在接下来的请求里指定 volume 了, 要写在 spec.yaml 里:

```
appname: zc
entrypoints:
  zc:
    cmd: sleep 1000000
volumes:
- /tmp:/tmp
- AUTO:/data2:rw:20000000
volumes_request:
- /tmp:/tmp
- AUTO:/data2:rw:20000000
```

命令行照旧:

```
root@localhost:~# eru-cli workload deploy --pod testpod --image bash --entry zc  ./spec.yaml
INFO[2021-03-15 04:04:16] [Deploy] Success e07fdc0ce44f332e502b3bf77b8f343fe6f5e0fa5f35b04d1f71042bf16a0026 zc_zc_ogOlab node1 1 1 map[] 536870912 536870912 map[] map[]
```

可以看到容器从节点的 `/data` volume 上划分出去里 20,000,000 bytes  的空间给容器, 节点上的资源也能反映出来被使用了这么多:

```
root@localhost:~# eru-cli workload get e07fdc0ce44f332e502b3bf77b8f343fe6f5e0fa5f35b04d1f71042bf16a0026
┌──────────────────────────────────────────────────────────────────┬───────────────────────────┬─────────────────────────────────────────────────────────────────────────────────────┬──────────┐
│ NAME/ID/POD/NODE                                                 │ STATUS                    │ VOLUME                                                                              │ NETWORKS │
├──────────────────────────────────────────────────────────────────┼───────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────┼──────────┤
│ zc_zc_ogOlab                                                     │ CPUQuotaRequest: 1.000000 │ VolumePlanRequest: map[AUTO:/data2:rw:20000000:volume:{key:"/data" value:20000000}] │          │
│ e07fdc0ce44f332e502b3bf77b8f343fe6f5e0fa5f35b04d1f71042bf16a0026 │ CPUQuotaLimit: 1.000000   │ VolumePlanLimit: map[AUTO:/data2:rw:20000000:volume:{key:"/data" value:20000000}]   │          │
│ testpod                                                          │ CPUMap: map[]             │                                                                                     │          │
│ node1                                                            │ MemoryRequest: 536870912  │                                                                                     │          │
│                                                                  │ MemoryLimit: 536870912    │                                                                                     │          │
│                                                                  │ StorageRequest: 20000000  │                                                                                     │          │
│                                                                  │ StorageLimit: 20000000    │                                                                                     │          │
│                                                                  │ Privileged: false         │                                                                                     │          │
└──────────────────────────────────────────────────────────────────┴───────────────────────────┴─────────────────────────────────────────────────────────────────────────────────────┴──────────┘
root@localhost:~# eru-cli node get node1
┌───────┬──────────────────────┬──────────┬───────────────────────────────┬─────────────────────────────┬─────────────────────────────┐
│ NAME  │ ENDPOINT             │ CPU      │ MEMORY                        │ VOLUME                      │ STORAGE                     │
├───────┼──────────────────────┼──────────┼───────────────────────────────┼─────────────────────────────┼─────────────────────────────┤
│ node1 │ tcp://127.0.0.1:2376 │ 3.00 / 4 │ 1626341376 / 6261624012 bytes │ 20000000 / 2147483648 bytes │ 20000000 / 2147483648 bytes │
└───────┴──────────────────────┴──────────┴───────────────────────────────┴─────────────────────────────┴─────────────────────────────┘
```

5. 使用 NUMA

和 Volume 一样, 在使用 NUMA 之前要先注册.

首先查看 NUMA 节点:

```
# numactl -H
available: 2 nodes (0-1)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 20 21 22 23 24 25 26 27 28 29
node 0 size: 64024 MB
node 0 free: 50032 MB
node 1 cpus: 10 11 12 13 14 15 16 17 18 19 30 31 32 33 34 35 36 37 38 39
node 1 size: 64507 MB
node 1 free: 61092 MB
node distances:
node   0   1
  0:  10  21
  1:  21  10
```

然后添加节点的时候需要指定:

```
eru-cli node add --numa-cpu 0,1,2,3,4,5,6,7,8,9,20,21,22,23,24,25,26,27,28,29 --numa-cpu 10,11,12,13,14,15,16,17,18,19,30,31,32,33,34,35,36,37,38,39 --numa-memroy 64024M --numa-memory 64507M
```

NUMA 会影响调度行为, 在计算绑核的时候会保证绑定在同一 NUMA 节点上.
