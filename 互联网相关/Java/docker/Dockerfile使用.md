2021-09-11
14:45:31
author:陈建浩

#Docker 
--- 


## Docker file简介
镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

Dockerfile就是文件名，这是约定好的，每一个构建脚本文件都是 `Dockerfile` 文件。

Dockerfile 在整个docker生态里面的文件

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/image-20200404111908085-0291323.png)


> 为什么要存在一个 Dockerfile ？
> docker hub中官方提供了很多镜像已经满足了所有服务，那为什么还需要自定义镜像？
> 答：日后用户可以将自己的应用打包成镜像，这样就可以让我们应用能在容器内运行；


## Dockerfile的命令
| 保留字         | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| **FROM**       | **当前镜像是基于哪个镜像的** `第一个指令必须是FROM`          |
| MAINTAINER     | 镜像维护者的姓名和邮箱地址                                   |
| **RUN**        | **构建镜像时需要运行的指令**                                 |
| **EXPOSE**     | **当前容器对外暴露出的端口号**                               |
| **WORKDIR**    | **指定在创建容器后，终端默认登录进来的工作目录，一个落脚点** |
| **ENV**        | **用来在构建镜像过程中设置环境变量**                         |
| **ADD**        | **将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar包** |
| **COPY**       | **类似于ADD，拷贝文件和目录到镜像中<br/>将从构建上下文目录中<原路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置** |
| **VOLUME**     | **容器数据卷，用于数据保存和持久化工作**                     |
| **CMD**        | **指定一个容器启动时要运行的命令<br/>Dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被docker run之后的参数替换** |
| **ENTRYPOINT** | **指定一个容器启动时要运行的命令<br/>ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及其参数** |


Dockerfile 由一行行命令语句组成，并且支持以 `#` 开头的注释行。

一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

其中，一开始必须指明所基于的镜像名称，接下来推荐说明维护者信息;
后面则是镜像操作指令，例如 `RUN` 指令，`RUN` 指令将对镜像执行跟随的命令。每运行一条 `RUN` 指令，镜像添加新的一层，并提交。

最后是 `CMD` 指令，来指定运行容器时的操作命令。


### FROM指定基础镜像

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。基础镜像是必须指定的。而 `FROM` 就是指定 **基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。

在 [Docker Hub (opens new window)](https://hub.docker.com/search?q=&type=image&image_filter=official)上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像，如 [`nginx` (opens new window)](https://hub.docker.com/_/nginx/)、[`redis` (opens new window)](https://hub.docker.com/_/redis/)、[`mongo` (opens new window)](https://hub.docker.com/_/mongo/)、[`mysql` (opens new window)](https://hub.docker.com/_/mysql/)、[`httpd` (opens new window)](https://hub.docker.com/_/httpd/)、[`php` (opens new window)](https://hub.docker.com/_/php/)、[`tomcat` (opens new window)](https://hub.docker.com/_/tomcat/)等；也有一些方便开发、构建、运行各种语言应用的镜像，如 [`node` (opens new window)](https://hub.docker.com/_/node)、[`openjdk` (opens new window)](https://hub.docker.com/_/openjdk/)、[`python` (opens new window)](https://hub.docker.com/_/python/)、[`ruby` (opens new window)](https://hub.docker.com/_/ruby/)、[`golang` (opens new window)](https://hub.docker.com/_/golang/)等。可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。

如果没有找到对应服务的镜像，官方镜像中还提供了一些更为基础的操作系统镜像，如 [`ubuntu` (opens new window)](https://hub.docker.com/_/ubuntu/)、[`debian` (opens new window)](https://hub.docker.com/_/debian/)、[`centos` (opens new window)](https://hub.docker.com/_/centos/)、[`fedora` (opens new window)](https://hub.docker.com/_/fedora/)、[`alpine` (opens new window)](https://hub.docker.com/_/alpine/)等，这些操作系统的软件库为我们提供了更广阔的扩展空间。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 `scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

  
格式为 `FROM <image>`或`FROM <image>:<tag>`。

第一条指令必须为 `FROM` 指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个 `FROM` 指令（每个镜像一次）。

如果你以 `scratch` 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

### MAINTAINER
该镜像的维护者以及邮箱
格式为 `MAINTAINER <name>`，指定维护者信息。
```bash
MAINTAINER jianhao.chen jianhao66@qq.com
```

### RUN
`RUN` 指令是用来执行命令行命令的。由于命令行的强大能力，`RUN` 指令在定制镜像时是最常用的指令之一。格式有两种：

 `RUN <命令>` 或 `RUN ["可执行文件", "参数1", "参数2"]`：
`RUN ["sudo","apt-get","install","vim"]`

前者将在 shell 终端中运行命令，即 `/bin/sh -c`；后者则使用 `exec` 执行。指定使用其它终端可以通过第二种方式实现，例如 `RUN ["/bin/bash", "-c", "echo hello"]`。

每条 `RUN` 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 `\` 来换行。

示例：
```bash
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

之前说过，Dockerfile 中每一个指令都会建立一层，`RUN` 也不例外。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，`commit` 这一层的修改，构成新的镜像。

而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。

正确的做法是一个 `RUN` 指令使用 `&&` 将各个所需命令串联起来，正确示例：
```bash
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```


### EXPOSE
格式为 `EXPOSE <port> [<port>...]`。

告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 -P，Docker 主机会自动分配一个端口转发到指定的端口。

> 只有在构建镜像的时候使用 `EXPOSE` 指令告诉Docker服务器暴露的端口时，在运行容器的时候才能使用 -p 去对外暴露端口
> 也就是说 启动容器 -p 的端口没有在指令 `EXPOSE`暴露的端口内出现的话，-p暴露的端口无效！

### WORKDIR
**指定在创建容器后，终端默认登录进来的工作目录，一个落脚点**
格式为 `WORKDIR /path/to/workdir`。

为后续的 `RUN`、`CMD`、`ENTRYPOINT` 指令配置工作目录。

可以使用多个 `WORKDIR` 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。
```bash
WORKDIR /a
WORKDIR b
WORKDIR c

# 最终进入终端后的目录在： /a/b/c
```

### ENV
格式为 `ENV <key> <value>`。 指定一个环境变量，会被后续 `RUN` 指令使用，并在容器运行时保持。

用来为构建镜像设置环境变量。这个值将出现在构建阶段中所有后续指令的环境中。在使用的时候用 `$key` 来引用。

### ADD
格式为 `ADD <src> <dest>`。
用来从context上下文复制新文件、目录或远程文件url，并将它们添加到位于指定路径的映像文件系统中。

该命令将复制指定的 `<src>` 到容器中的 `<dest>`。 其中 `<src>` 可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）。

ADD可以实现的语法：
```bash
ADD hom* /mydir/       通配符添加多个文件
ADD hom?.txt /mydir/   通配符添加
ADD test.txt relativeDir/  可以指定相对路径
ADD test.txt /absoluteDir/ 也可以指定绝对路径
ADD test.txt test1.text   /添加到容器并且改名
ADD url 
```

### COPY
格式为 `COPY <src> <dest>`。


复制本地主机的 `<src>`（为 Dockerfile 所在目录的相对路径(上下文目录)）到容器中的 `<dest>`。

当使用本地目录为源目录时，推荐使用 `COPY`。

### VOLUME
格式为 `VOLUME ["/data"]`。

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

### CMD
**指定一个容器启动时要运行的命令  
Dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被docker run之后的参数替换**

支持三种格式

-   `CMD ["executable","param1","param2"]` 使用 `exec` 执行，推荐方式；
-   `CMD command param1 param2` 在 `/bin/sh` 中执行，提供给需要交互的应用；
-   `CMD ["param1","param2"]` 提供给 `ENTRYPOINT` 的默认参数；

指定启动容器时执行的命令，每个 Dockerfile 只能有一条 `CMD` 命令。如果指定了多条命令，只有最后一条会被执行。

> 如果用户启动容器时候指定了运行的命令，则会覆盖掉 `CMD` 指定的命令。

### ENTRYPOINT
**指定一个容器启动时要运行的命令  
ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及其参数**

两种格式：

-   `ENTRYPOINT ["executable", "param1", "param2"]`
-   `ENTRYPOINT command param1 param2`（shell中执行）。

> 配置容器启动后执行的命令，并且不可被 `docker run` 提供的参数覆盖。

每个 Dockerfile 中只能有一个 `ENTRYPOINT`，当指定多个时，只有最后一个起效。



## 使用Dockerfile构建镜像

### 构建一个仅有 FROM 基础镜像的镜像

在一个目录中创建 `Dockerfile` 文件
```bash
vim Dockerfile
```
该文件所在的目录被称之为 `上下文目录` ，这个目录下面的所有文件都会打包进镜像里面，所以在进行构建的时候需要额外斟酌哪个文件是必须的哪些文件不是必须的，减少所构建成功的镜像的大小。

创建了 `Dockerfile` 文件之后，第一条指令必须是 `FROM` 指令
```bash
FROM centos:7
```

在配置好了文件内容之后就可以使用 `docker build` 命令来构建镜像
格式为：`docker build -t [构建好的镜像名]:[tag] [Dockerfile目录]`
参数 t 代表：构建好的镜像名和版本号

```bash
docker build -t myCentos7:01 .

# 以上命令意思是，在当前目录构建一个名字为myCentos7，tag为01的镜像
# .代表着是Dockerfile文件在当前目录，当前目录是上下文目录
```

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-09-11_15-43.png)
构建成功！

构建成功之后使用 `docker images` 命令来查询构建的镜像是否出现

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-09-11_16-07.png)

构建的镜像已经出现！



### 构建一个相对复杂的镜像
1、创建 `Dockerfile` 文件，并进行配置
```bash
FROM centos:7
ADD apache-tomcat-9.0.52.tar.gz /data/tomcat
RUN yum -y install wget
	&& yum -y install vim
	&& 
WORKDIR /data/tomcat/apache-tomcat-9.0.52/
EXPOSE 8080
ENV 
```