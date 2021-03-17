# Sample Application

### 第一个应用

“应用”在 Eru 中用于描述一个可部署的项目, 详细的文档在[这里](https://book.eru.sh/conception/application)

部署 eru 应用需要写一个 spec.yaml, 详细的 spec 在[这里](https://book.eru.sh/specs/app).

不过最简单的 spec 写成这样就可以了:

```
cat <<! >spec.yaml
appname: zc
entrypoints:
  zc:
    cmd: sleep 1000000
!
```

然后就可以创建出第一个应用:

```
root@localhost:~# eru-cli workload deploy --image bash --pod testpod --entry zc spec.yaml
INFO[2021-03-12 10:03:39] [Deploy] Success 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f zc_zc_cuQhQH node1 1 1 map[] 536870912 536870912 map[] map[]
```

`--image bash` 指定了使用 bash 镜像; `--pod testpod` 指定使用刚才创建的 pod (里面有一个节点); `--entry` 指定 spec 里的 `zc`.

### Workload

Workload 可以先参看[文档](https://book.eru.sh/conception/workload)

1. 查看已部署的 workload

```
root@localhost:~# eru-cli workload get 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f
┌──────────────────────────────────────────────────────────────────┬───────────────────────────┬──────────────────────────┬──────────┐
│ NAME/ID/POD/NODE                                                 │ STATUS                    │ VOLUME                   │ NETWORKS │
├──────────────────────────────────────────────────────────────────┼───────────────────────────┼──────────────────────────┼──────────┤
│ zc_zc_cuQhQH                                                     │ CPUQuotaRequest: 1.000000 │ VolumePlanRequest: map[] │          │
│ 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f │ CPUQuotaLimit: 1.000000   │ VolumePlanLimit: map[]   │          │
│ testpod                                                          │ CPUMap: map[]             │                          │          │
│ node1                                                            │ MemoryRequest: 536870912  │                          │          │
│                                                                  │ MemoryLimit: 536870912    │                          │          │
│                                                                  │ StorageRequest: 0         │                          │          │
│                                                                  │ StorageLimit: 0           │                          │          │
│                                                                  │ Privileged: false         │                          │          │
└──────────────────────────────────────────────────────────────────┴───────────────────────────┴──────────────────────────┴──────────┘
```

2. 停止, 重启

```
root@localhost:~# eru-cli workload stop 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f
INFO[2021-03-12 10:11:26] [ControlWorkload] 38078e6
root@localhost:~# eru-cli workload start 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f
INFO[2021-03-12 10:11:34] [ControlWorkload] 38078e6
root@localhost:~# eru-cli workload restart 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f
INFO[2021-03-12 10:11:55] [ControlWorkload] 38078e6
```

3. exec

执行一个非交互式命令

```
root@localhost:~# eru-cli workload exec 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f ls
bin
dev
etc
home
lib
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

执行一个交互式命令

```
root@localhost:~# eru-cli workload exec -i 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f bash
bash-5.1# ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
```

4. 把文件发送到容器内


```
root@localhost:~# eru-cli workload send --file ./spec.yaml:/a.yaml 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f
INFO[2021-03-12 10:21:59] [Send] Send /a.yaml to 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f success

root@localhost:~# eru-cli workload exec 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f cat /a.yaml
appname: zc
entrypoints:
  zc:
    cmd: sleep 1000000
```

5. 删除容器

```
root@localhost:~# eru-cli workload remove -f 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f
WARN[2021-03-12 11:01:09] [RemoveWorkload] If workload not stopped, force to remove will not trigger hook process if set
INFO[2021-03-12 11:01:09] [RemoveWorkload] 38078e63ac63c4f8d805ed3d0b94ab2ab23333f9e05ac944e9028571fc0a065f Success
```
