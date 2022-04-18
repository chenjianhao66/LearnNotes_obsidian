#Docker #容器技术 


## Docker Compose简介

`Compose` 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。从功能上看，跟 `OpenStack` 中的 `Heat` 十分类似。

其代码目前在 [https://github.com/docker/compose](https://github.com/docker/compose) 上开源。

`Compose` 定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」，其前身是开源项目 Fig。

使用一个 `Dockerfile` 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

`Compose` 恰好满足了这样的需求。它允许用户通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

`Compose` 中有两个重要的概念：

-   服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
    
-   项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。
    

`Compose` 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

`Compose` 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 `Compose` 来进行编排管理。

> 目前 Docker 官方用 GO 语言 [重写 (opens new window)](https://github.com/docker/compose-cli)了 Docker Compose，并将其作为了 docker cli 的子命令，称为 `Compose V2`。你可以参照官方文档安装，然后将熟悉的 `docker-compose` 命令替换为 `docker compose`，即可使用 Docker Compose。
> 
--- 

## docker-compose常用命令
### 1. 命令对象与格式

对于 Compose 来说，大部分命令的对象既可以是项目本身，也可以指定为项目中的服务或者容器。如果没有特别的说明，命令对象将是项目，这意味着项目中所有的服务都会受到命令影响。

执行 `docker-compose [COMMAND] --help` 或者 `docker-compose help [COMMAND]` 可以查看具体某个命令的使用格式。

`docker-compose` 命令的基本的使用格式是

```bash
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```

### 2. 命令选项

-   `-f, --file FILE` 指定使用的 Compose 模板文件，默认为 `docker-compose.yml`，可以多次指定。
    
-   `-p, --project-name NAME` 指定项目名称，默认将使用所在目录名称作为项目名。
    
-   `--x-networking` 使用 Docker 的可拔插网络后端特性
    
-   `--x-network-driver DRIVER` 指定网络后端的驱动，默认为 `bridge`
    
-   `--verbose` 输出更多调试信息。
    
-   `-v, --version` 打印版本并退出。


### 3.命令使用说明
#### up

格式为 `docker-compose up [options] [SERVICE...]`。

-   该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。
    
-   链接的服务都将会被自动启动，除非已经处于运行状态。
    
-   可以说，大部分时候都可以直接通过该命令来启动一个项目。
    
-   默认情况，`docker-compose up` 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试。
    
-   当通过 `Ctrl-C` 停止命令时，所有容器将会停止。
    
-   如果使用 `docker-compose up -d`，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。
    
-   默认情况，如果服务容器已经存在，`docker-compose up` 将会尝试停止容器，然后重新创建（保持使用 `volumes-from` 挂载的卷），以保证新启动的服务匹配 `docker-compose.yml` 文件的最新内容

#### down
此命令将会停止 `up` 命令所启动的容器，并移除网络

#### exec
进入指定容器内部

#### ps
格式为 `docker-compose ps [options] [SERVICE...]`。

列出项目中目前的所有容器。

选项：

-   `-q` 只打印容器的 ID 信息。

#### restart
格式为 `docker-compose restart [options] [SERVICE...]`。

重启项目中的服务。

选项：

-   `-t, --timeout TIMEOUT` 指定重启前停止容器的超时（默认为 10 秒）。


#### rm
格式为 `docker-compose rm [options] [SERVICE...]`。

删除所有（停止状态的）服务容器。推荐先执行 `docker-compose stop` 命令来停止容器。

选项：

-   `-f, --force` 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。
    
-   `-v` 删除容器所挂载的数据卷。


#### start
格式为 `docker-compose start [SERVICE...]`。

启动已经存在的服务容器。

#### stop
格式为 `docker-compose stop [options] [SERVICE...]`。

停止已经处于运行状态的容器，但不删除它。通过 `docker-compose start` 可以再次启动这些容器。

选项：

-   `-t, --timeout TIMEOUT` 停止容器时候的超时（默认为 10 秒）。



--- 

## Docker-compose模板命令
模板文件是使用 `Compose` 的核心，涉及到的指令关键字也比较多。这里面大部分指令跟 `docker run` 相关参数的含义都是类似的。

默认的模板文件名称为 `docker-compose.yml`，格式为 YAML 格式。
```yml
version: "3"
services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```

### version
在模板文件中，第一行总是compose的版本号，最高只能到4.0
```yml
version: "3"
```


### services
一个项目中所涉及到的服务在该标签里面定义
```yml
version: "3.0"
services:  						//1
	redis:						//2
		images: redis 			//3
		container_name: redis	//4
		ports:
			- 6379:6379			//5
	tomcat:
		...
	mysqld:
		...
	mongo:
		...
	...
```

1、该项目里所涉及到的服务都需要在该标签里面定义
2、需要定义的具体服务名，该服务名需要唯一标识
3、这个服务所使用的镜像名
4、该镜像所构造出的容器名，可选
5、该容器对外暴露的端口，宿主机端口：容器端口

### image
指定为镜像名称或镜像 ID。如果镜像在本地不存在，`Compose` 将会尝试拉取这个镜像。
有3种写法：
```yml
image: ubuntu   				##镜像名
image: orchardup/postgresql   	##镜像名加标签
image: a4bc65fd					##镜像摘要
```


### container_name
指定容器名称。默认将会使用 `项目名称_服务名称_序号` 这样的格式。相当于启动容器时的 `--name`
```yml
container_name: docker-web-container
```
> 注意: 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。
### network
配置容器连接的网络。

在使用 `network` 的时候需要注意，需要显式声明需要用的网桥，而且 `networks` 该标签和 `services` 同级

```yml
version: "3"
services:

  some-service:
    networks: 			# 指定服务使用的网络桥
     - some-network
     - other-network

networks:
  some-network:			# 声明服务所使用到的网桥
  other-network:
```

### ports
暴露端口信息。相当于 `run -p` 操作

使用宿主端口：容器端口 `(HOST:CONTAINER)` 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。
```yml
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

>注意：当使用 `HOST:CONTAINER` 格式来映射端口时，如果你使用的容器端口小于 60 并且没放到引号里，可能会得到错误结果，因为 `YAML` 会自动解析 `xx:yy` 这种数字格式为 60 进制。为避免出现这种问题，建议数字串都采用引号包括起来的字符串格式。


### volumes
数据卷所挂载路径设置。可以设置为宿主机路径(`HOST:CONTAINER`)或者数据卷名称(`VOLUME:CONTAINER`)，并且可以设置访问模式 （`HOST:CONTAINER:ro`）。相当于 `run -v` 操作

该指令中路径支持相对路径。
```yml
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache 			#使用数据卷名去映射容器目录，注意该数据卷名需要在yml文件内显示声明
 - ~/configs:/etc/configs/:ro
```

如果使用数据卷去映射容器目录，该数据卷一定要在文件里面使用 `volume` 命令显式的声明，而且 `volume` 与 `services` 同级
```yml
version: "3"

services:
  my_src:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

注意，这时候在文件里面声明了一个数据卷，docker里面如果没有数据卷则会自动创建数据卷，但是该数据卷名会在数据卷名之前加入项目名

示例：如果你在 `/var/docker-compose/yourProjectName/` 目录下创建了 `docker-compose.yml`文件，并且在里面声明了一个名为 `test` 的数据卷去映射容器的目录，那么docker就会自动创建一个名字为 `yourProjectName_test` 的数据卷。
> 那不想要docker给我自动生成数据卷名而使用自定义数据卷名呢？
> 通过设置 `external` 字段里的boolen值来实现

沿用上面的代码：
```yaml
version: "3"

services:
  my_src:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:		# 声明使用的数据卷名
  	external:  		# 是否使用自定义卷名
		true		# true表示使用指定卷名，默认是false
```


### environment
设置环境变量。你以使用数组或字典两种格式。

只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。


```yml
environment:
  RACK_ENV: development
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SESSION_SECRET
```


>如果变量名称或者值中用到 `true|false，yes|no` 等表达 [布尔](https://yaml.org/type/bool.html) 含义的词汇，最好放到引号里，避免 YAML 自动解析某些内容为对应的布尔语义。这些特定词汇，包括
>`y|Y|yes|Yes|YES|n|N|no|No|NO|true|True|TRUE|false|False|FALSE|on|On|ON|off|Off|OFF`


### env_file
从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过 `docker-compose -f FILE` 方式来指定 Compose 模板文件，则 `env_file` 中变量的路径会基于模板文件路径。

如果有变量名称与 `environment` 指令冲突，则按照惯例，以后者为准。
```
env_file: .env

env_file:
  - ./common.env
  - ../apps/web.env
  - /opt/secrets.env
```

> 上面获取到的环境变量配置文件分别是
> - **当前目录下的common.env文件**
> - **上一级目录下的app/web.env配置文件**
> - **绝对路径下的secrets.env配置文件**


### depends_on
在使用Compose时，最大的好处就是少打启动命令，但一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。例如在没启动数据库容器的时候启动应用容器，应用容器会因为找不到数据库而退出。depends_on标签用于解决容器的依赖、启动先后的问题。

解决容器的依赖、启动先后的问题。以下例子中会先启动 `redis` `db` 再启动 `web`
```yml
version: '3'

services:
  web:
    build: .
    depends_on:
      - db
      - redis

  redis:
    image: redis

  db:
    image: postgres
```










## 参考博客、文章

[Docker-Compose的一些常用命令](https://cloud.tencent.com/developer/article/1499032)
[docker-compose模板常用命令](https://www.cnblogs.com/linagcheng/p/14000245.html)