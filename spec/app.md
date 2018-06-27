# Spec

#### 定义

Spec 是用来定义如何操作源码或者镜像的 Yaml 描述文件。在 Spec 中分2个部分

* App 描述
* Build 描述

这2个部分可以放在一个文件中，也可以分开放。Cli 读取之后会分别进行部署操作或者打包操作。

#### App

一个标准的 App 描述如下：

```
appname: "{APPNAME}"                        Application ident
entrypoints:
  {ENTRYPOINT_A}:                           Application role A
    cmd: "run something"                    How to run this role
    restart: always/""                      Always means alway restart when failed, otherwise it will try 3 times then stop
    privileged: true/false                  Run in privileged mode
    log_config: "journald/none"             Same as docker log config
    dir: "path"                             Working dir
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
      force: true/false
  {ENTRYPOINT_B}:                           Application role B
        ...
  {ENTRYPOINT_C}:                           Application role C
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
