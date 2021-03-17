# SDN

Eru 目前的 SDN 其实是独立于 eru-core, 而是让各个 runtime 实现各自的 SDN.

### Docker

Eru 提供了基于 Calico 的 fixed-ip CNM 插件: https://github.com/projecteru2/barrel/

它要解决的问题是, 一个容器被 restart (stop && start) 前后的 IP 可能会发生变化, 而对于某些业务来说这是不可接受的风险.

因此使用 eru-barrel CNM 插件之后, 一个 fixed-ip 容器流程是这样的:

1. `docker -H unix:///var/run/barrel.sock run -td --name zc2 --net clouddev --label 'fixed-ip=1' bash bash`: 正常创建容器, 只是多加一个 label `fixed-ip=1`
2. `docker exec -it zc2 ip a sh cali0` 显示容器 IP
2. `docker stop -t0 zc2` 停止容器
3. `calicoctl ipam show --ip ` 看到 IP 并没有归还给 calico ippool
4. `docker start zc2` 重启后可以检查 IP 没有改变

Barrel 的配置和安装请参考[文档](TODO).

### Yavirt

Yavirt 当前的集成 Calico 方案是基于 tun/tap, 不过在不久的将来会拆解到 CNI.

Yavirt 网络设置请参考[文档](TODO).
