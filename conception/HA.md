# HA

eru-core 提供了 Go SDK, 只要客户端使用 SDK 与 eru-core 通讯, 就能自动发现多实例的 eru-core, 并在部分 eru-core 实例宕的时候自动切换到其他实例.

它的实现分为服务端和客户端两部分.

### 服务端

eru-core 会不断把自己的 outbound IP 注册到 etcd 上, 其中 outbound IP 是通过 `net.Dial("8.8.8.8")` 后检查 src IP 获得的.

注册到 etcd 上到地址绑定了 lease, 如果没有及时被 keepalive 会被超时删除, 这就是 eru-core 的心跳机制, 如果节点宕, 或者进程崩溃, 进程阻塞, 那么在 keepalive interval 之后会被探测到, 并从可用 eru-core 实例池里剔除.

### 客户端

eru-core SDK 会注册 eru resolver, 该 resolver 会有个 goroutine watch eru-core API `WatchServiceStatus`, 从而获得最新到可用 core 实例地址. 一旦有变化(新增或者减少), 都会通过 gRPC Resolver 机制动态生效.

gRPC 提供了 LB per Call 的机制, 也就是说用户先后发送两个请求给 eru-core, 每次都会被 Round Robin 重新计算一次, 选择一个可用的链接发送请求, 这比 HTTP1.1 长链接高明不少.

此外 eru-core SDK 还定制了 Interceptor, 让各种 Stream 失败的时候能够重试, 从而新选择一个可用的连接.
