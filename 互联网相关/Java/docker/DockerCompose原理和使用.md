2021-09-11
13:25:16
author:陈建浩


--- 

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


### Volumes
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