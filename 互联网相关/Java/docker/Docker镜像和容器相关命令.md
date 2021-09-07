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
