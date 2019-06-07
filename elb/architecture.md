# 架构

## 基本概念

* **backend** ELB 中 backend 是一条 [dyups](https://github.com/yzprofile/ngx_http_dyups_module) 中的记录。每条记录都是一个 k-v 对，key 是 backend 的名字和对应的容器 ip。

* **rule** rule 是保存在 ELB [sharedict](https://github.com/openresty/lua-nginx-module#ngxshareddict) 中的记录，也是 k-v 对。key 是绑定的域名，value 是对应的分流规则。 ELB 中的分流规则是一个 json 格式的树状结构，它可以支持多级条件分流。

## redis

ELB 中 redis 主要有两个作用：

* 数据存储，所有 backend 和 rule 的数据都会保存在 redis 中，更新时，ELB 会在启动阶段从 redis 载入数据。

* 服务发现，通过 redis pub/sub 可以更新集群内所有 ELB 的数据。

## monitor

ELB 通过 [ngx.timer.at](https://github.com/openresty/lua-nginx-module#ngxtimerat) 启动一个守护进程(monitor)监听 redis。当有新域名需要绑定，或者有容器上下线的时候， 向 redis publish 相关信息，然后 monitor 会 subcribe 这个信息，并更新 ELB 数据。

## erulbpy

因为设计缺陷，ELB 原生的更新接口并不好用，于是我们实现了 erulbpy —— 作为 ELB 的 client 它会将更新封装成 ELB 要求的数据格式（主要是生成 ELB rule json），然后更新 ELB。

## API

ELB 保留了两个查询接口：

* `/__erulb__/upstream`: 查询 ELB 上的 backend。

* `/__erulb__/rule`: 查询 ELB 上的 rule。