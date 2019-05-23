# App Spec

#### 定义

App Spec 是用来定义如何操作源码或者镜像的 Yaml 描述文件。在 Spec 中分2个部分

* App 描述
* Build 描述

这2个部分可以放在一个文件中，也可以分开放。Cli 读取之后会分别进行部署操作或者打包操作。

#### App

一个标准的 App 描述如下：

```
appname: "{APPNAME}"                        Application ident
entrypoints:
  {ENTRYPOINT_A}:                           Application role A
    cmd: "run something"                    How to run this role, depend on image
    privileged: true/false                  Run in privileged mode
    dir: "path"                             Working dir
    log: 
      type: "journald/none"                 Same as docker log driver name
      config:                               Same as docker log driver config
        config1: "value"
    publish:                                Which port(s) bind this Application
      - "12345"
      - "7788"
      ...
    healthcheck:
      tcp_ports:                            Tcp port check
        - "12345"
        - "7788"
        ...
      http_port: "80"                       A http port for advance checking
      url: "/lol"                           Check url
      code: 200                             Which code is correct
    hook:
      after_start:                          Some commands run after start
        - cmd1
        - cmd2
        ...
      before_stop:                          Some commands run before stop
        - cmd1
        - cmd2
        ...
      force: true/false                     Force to stop container if it set true and before_stop failed.
    restart: always/""                      Always or empty. Always means alway restart when failed, otherwise it will try 3 times then stop
    sysctls:
      net.ipv4.ip_forward: "1"
  {ENTRYPOINT_B}:                           Application role B
        ...
  {ENTRYPOINT_C}:                           Application role C
        ...
volumes:                                    Mount local dir insider container/vm
  - path1:path2:ro
  - path3:path4
  ...
labels:                                     Add labels to container/vm
  label1: "something"
  label2: "anything"
  ...
dns:                                        User define dns
  - {DNS1}
  - {DNS2}
  ...
extra_hosts:                                Add hosts item in host file
  - domain1:IP1
  - domain2:IP2
  ...
```

#### Build

一个标准的 Build 描述如下：

```
stages:                                                 Define stages
  - {BUILD_STAGE_A}
  - {BUILD_STAGE_B}
  - {BUILD_STAGE_C}
  ...
builds:
  {BUILD_STAGE_A}:
    base: "golang:1.10.3-alpine3.7"                     Base image
    repo: "git@github.com:projecteru2/agent.git"        Github/gitlab repo, only support ssh protocol
    version: "HEAD"                                     Version sha
    dir: "/go/src/github.com/projecteru2/agent"         Working dir
    submodule: true                                     Prepare submodules
    commands:                                           Building commands
      - cmd1
      - cmd2
      ...
    envs:                                               Building envars
      A: B
      C: D
      ...
    args:                                               Building args same as dockerfile definition
      A: B
      C: D
      ...
    labels:                                             Building labels
      A: B
      C: D
    artifacts:                                          If provide, eru will earse whole working dir and download artifact in it
      ident1: path1
      ident2: path2
      ...
    cache:                                              If provide, next stage will get those things to working dir.
      src_path1: dst_path1
      src_path2: dst_path2
      ...
  {BUILD_STAGE_B}:
    ...
  {BUILD_STAGE_C}:
    ...
```
