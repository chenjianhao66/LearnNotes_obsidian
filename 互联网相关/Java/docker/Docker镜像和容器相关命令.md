2021-09-02
21:41:53
author:陈建浩
#Docker #容器技术 

--- 


# 一、镜像相关命令
## 1、查看本机所有镜像
```bash
docker images
```
参数
- q：只显示镜像id
- a：列出所有镜像

## 2、获取一个新的镜像
```bash
docker pull [images name]:[tag]

# 示例
docker pull mysql:8.0.26
```
下载完成后，我们可以直接使用这个镜像来运行容器。

## 3、搜索镜像
我们可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为： **[https://hub.docker.com/](https://hub.docker.com/)**

我们也可以使用 docker search 命令来搜索镜像。比如我们需要一个 httpd 的镜像来作为我们的 web 服务。我们可以通过 docker search 命令搜索 httpd 来寻找适合我们的镜像。

## 4、删除镜像
镜像删除使用 **docker rmi** 命令，比如我们删除 hello-world 镜像：
```bash
$ docker rmi hello-world
```
参数
- f：强制删除


## 5、 创建镜像

当我们从 docker 镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。

-   1、从已经创建的容器中更新镜像，并且提交这个镜像
-   2、使用 [[Dockerfile]] 指令来创建一个新的镜像


# 二、容器相关命令

## 1、创建容器
```bash
docker run [参数] [镜像名]
```
参数：
- i：交互式操作
- t：终端
- d：后台运行
- p：映射的端口 
	-  宿主机端口：容器端口
- name：容器名称，注意是2个--
- v：挂在数据卷
## 2、查看容器
查看目前正在运行的容器
```bash
docker ps
```

查看所有容器
```bash
docker ps -a
```

## 3、启动一个已停止的容器
```bash
docker start [容器id]
```

## 4、停止一个容器
```bash
docker stop [容器id]
```

## 5、进入容器
```bash 
docker exec [参数] [容器id] [进入容器的命令界面]

# 示例
docker exec -it 8393c9e84f5f bash

以交互式、终端的方式进入一个 id为8393c9e84f5f的容器，bash界面进去
```

## 6、删除容器

删除容器使用 **docker rm** 命令：
```bash
$ docker rm -f 1e560fca3906
```

## 7、容器宿主机之间的文件操作
```bash
docker cp 文件|目录 容器id:容器路径           -----------------   将宿主机复制到容器内部
docker cp 容器id:容器内资源路径 宿主机目录路径  -----------------   将容器内资源拷贝到主机上
```

## 8、数据卷实现容器与宿主机目录共享
```bash
docker run -v 宿主机的路径|任意别名:/容器内的路径 镜像名
```

如果使用别名的形式实现数据卷的话，该卷会在 `/var/lib/docker/别名`创建目录
> 可以使用 `find / -name 别名` 命令来查询该别名目录在 `/` 根的哪个位置


如果对宿主机的目录进行改变就可以实现容器内的目录改变。

## 9、查看容器内部细节
```bash
docker inspect 容器id 		------------------ 查看容器内部细节
```


```
[
    {
        "Id": "8393c9e84f5ff69a8e1c29cc3d79c410abc746a81c91bd5ffeb7615cb4555f9c",
        "Created": "2021-08-27T14:53:12.985973618Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "rabbitmq-server"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 1393,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-09-11T03:53:37.159853622Z",
            "FinishedAt": "2021-09-11T03:49:27.769576198Z"
        },
		
        
		"Image": "sha256:f4acdf2a0a61fb7f190ddd889964fa37a91b6cad6c04a8de9ada89d1ff78b97f",
        "ResolvConfPath": "/var/lib/docker/containers/8393c9e84f5ff69a8e1c29cc3d79c410abc746a81c91bd5ffeb7615cb4555f9c/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/8393c9e84f5ff69a8e1c29cc3d79c410abc746a81c91bd5ffeb7615cb4555f9c/hostname",
        "HostsPath": "/var/lib/docker/containers/8393c9e84f5ff69a8e1c29cc3d79c410abc746a81c91bd5ffeb7615cb4555f9c/hosts",
        "LogPath": "/var/lib/docker/containers/8393c9e84f5ff69a8e1c29cc3d79c410abc746a81c91bd5ffeb7615cb4555f9c/8393c9e84f5ff69a8e1c29cc3d79c410abc746a81c91bd5ffeb7615cb4555f9c-json.log",
        "Name": "/myrabbit",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": [
            "34b6b7fc5695be25dda581f2bf7fa3ae985014cf024208c39ab636215e6a0ea6"
        ],
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "15672/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "15672"
                    }
                ],
                "1883/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "1883"
                    }
                ],
                "25672/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "25672"
                    }
                ],
                "5672/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "5672"
                    }
                ],
                "61613/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "61613"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                30,
                120
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/b73dc78ffd6f92b0ebf42b6e36d235c66bbbbc167837cddaa757f488b6214918-init/diff:/var/lib/docker/overlay2/e6be50b1e2ab7976aa755fb7697d8b770b26db22e6ca14ee71b24886dd36731a/diff:/var/lib/docker/overlay2/f4c3edc946f937b32498f037249c51edce23e15f64d5aecf817b1f9687829415/diff:/var/lib/docker/overlay2/32b85557a958da6e48cc108b4f6bf29da80dec2f0ea605047f03a3a5596dc57f/diff:/var/lib/docker/overlay2/0d9ca425bafb28178576b40b1c4920c0ac7740a9e7eafbc7224570314ac94508/diff:/var/lib/docker/overlay2/15031d31eaf0fefe76358babde213b9556f67cacf847d5067bb6af0f3998c391/diff:/var/lib/docker/overlay2/037d6f76bb40e8bb17eb371c99ce1fc3c8d2b8db14a1eeb8dc4cf5d98fef4203/diff:/var/lib/docker/overlay2/204daec9b90adaf75c3bb273706d7368c646c7a47095b4902cd2b08c60b656b3/diff:/var/lib/docker/overlay2/dc1545802075b2ec6274a0b2585ada5be7c61f30986a12778eeaef3c30d5f9e0/diff:/var/lib/docker/overlay2/8a5e40ec2842aa3a0ac8655df26d49b5c01b21000ed314818fc50dbb5fb70d27/diff:/var/lib/docker/overlay2/8e22e456bc136e3448f82a098dc945731c69b6029a0cdf8c10c6020bff26df22/diff",
                "MergedDir": "/var/lib/docker/overlay2/b73dc78ffd6f92b0ebf42b6e36d235c66bbbbc167837cddaa757f488b6214918/merged",
                "UpperDir": "/var/lib/docker/overlay2/b73dc78ffd6f92b0ebf42b6e36d235c66bbbbc167837cddaa757f488b6214918/diff",
                "WorkDir": "/var/lib/docker/overlay2/b73dc78ffd6f92b0ebf42b6e36d235c66bbbbc167837cddaa757f488b6214918/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "873adf109dbe395910a308e11329b0bf5de323160f06b69410fc480c77f24081",
                "Source": "/var/lib/docker/volumes/873adf109dbe395910a308e11329b0bf5de323160f06b69410fc480c77f24081/_data",
                "Destination": "/var/lib/rabbitmq",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "8393c9e84f5f",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "15671/tcp": {},
                "15672/tcp": {},
                "15691/tcp": {},
                "15692/tcp": {},
                "1883/tcp": {},
                "25672/tcp": {},
                "4369/tcp": {},
                "5671/tcp": {},
                "5672/tcp": {},
                "61613/tcp": {}
            },
            "Tty": false,
            "OpenStdin": true,
            "StdinOnce": false,
            "Env": [
                "RABBITMQ_DEFAULT_USER=admin",
                "RABBITMQ_DEFAULT_PASS=admin",
                "PATH=/opt/rabbitmq/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "OPENSSL_VERSION=1.1.1l",
                "OPENSSL_SOURCE_SHA256=0b7a3e5e59c34827fe0c3a74b7ec8baef302b98fa80088d7f9153aa16fa76bd1",
                "OPENSSL_PGP_KEY_IDS=0x8657ABB260F056B1E5190839D9C4D26D0E604491 0x5B2545DAB21995F4088CEFAA36CEE4DEB00CFE33 0xED230BEC4D4F2518B9D7DF41F0DB4D21C1D35231 0xC1F33DD8CE1D4CC613AF14DA9195C48241FBF7DD 0x7953AC1FBC3DC8B3B292393ED5E9E43F7DF9EE8C 0xE5E52560DD91C556DDBDA5D02064C53641C25E5D",
                "OTP_VERSION=24.0.5",
                "OTP_SOURCE_SHA256=a5fec674b11d0a2b888963157a9de60fc384be27ff1a2175cd20708a5b9aa97d",
                "RABBITMQ_DATA_DIR=/var/lib/rabbitmq",
                "RABBITMQ_VERSION=3.9.4",
                "RABBITMQ_PGP_KEY_ID=0x0A9AF2115F4687BD29803A206B73A36E6026DFCA",
                "RABBITMQ_HOME=/opt/rabbitmq",
                "RABBITMQ_LOGS=-",
                "HOME=/var/lib/rabbitmq",
                "LANG=C.UTF-8",
                "LANGUAGE=C.UTF-8",
                "LC_ALL=C.UTF-8"
            ],
            "Cmd": [
                "rabbitmq-server"
            ],
            "Image": "rabbitmq:management",
            "Volumes": {
                "/var/lib/rabbitmq": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "574f2e2ad52792ef20742abd5e3d69d5445c99c4511e8de26714bf6f7efab64b",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "15671/tcp": null,
                "15672/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "15672"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "15672"
                    }
                ],
                "15691/tcp": null,
                "15692/tcp": null,
                "1883/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "1883"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "1883"
                    }
                ],
                "25672/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "25672"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "25672"
                    }
                ],
                "4369/tcp": null,
                "5671/tcp": null,
                "5672/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "5672"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "5672"
                    }
                ],
                "61613/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "61613"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "61613"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/574f2e2ad527",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "2de910c08ba7d0b1971ffb91de2c6973dc169c1f023fea5108330ad02566677a",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "0860c6a0083d7b8481862284faa8ed6f1bc5ba41316667bdd3080496f620874c",
                    "EndpointID": "2de910c08ba7d0b1971ffb91de2c6973dc169c1f023fea5108330ad02566677a",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```


## 10、容器打包成一个镜像
```bash
docker commit -m "描述信息" -a "作者信息"   （容器id或者名称）打包的镜像名称:标签
```

## 11、将镜像保存到本地电脑

```bash
docker save 镜像名 -o  名称.tar
```

然后可以将这个镜像给别人使用，用 `load` 命令载入镜像

## 12、载入本地镜像使用
```bash
docker load -i   名称.tar
```