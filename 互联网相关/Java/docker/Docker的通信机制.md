2021-09-11
12:20:22
author:陈建浩


--- 

## Docker中的通信机制和网络功能

Docker 允许通过外部访问容器或容器互联的方式来提供网络服务。
随着 Docker 网络的完善，强烈建议大家将容器加入自定义的 Docker 网络来连接多个容器，而不是使用 `--link` 参数。


### 新建网络
```sh
$ docker network create -d bridge my-net
```

`-d` 参数指定 Docker 网络类型，有 `bridge` `overlay`。其中 `overlay` 网络类型用于 Swarm mode。
