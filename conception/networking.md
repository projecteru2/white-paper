# Networking

eru-core 的 SDN 网络主要由 Runtime 层实现, 用户请求里的 `network` 会被透传.

1. Docker Runtime: 只要有对应的 CNM 插件, 创建了对应都 docker network, 那么用户指定的 network 就能成功创建.
2. Yavirt Runtime: 绑定使用 Calico SDN, 同 docker network 类似, 需要先在 yavirt 中注册网络与 CIDR.
3. Systemd Runtime: 通过 CNI 把 daemon 扔进 Calico SDN, 尚在施工中.

这套机制的好处是:

1. BGP-based: 可通过 Router Reflector 与现有基础设施集成
2. Pure L3: 无封包, 高性能
3. CNI/CNM-based: 成熟的 SDN API, 官方背书
4. 统一网络, 便于混合编排容器和虚拟机

除此之外, eru-core 提供了两套 CNM 插件: eru-minions, eru-barrel

### Minions

Minions 是一个 Calico SDN 的 CNM 插件, 用于在 etcd v3 上打通 dockerd 和 calico-node.

Minions 虽然稳定, 但是推荐使用 eru-barrel, 提供了更多的功能.

### Barrel

Barrel 也是一个 Calico SDN 的 CNM 插件, 但是与 Minions 相比, 他额外提供了 fix-ip 的功能.

先创建一个有 `fixed-ip` label 的容器:

```
docker run -td --label fixed-ip --network calico-sdn bash bash
```

然后把它停止, barrel 保证它的 ip 不会被之后的新容器占用, 因此可以安全有保障地 restart 且拥有相同 ip.

wiki: https://github.com/projecteru2/barrel/wiki/Quick-Start
