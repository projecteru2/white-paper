# CPU 资源限制在 Runtime 的实现

创建实例的时候, 经过 [调度](https://book.eru.sh/conception/scheduling) 之后, 最后 CPU 以两个属性传递给 [Runtime](https://book.eru.sh/conception/runtime), 并且由具体的 runtime 去实现对 CPU 对分配和限制.

传递给 Runtime 的两个 CPU 属性是 `CPUQuota`(int) 和 `CPUMap`(map[string]int), 前者代表一个实例最多使用几个核, 后者代表核的绑定情况.

比如:
1. Quota=1, Map=nil: 不绑核, 限制一个 CPU
2. Quota=1, Map={0:100}: 绑定0号核, 限制一个 CPU
3. Quota=1.2, Map={0:100, 1:20}: 绑定0号1号核, 分别占用 100% 和 20%, 总共限制 1.2 个 CPU

### 进程级 Runtime 对上述语义对实现

Docker 和 Systemd Runtime 都是进程级的 Runtime, 对于这种 Runtime 我们使用 Cgroups 去实现对 CPU 的限制.

1. 对于不绑核实例, 设置 cpuset.cpus 为 share cpu pool, 同时设置 cpu.cfs_quota_us 限制 CPU 时间片.
2. share cpu pool 会动态改变, 每有一个新的实例绑核, pool 就会把这个已经绑定的核摘除.
3. 对于绑核实例, 同时 CPUMap 全是完整核, 只设置  cpuset.cpus 为 CPUMap 绑定的要求.
4. 对于绑核实例, 同时 CPUMap 里有碎片核, 同时设置 cpuset.cpus 和 cpu.shares; cpu.shares 按照碎片核的占比设置.

实际用例:

1. 请求一 cpu=1, bind=false, 最终 cpuset.cpus=0-7, cpu.cfs_quota_us=100000
2. 请求二 cpu=1, bind=true, 最终 cpuset.cpus=0, cpu.shares=1024 (1024 是默认 share, 相当于没有修改); 随后 share pool 被调整为 1-7, 请求一创建的进程的 cpuset.cpus 被调整为 1-7.
3. 请求三 cpu=1.2, bind=true, 最终 cpuset.cpus=1,3, cpu.shares=20; 之后请求 1 创建的进程 cpuset.cpus 再次被调整为 2,4-7.
4. 请求四 cpu=1.2, bind=false, 最终 cpuset.cpus=2,4-7, cpu.cfs_quota_us=120000
